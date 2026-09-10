---
title: 鸿蒙ArkWeb与Http请求的Cookie共享与踩坑记录
categories:
  - 鸿蒙HarmonyOS启程之路
excerpt: ArkWeb(WebView)与Http请求要共享会话时，Cookie经常是第一坑。本文记录从“invalid url”到“cookie value格式错误”的两轮排查，澄清NetworkKit的response.cookies并非原始Cookie串，给出从响应头提取Cookie并写入WebCookieManager的做法，以及常见错误码定位思路与注意事项。
date: 2025-06-30 19:33:35
tags:
  - HarmonyOS
  - ArkWeb
  - HTTP
  - Cookie
  - 网络请求
  - 问题排查
---

## 背景

ArkWeb(WebView)与Http网络请求需要共享Cookie以维持登录态，但实际接入时频繁报错。本文按排查顺序记录问题与修复。

## 参考：官方错误码文档

https://developer.huawei.com/consumer/cn/doc/harmonyos-references/errorcode-webview

## 错误一：invalid url（17100002）

官方说明：

```
17100002 Url格式错误
URL error. Possible causes: 1. No valid cookie found for the specified URL. 2. The webpage corresponding to the URL is invalid, or the URL length exceeds 2048.
处理：检查输入的url是否正确且长度不超过2048。
```

实情：我这边是“传入的url不对”导致（带了不必要的参数/格式不规范）。修正URL后继续验证。

## 错误二：cookie value格式错误（17100005）

官方说明：

```
17100005 cookie value格式错误
The provided cookie value is invalid. It must follow RFC 6265.
```

定位过程：
- 我最初使用 `import { http } from '@kit.NetworkKit';` 发起请求，读取 `response.cookies`，看到类型是 string，就以为可以直接 `configCookieSync(url, cookies)`；
- 但真实情况是：这个 `cookies` 并非“原始Cookie请求头字符串”，而是一个“格式化后的cookie导出文本”，中间包含制表符（\t）。例如：

```
#HttpOnly_abc.com.cn	FALSE	/v4/	FALSE	0	JSESSIONID	C939723C706D2182B11F33D5A37927BE
```

- 这种格式不符合 `WebCookieManager.configCookieSync()` 所需要的 Cookie 写入规范，因而被系统拒绝。

## 正确做法：从响应头提取原始Cookie

不要直接用 `response.cookies`。应当从响应头的 `Set-Cookie` / `Set-Cookie2`（以及少量旧接口的 `Cookie` / `Cookie2`）中提取逐条Cookie后写入。

下面方法做了兼容性解析，处理了：
- `Set-Cookie` 一行内的多个 Cookie（逗号与 Expires 冲突）；
- 多行合并/换行的场景；
- 既支持数组，也支持单值；
- 兼容少量旧服务端返回的 `Cookie`/`Cookie2`。

```ts
/** 解析响应头中的 Cookie，返回逐条 cookie 字符串 */
public static extractCookiesFromHeaders(headers: Object | undefined): string[] {
  const out: string[] = [];
  if (!headers) { return out; }
  const headerMap = CookieUtils.transformObjectToRecord(headers);
  if (!headerMap) { return out; }

  const toStringArray = (val: Object): string[] => {
    const res: string[] = [];
    if (Array.isArray(val)) {
      const arr = val as Array<Object>;
      for (const item of arr) { const s: string = String(item); if (s) { res.push(s); } }
    } else {
      const s: string = String(val); if (s) { res.push(s); }
    }
    return res;
  };

  const splitSetCookieCombined = (value: string): string[] => {
    const lines = value.indexOf('\n') >= 0 || value.indexOf('\r') >= 0 ? value.split(/\r?\n/g) : [value];
    const result: string[] = [];
    for (const line of lines) {
      const s = line.trim(); if (!s) { continue; }
      let inExpires = false;
      let start = 0;
      const lower = s.toLowerCase();
      for (let i = 0; i < s.length; i++) {
        if (!inExpires && lower.indexOf('expires=', i) === i) { inExpires = true; i += 'expires='.length - 1; continue; }
        if (inExpires && s.charAt(i) === ';') { inExpires = false; continue; }
        if (!inExpires && s.charAt(i) === ',') {
          const part = s.substring(start, i).trim(); if (part) { result.push(part); }
          start = i + 1;
        }
      }
      const tail = s.substring(start).trim(); if (tail) { result.push(tail); }
    }
    return result;
  };

  const keys: string[] = Object.keys(headerMap);
  for (const key of keys) {
    const lowerKey = key.toLowerCase();
    const rawVal: Object = (headerMap as Record<string, Object>)[key];
    if (lowerKey === 'set-cookie' || lowerKey === 'set-cookie2') {
      const arr = toStringArray(rawVal);
      for (const item of arr) {
        const pieces = splitSetCookieCombined(item);
        for (const p of pieces) { const s = p.trim(); if (s) { out.push(s); } }
      }
    } else if (lowerKey === 'cookie' || lowerKey === 'cookie2') {
      const arr = toStringArray(rawVal);
      for (const item of arr) {
        const lineParts = item.indexOf('\n') >= 0 || item.indexOf('\r') >= 0 ? item.split(/\r?\n/g) : [item];
        for (const lp of lineParts) { const s = lp.trim(); if (s) { out.push(s); } }
      }
    }
  }

  return out;
}
```

## 把Cookie写入ArkWeb的CookieStore

```ts
import { webview as ArkWebview } from '@kit.ArkWeb';

/** 为指定 URL 保存一条 Cookie 到 WebView */
public static setCookie(url: string, value: string): boolean {
  if (!url || !value) {
    BDebug.warn(TAG, 'setCookie', 'url or value empty');
    return false;
  }
  const urlWithoutQuery = CookieUtils.stripQuery(url);
  try {
    ArkWebview.WebCookieManager.configCookieSync(urlWithoutQuery, value);
    ArkWebview.WebCookieManager.saveCookieSync();
    return true;
  } catch (error) {
    BDebug.errorStackPrint(TAG, `setCookie failed: ${urlWithoutQuery}`, error as object);
    return false;
  }
}
```

要点：
- （非必需，只为了缩短url长度）写入前剥离URL上的查询参数，只保留协议+主机+路径（与Cookie作用域匹配）；
- 写入后调用 `saveCookieSync()` 持久化（不调用持久化接口的话，更新会延迟一些）；

## 常见坑位与排查建议

- 不要把 NetworkKit 的 `response.cookies` 直接喂给 `configCookieSync()`；
- URL不规范会触发 17100002；Cookie串不合规会触发 17100005；
- `Set-Cookie` 逗号切分要避开 `Expires=` 中的逗号；

## 总结

终于生效了，总算是把这个小小的 Cookie 坑填上了。看起来不复杂，但细节是真的多：URL 一不规范就 17100002，Cookie 串不合规就 17100005，`response.cookies` 还不是原始值，直接拿来用就会挨打。说实话，这种“看起来像样其实不能直接用”的字段最容易骗新人（也骗我）。
