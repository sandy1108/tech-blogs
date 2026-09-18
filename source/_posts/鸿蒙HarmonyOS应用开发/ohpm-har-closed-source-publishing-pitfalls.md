---
title: 鸿蒙 OHPM 包发布踩坑记录：从 HAR 构建到闭源发布
categories:
  - 鸿蒙HarmonyOS应用开发
excerpt: 记录一次 AppCan 鸿蒙引擎更新 OHPM 包时遇到的发布问题，包括 release HAR 构建、README 与 CHANGELOG、author 和 repository 校验、Publish ID 与私钥、闭源许可及 source_type 等容易忽略的细节。
date: 2026-06-22 01:14:00
tags:
  - HarmonyOS
  - OHPM
  - HAR
  - 鸿蒙开发
  - 踩坑记录
---

之前已经把 AppCan 鸿蒙版的 engine 发布到 OHPM 过一次。后来准备更新到 `1.0.1`，本来以为就是重新打个 HAR、再执行一次发布命令，结果又撞上了一串小问题。

有些坑第一次发布时其实已经踩过，比如 README 和 CHANGELOG 的要求；有些则是这次更新才发现，比如 author、repository、闭源许可和 `source_type`。单独看都不复杂，凑在一起却很容易让一次发布来回返工，所以干脆统一记录一下。

## 先简单说说 OHPM 包怎么发布

OHPM 是 OpenHarmony 的包管理体系。准备发布包时，先进入 [OpenHarmony 三方库中心](https://ohpm.openharmony.cn/)，注册并登录账号，然后按照网站流程创建用于发布的软件包或仓库，并准备好发布身份与密钥配置。

这里有一点我记得稍微有点特殊，就是发布需要的密钥必须要带密码的，本来我想让它复用我的ssh密钥，还有github密钥，结果报错，说必须要设密码。这一点后面会提到。

## 正常的更新发布流程

如果前面的账号和仓库都已经准备好，一次正常更新大致是下面这几步：

1. 修改 `oh-package.json5` 中的版本号和发布元数据。
2. 更新 README 和 CHANGELOG。
3. 构建 release HAR。
4. 检查 HAR 内部真正携带的 `oh-package.json5`。
5. 执行 `ohpm prepublish` 做本地预检。
6. 使用正确的 Publish ID 和私钥发布。
7. 查询线上版本，确认 latest 和元数据已经更新。

## 这次以及之前遇到的小坑

### README 和 CHANGELOG 不能随便应付

第一次发布时就发现，OHPM 包不能只有代码和一个简单的包配置。README 和 CHANGELOG 都要准备，README 的内容太简单、结构不完整或者不符合平台要求，也可能过不了审核。

README 至少应该让别人知道这个包是做什么的、怎么安装、怎么使用、支持什么环境以及有哪些注意事项。CHANGELOG 则要随着版本更新，不能发布了新版本，里面仍然完全看不到这次改了什么。

### 正式发布要明确构建 release HAR

HAR 也有 debug 和 release 的区别。至少在当前 AppCan engine 工程里，release 构建会生成 bytecode HAR，启用混淆并关闭 source map；debug 更适合本地调试，不适合作为正式发布包。

我最后使用的构建命令如下：

```powershell
node "<DevEco Studio安装目录>\tools\hvigor\bin\hvigorw.js" `
  --mode module `
  -p product=default `
  -p module=engine@default `
  -p buildMode=release `
  assembleHar `
  --no-daemon
```

这里最好显式写上 `-p buildMode=release`，不要依赖默认值。当前 engine 没有 product 专属依赖或 target 配置，所以 `product=default` 对包内容没有实质影响，但保留它可以让构建上下文更明确。

生成位置是：

```text
engine/build/default/outputs/default/engine.har
```

### Publish ID 不是仓库 ID

发布命令里有一个 `publish_id`，我一开始隔了一段时间也有点忘了它是干什么的。

它对应的是发布者身份，不是某一个软件包或仓库的唯一 ID。不同发布者应该使用自己的 Publish ID，并且它必须和用于签名的私钥相匹配。不能随便拿同事的 Publish ID，再配上自己的私钥一起用，它是跟你的账号关联的。

发布命令和本机 `.ohpmrc` 都可以指定publish_id。

### 私钥必须是带密码的加密私钥

这次还遇到一个很现实的问题：私钥需要设置密码的，但时间久了我当时差点想不起来了😂。

OHPM 发布使用的不是随便一把 SSH 私钥。当前客户端明确要求使用带非空 passphrase 的加密私钥，未加密私钥也会被拒绝。`key_path`、`key_passphrase` 和 Publish ID 必须互相对应，任何一个不正确都可能在签名阶段失败。

如果密码真的忘了，通常不能从私钥文件中反推出原密码。更稳妥的办法是重新生成一套满足要求的密钥，并在发布平台更新公钥或发布身份配置。

### author 缺少 URL 或 email，直接 HTTP 400

这次第一次上传时遇到的错误是：

```text
Original Error: HttpCode 400 The format of the OHPM package no author url or author email
```

最初上传的时候，我这边的 author 只是一个简单字符串，现在似乎平台要求严格了，要求它提供更完整的信息。类似于下面这样的对象：

```json5
"author": {
  "name": "AppCan鸿蒙开发组",
  "email": "<联系邮箱>",
  "url": "https://www.appcan.cn"
}
```

构建并不会检查这个错误，可以接受一个简单的 author 字符串，但发布中心仓库的时候，还会有自己的元数据校验，上传时会给出 400 错误。

### repository 的 URL 格式不对，又是一个 HTTP 400

修正 author 后，紧接着又遇到了第二个错误：

```text
Original Error: HttpCode 400 Invalid OHPM package repository in oh-package.json5.
The URL of the repository must start with https|http|ftp|rtsp|mms
```

`repository` 不能写成下面这种 Git SSH 地址：

```text
git@gitee.com:organization/repository.git
```

发布平台要求它以 `https://`、`http://` 等协议开头。麻烦的是，我们当前仓库还没有放在开源的仓库中，没有开源地址。最后采用了一个折中方案：字段继续保留，但填写 AppCan 官网地址，不填写真实私有仓库地址。

```json5
"repository": "https://www.appcan.cn"
```

从字段语义上说，这不如填写真正的代码仓库精确；但在代码暂时不公开的情况下，至少没有暴露私有仓库位置，也满足了服务端对 URL 格式的要求。以后如果 engine 正式开源，再把它改成公开仓库地址更合适。

### license 和 source_type 不是一回事

旧版本的 package 配置一直写着：

```json5
"license": "LGPL-3.0"
```

这相当于对外声明这个版本按 LGPL-3.0 授权。旧版本已经发布过，这个授权不能因为新版本改了字段就追溯撤销；但版权归属和第三方依赖允许的前提下，新版本可以采用新的许可方式。

因为当前阶段暂时不准备公开源码，所以 `1.0.1` 改成了：

```json5
"license": "LicenseRef-AppCan-Proprietary"
```

同时在包内放入对应的 `LICENSE` 文件。

不过只改 license 还不够。OHPM 发布命令还有一个 `source_type`，只允许 `open` 或 `closed`。当前客户端在没有显式指定时默认按 `open` 发布，所以闭源包还必须增加：

```text
--source_type closed
```

license 是使用许可声明，source_type 是发布平台记录的源码开放类型，两者不是一个概念，最好保持一致。

### 修改配置后一定别忘了重新构建 HAR

这个不算坑，属于脑子犯糊涂了。

如果先构建了 HAR，之后才修改版本号、author、repository 或 license，却没有重新执行 `assembleHar`，上传的仍然是旧包。因此每次调整发布元数据后，都应该重新构建 release HAR。

建议确认：

- `name` 和 `version` 是否正确；
- author 是否包含 name、email 和 url；
- repository 是否是合法 URL；
- license 是否是本次要发布的许可；
- `metadata.debug` 是否为 `false`；
- `metadata.byteCodeHar` 和 `obfuscated` 是否符合 release 预期。

### 发布前先跑 prepublish

真正上传前，可以先执行本地预检：

```powershell
ohpm prepublish "engine\build\default\outputs\default\engine.har"
```

成功时会看到类似输出：

```text
prepublish @appcan/engine 1.0.1 succeed.
```

它不能代替服务端的全部校验，author 和 repository 这类问题仍然可能在上传时才暴露，但至少可以提前发现一部分包结构和兼容性问题。

## 最后整理一套发布命令

先构建 release HAR：

```powershell
node "<DevEco Studio安装目录>\tools\hvigor\bin\hvigorw.js" `
  --mode module `
  -p product=default `
  -p module=engine@default `
  -p buildMode=release `
  assembleHar `
  --no-daemon
```

检查包内元数据：

```powershell
tar -xOf `
  "engine\build\default\outputs\default\engine.har" `
  "package/oh-package.json5"
```

执行预检：

```powershell
ohpm prepublish "engine\build\default\outputs\default\engine.har"
```

闭源方式发布：

```powershell
ohpm publish `
  --publish_registry https://ohpm.openharmony.cn/ohpm/ `
  --publish_id=YOUR_PUBLISH_ID `
  --source_type closed `
  "engine\build\default\outputs\default\engine.har"
```

发布完成后查询线上信息：

```powershell
ohpm info "@appcan/engine" `
  --registry https://ohpm.openharmony.cn/ohpm/
```



## 总结

每个错误都不算难，但这种“小地方不对就给一个 400”的体验确实有点折腾了。好在流程完整走过一遍以后，后面再更新版本，就可以按构建、检查、预检、发布、线上确认这条链路固定下来，不用每次都从头回忆了。
