---
title: 使用Deeplink进行团队内部测试版本发布实践记录
categories: 
  - 鸿蒙HarmonyOS启程之路
excerpt: 在鸿蒙应用开发过程中，团队内部测试版本的分发有点麻烦。最近发现鸿蒙提供了一种基于Deeplink的内部测试发布机制，可以通过生成描述文件让测试人员直接安装应用，体验下来确实比传统方式方便不少。本文记录实践过程，顺便聊聊这种方式与其他测试发布方式的区别。
date: 2025-08-24 15:22:00
tags:
  - HarmonyOS
  - DeepLink
  - 应用分发
  - 内部测试
  - HAP
  - 测试发布
---

## 背景与起因

最近在做鸿蒙版AppCan容器的开发，团队内部测试版本的分发成了个小麻烦。以前Android开发时，直接丢个APK文件到群里就完事了，但鸿蒙的HAP文件不能直接安装，需要通过DevEco Studio或者其他方式才能装到设备上。

测试同事抱怨说每次都要找开发帮忙安装，开发也觉得频繁被打断影响效率。正好在翻鸿蒙文档时发现了一个叫"内部测试发布"的功能，可以通过Deeplink让测试人员直接安装应用，感觉挺有意思的。

## 鸿蒙内部测试发布机制

### 基本原理

鸿蒙的内部测试发布机制其实与iOS的企业应用下载方式如出一辙，主要流程是：

1. 准备一个HTTPS服务器，用于存放HAP包和描述文件
2. 打包HAP文件，注意使用发布证书以及内部测试描述文件，描述文件生成时需要提前注册设备UDID，上限100个。[鸿蒙设备UDID获取方式](https://developer.huawei.com/consumer/cn/doc/app/agc-help-add-device-0000001946142249#section67331926102911)
3. 将HAP包上传到服务器，生成HTTPS下载链接
4. 根据应用信息生成一个描述文件（manifest.json5）
5. 使用鸿蒙提供的签名工具对描述文件进行签名
6. 测试人员通过浏览器访问描述文件链接，系统会自动拉起安装流程

整个过程对测试人员来说就像访问一个网页链接一样简单，点击后系统会自动下载并安装应用。

而这个功能目前是鸿蒙普通开发者也可以使用，只不过对于安装设备总数是有限制的，这一点比iOS的稍微宽松一些。

### 与其他测试方式的区别

在鸿蒙生态中，应用测试发布有几种不同的方式：

**调试版本**：

开发阶段通过DevEco Studio直接安装，只能在开发环境使用，无法线上分发给其他人。

**AppTest邀请测试**：

通过华为开发者联盟的AppTest平台进行测试，需要上传到华为服务器，并经过审核，测试人员需要安装AppTest应用才能参与测试。

**AppGallery邀请测试**：

应用商店的内测功能，需要先提交应用审核，然后邀请测试用户，流程相对正式。

**内部测试发布**：

本文介绍的这种方式，完全由开发者自己控制，不依赖华为的测试平台，灵活性最高。

相比之下，内部测试发布的优势很明显：

- **部署灵活**：可以部署在自己的服务器上，不受第三方平台限制
- **访问简单**：测试人员只需要一个链接，无需安装额外应用
- **更新及时**：新版本生成后立即可用，无需等待平台审核
- **权限控制**：可以通过服务器访问控制来限制测试范围

## 实际操作体验

### 描述文件的生成

描述文件（manifest.json5）包含了应用的基本信息和下载链接：

```json5
{
  "app": {
    "bundleName": "com.example.myapp",
    "versionCode": 1000000,
    "versionName": "1.0.0",
    "label": "我的应用",
    "deployDomain": "example.com",
    "icons": {
      "normal": "https://example.com/icons/app_icon.png",
      "large": "https://example.com/icons/app_icon_large.png"
    },
    "modules": [
      {
        "name": "entry",
        "type": "entry",
        "deviceTypes": ["phone", "tablet"],
        "packageUrl": "https://example.com/packages/entry-signed.hap",
        "packageHash": "sha256值"
      }
    ]
  },
  "sign": "签名字符串"
}
```

不过呢，手动制作这个文件还是挺繁琐的，需要从项目的各个配置文件中提取信息，还要计算HAP文件的SHA256值，最后还要用鸿蒙的签名工具进行签名。

### 测试人员的使用体验

从测试人员的角度来看，整个安装过程确实很顺滑，但需要注意华为的安全限制：

1. 收到一个应用下载页面链接，比如 `https://example.com/download-app.html`
2. 在鸿蒙设备的**系统自带浏览器**中打开链接（第三方浏览器不支持）
3. 页面显示应用信息、版本说明等，并提供一个"下载安装"按钮
4. 点击按钮后，页面通过JavaScript触发Deeplink跳转，格式为：
   ```
   store://enterprise/manifest?url=https://example.com/internal-manifest-signed.json5
   ```
5. 系统自动识别Deeplink并拉起安装流程，下载描述文件
6. 确认安装，应用自动下载并安装完成

**重要限制**：

- 描述文件不能直接通过HTTPS链接访问，必须封装为Deeplink格式
- Deeplink格式：`store://enterprise/manifest?url=描述文件的HTTPS地址`
- 必须通过用户点击事件触发，不能直接在地址栏输入
- 必须使用系统自带浏览器（华为浏览器），第三方浏览器无法触发安装
- URL参数需要进行编码处理

**Web页面示例**：

```html
<!DOCTYPE html>
<html>
<head>
    <title>应用下载</title>
</head>
<body>
    <h1>我的应用 v1.0.0</h1>
    <p>应用描述信息...</p>
    <button onclick="downloadApp()">下载安装</button>

    <script>
    function downloadApp() {
        const manifestUrl = 'https://example.com/internal-manifest-signed.json5';
        const deeplink = 'store://enterprise/manifest?url=' + encodeURIComponent(manifestUrl);
        window.location.href = deeplink;
    }
    </script>
</body>
</html>
```

因此我们还需要制作一个专门的Web页面来展示应用信息并提供下载按钮，这个页面才是真正分发给测试人员的链接。

## 踩坑记录

### 先贴错误码链接

手机上点击安装如果报错了，有个错误码，对照来排查即可。我遇到过两个错误，10021是设备没注册在描述文件里，或者描述文件有错误；10024是App信息与描述文件内的信息不匹配。

https://developer.huawei.com/consumer/cn/doc/app/agc-help-internal-test-errorcode-0000002295325157

### 域名一致性问题

描述文件中的 `deployDomain` 必须与所有URL中的域名保持一致，否则系统会拒绝安装。这个细节在文档中提到了，但实际操作时容易忽略。

### 远程服务器文件部署问题

在部署HAP文件和应用图标到远程服务器时，遇到了一些意想不到的问题。

**GitHub/Gitee方案失败**：

最初我想偷个懒，直接把HAP文件和图标上传到GitHub或Gitee，然后通过raw文件链接来提供下载。但实际测试发现这种方式不行，因为鸿蒙系统要求下载文件时服务器必须返回 `Content-Disposition` 响应头，而GitHub的raw文件服务并不提供这个头部信息。

**Python文件服务方案**：

最终我采用了自定义Python服务来解决这个问题。由于我的服务端本来就运行着Python服务，所以增加一个文件下载功能相对简单：

```python
from flask import Flask, send_file, make_response
import os

app = Flask(__name__)

@app.route('/packages/<filename>')
def download_package(filename):
    file_path = os.path.join('/path/to/packages', filename)
    if os.path.exists(file_path):
        response = make_response(send_file(file_path, as_attachment=True))
        response.headers['Content-Disposition'] = f'attachment; filename={filename}'
        return response
    return "File not found", 404
```

**Nginx配置方案**：

如果使用Nginx作为文件服务器，也需要在配置中添加相应的响应头（示例，我暂时没有采用，预计未来会用）：

```nginx
location /packages/ {
    alias /path/to/packages/;
    add_header Content-Disposition 'attachment';
    try_files $uri =404;
}
```

这个细节在官方文档中提到得不够明显，实际部署时容易忽略，导致安装时提示下载失败。

### 描述文件内字段的准确性

描述文件中的各个字段都需要严格按照规范填写，并且与打包生成hap时所采用的项目参数完全一致，否则系统会拒绝安装。

### HAP文件的SHA256计算

不同平台计算SHA256的命令不同：
- Windows: `certutil -hashfile 文件路径 SHA256`
- Mac/Linux: `shasum -a 256 文件路径`

而且要注意提取正确的哈希值，有些命令输出包含额外信息需要过滤。

### 签名工具的使用

鸿蒙提供的签名工具是一个Java程序（manifest-sign-tool-1.0.0.jar），用于对描述文件进行数字签名。这个工具的使用相对复杂，需要准备证书文件和正确的参数配置。

**工具获取**：

签名工具可以从华为开发者官网下载，通常包含在鸿蒙开发工具包中。

**官方参考资料**：

- 工具下载和详细用法说明：https://gitee.com/arkin-internal-testing/internal-testing

**证书准备**：

需要准备P12格式的证书文件，必须是发布证书（也就是与正式发布的证书相同类型，也可以是同一个），这个证书用于对描述文件进行签名。证书配置需要包含以下信息：
- 证书文件路径（.p12文件）
- 证书库密码（keystorePassword）
- 证书别名（keyAlias）
- 私钥密码（keyAliasPassword）

**命令格式**：

官方提供了一个bat文件，供windows环境使用。实际上是调用了一个jar包，具体示例如下：

```bash
java -jar manifest-sign-tool-1.0.0.jar \
  --operation sign \
  -mode localjks \
  -inputFile "input-manifest.json5" \
  -outputFile "output-manifest-signed.json5" \
  -keystore "path/to/certificate.p12" \
  -keystorepasswd "证书库密码" \
  -keyaliaspasswd "私钥密码" \
  -privatekey "证书别名"
```

**参数说明**：
- `--operation sign`：固定参数，指定签名操作
- `-mode localjks`：固定参数，指定本地证书模式
- `-inputFile`：输入的原始描述文件路径
- `-outputFile`：输出的签名后描述文件路径
- `-keystore`：P12证书文件的完整路径
- `-keystorepasswd`：证书库密码
- `-keyaliaspasswd`：私钥密码
- `-privatekey`：证书别名

**常见错误**：

- 证书路径错误或文件不存在
- 密码不正确导致证书无法读取
- 证书别名不匹配
- 输入文件格式不正确

签名成功后，原始描述文件中的 `"sign": ""` 字段会被填入实际的签名字符串，生成最终可用的描述文件。

## 自动化脚本的必要性

手动制作描述文件确实太繁琐了，特别是需要从多个配置文件中提取信息，获取hash值，签名，还要处理各种格式转换。为了提高效率，我写了个Node.js脚本来自动化这个过程。

脚本的核心思路很简单：
1. 从项目配置文件（app.json5、module.json5等）中自动提取应用信息
2. 根据用户配置的部署信息生成完整的描述文件
3. 自动计算HAP文件的SHA256值
4. 调用鸿蒙签名工具完成签名

这样每次发布新版本时，只需要运行一个命令就能生成完整的描述文件，大大节省了时间，也避免了手动操作可能带来的错误。

## 总结

鸿蒙的内部测试发布机制确实是个不错的功能，特别适合团队内部的快速迭代测试。虽然初次配置稍微复杂一些，但一旦流程跑通，后续的使用体验还是很不错的。

对于有频繁内测需求的团队来说，投入一些时间做自动化工具是值得的。毕竟开发效率的提升，最终还是要体现在产品质量上。

## 后续

还有高手啊！看到一篇文章，这哥们做了个DevEco里的IDEA插件，直接图形化点击完成上面的描述文件生成操作，并且还支持一键上传到云服务，看起来挺方便的。不过既然我已经写好脚本了，也比较适合后续我们团队内部用于其他产品线自动化打包提供参考，所以也算没白忙活。

---

**参考文档**：

[鸿蒙应用内部测试发布指南](https://developer.huawei.com/consumer/cn/doc/app/agc-help-internal-test-release-app-0000002260691994)
[鸿蒙设备UDID获取方式](https://developer.huawei.com/consumer/cn/doc/app/agc-help-add-device-0000001946142249#section67331926102911)
