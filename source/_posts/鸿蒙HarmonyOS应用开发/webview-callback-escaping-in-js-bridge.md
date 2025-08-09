---
title: 鸿蒙ArkTS与ArkWeb交互中的字符串转义问题与修复
categories: 
  - 鸿蒙HarmonyOS启程之路
excerpt: 我在鸿蒙版AppCan框架的开发中，遇到一个场景，服务端通过http接口返回的JSON内嵌HTML，但因未对其中双引号的转义进行保护，导致前端JSON.parse报错。看似错误离奇，实则是开发过程中思考不够谨慎所致，并不是什么新鲜bug。故记录以提醒自己。
date: 2025-08-09 13:07:43
tags: 
---

## 背景与现象

在某项目App的集成测试中：

- 某新闻板块 → 列表页（listPage）→ 详情页（pageDetail）
- 进入“媒体报道”分支后渲染阶段报错：
  - `SyntaxError: Expected ',' or '}' after property value in JSON ...`
  - 连锁触发 `appcan.js` 的 context 相关错误
- 结果：详情页空白

日志片段（节选）：

```
newsDetail: {"status":"000","msg":"success","data":[{"title":"[人民日报]...",
"content":"<div class="article" ...> ..."}]}
SyntaxError: Expected ',' or '}' after property value in JSON at position XXX
```

> 关键点：content 内含 HTML，存在大量引号与反斜杠。通过position定位到，报错点就是`class="article article-detail"`中的双引号与外面的json双引号冲突导致了json解析异常。

但是这个页面代码在Android里是正常的，看来还是跟鸿蒙具体的处理逻辑有关。

## 具体排查，梳理业务逻辑

1. H5页面通过 `appcan.request.ajax` 发起 POST，`contentType=application/json`，`data` 为 JSON 字符串，而appcan.request.ajax内部会调用ArkTS封装的uexXmlHttpMgr原生插件接口，内部通过`@kit.NetworkKit`网络库发送请求。
2. 原生插件（uexXmlHttpMgr）回调时有两种方式，其中v4回调方式以传入的js Function对象进行回调，未出现问题；而以“方法名回调（v3 兼容）”返回则出现了问题：
   - 拼接 JS 代码字符串：`javascript: if(uexXmlHttpMgr.onData){ onData(opCode, state, 'resStr', ...) }`
   - 其中 `resStr` = 代表了服务端返回的整段 JSON 文本，其中包含了双引号，并且**服务端返回时**，其中的双引号**已经转义**成 `\"` 了；
3. H5端接收到回调内容后 `success(data)` 内部继续 `JSON.parse(data)` → 报错，从报错原因来看，是因为双引号没有转义引起的。
4. 奇怪，内部已经转义了，为什么还会报错呢？

## 根因分析：将拼接的JS字符串作为JS代码执行时，会消费一次转义

- 插件直接把服务端 JSON 文本塞进回调：`... onData(..., 'resStr', ...)`；
- JS 引擎会先解析这个JSON字面量，其中的 `\"` 会在此阶段被还原成 `"`；
- 传到前端的 `data` 已经失去 JSON 需要的反斜杠，`JSON.parse(data)` 自然失败。
- 也就是说，本身服务端返回的数据是给前端拿到后直接解析的，结果我们中间加了转接层，导致把这个转义丢失了。这样是不对的，中间层要保持透明。
- 为了解决此问题，我们就要针对转义的字符串进行二次转义，这样前端收到的时候就是一次转义的结果了。

### 示例输入

服务端（概念化）：

```json
{
  "status":"000",
  "msg":"success",
  "data":[{
    "title":"[人民日报]...",
    "content":"<div class=\"article article-detail\" style=\"...\">...<img src=\"https://.../a.jpg\" />...</div>"
  }]
}
```

插件注入（抽象化）：

```js
uexXmlHttpMgr.onData(opCode, 1, '{"status":"000"..."content":"<div class=\"article\" ...>"}', 200, '...')
```

## 修复办法

仔细查阅了Android侧的实现，发现还有个小细节， Android中会通过`BUtility.transcoding`方法对服务端返回数据进行转码处理，其内部实现如下：

```java
public static String transcoding(String text) {
    String regEx = "\n|\r|\"|'|\\\\|&";
    Pattern p = Pattern.compile(regEx);
    Matcher m = p.matcher(text);
    StringBuffer sb = new StringBuffer();

    while(m.find()) {
        if (m.group(0).equals("\n")) {
            m.appendReplacement(sb, "\\\\n");
        } else if (m.group(0).equals("\r")) {
            m.appendReplacement(sb, "\\\\r");
        } else if (m.group(0).equals("\"")) {
            m.appendReplacement(sb, "\\\\\"");
        } else if (m.group(0).equals("'")) {
            m.appendReplacement(sb, "\\\\'");
        } else if (m.group(0).equals("\\")) {
            m.appendReplacement(sb, "\\\\\\\\");
        } else if (m.group(0).equals("&")) {
            m.appendReplacement(sb, "\\\\&");
        }
    }

    m.appendTail(sb);
    return sb.toString();
}
```

很好，不用费脑筋去罗列有哪些字符需要转义，直接照搬过来就行了~那么，我们将其移植到 ArkTS，并在 uexXmlHttpMgr 的 v3 回调中对回调结果字符串使用此方法进行转换即可：

ArkTS 示例实现：

```ts
  /**
   * 字符串转码（与AppCan Android版BUtility.transcoding逻辑保持一致）
   * 将字符串中的换行、回车、双引号、单引号、反斜杠、& 符号做JS字面量安全转义。
   * 注意：该方法仅用于在“拼接JS代码字符串进行回调”之前对字符串参数进行预转义，
   * 以确保经过JS引擎一次字符串字面量解析后仍然保留JSON所需的转义字符。
   * 对应AppCan Android实现：BUtility.transcoding(String text)
   */
  public static transcoding(text: string): string {
    if (text === undefined || text === null) {
      return '';
    }
    // 使用单次扫描替换，与Android实现一致
    return text.replace(/[\n\r"'\\&]/g, (match: string): string => {
      switch (match) {
        case '\n': return '\\n';
        case '\r': return '\\r';
        case '"': return '\\"';
        case "'": return "\\'";
        case '\\': return '\\\\';
        case '&': return '\\&';
        default: return match;
      }
    });
  }
```

> 所有使用 v3 的字符串回调接口受益，业务无需改代码。

## v3/v4 回调差异

- v3：方法名回调，需要拼字符串，注入 JS 代码字符串 → 必须做“JS 字面量级转义”；
- v4：直接传 Function 对象，不拼接 JS 字符串 → 不需要此类转义。

## 总结

凡是“要拼接成 JS 代码再回调”的字符串，而且这个字符串由模块外部提供（例如服务端），对其内容不可控的情况下，并且需要透传的情况下，则需要去检查是否需要做 **JS 字面量安全转义**（不一定是必须的，需要根据实际场景判断）。

