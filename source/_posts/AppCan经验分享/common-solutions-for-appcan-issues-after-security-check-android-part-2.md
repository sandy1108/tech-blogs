---
title: AppCan应用在安全检测后的常见问题解决方案之Android篇（下）
categories: 
  - AppCan经验分享
excerpt: 具有下载apk功能的组件存在导出漏洞，并且未对组件调用者进行校验。攻
date: 2017-04-14 09:47:51
tags: 
---

# 继续问题列举

11. 检测项目：下载任意APK漏洞

- 关键词

apk

- 详细报告

具有下载apk功能的组件存在导出漏洞，并且未对组件调用者进行校验。攻击者可利用导出组件的手段下载攻击者指定的任意apk文件，并且在下载过程中伪装apk文件的下载信息，例如图标、描述等，导致用户被诱导下载安装恶意应用。
```
1)检测出此APP有申请使用网络权限 
2)扫描代码，存在application/vnd.android.package-archive代码
检测出此APP存在下载任意APK漏洞风险
```

- 解决方案

此问题经查并没有找到有关下载apk方面的代码存在在外部直接调用组件导致下载错误apk的地方。待查。

这个问题的意思应该是，代码中存在一个组件，比如activity或者广播之类的，能够直接在外面app唤起，然后传入合适的参数，就可以直接下载对应的apk了，代码中未校验发起方的身份。

12. 检测项目：SecureRandom猜解漏洞

- 关键词

SecureRandom

- 详细报告

在SecureRandom生成随机数时，如果我们不调用setSeed方法，SecureRandom会从系统的中找到一个默认随机源。每次生成随机数时都会从这个随机源中取seed。在linux和Android中这个随机源位于/dev/urandom文件。 如果我们在终端可以运行cat /dev/urandom命令，会观察到随机值会不断的打印到屏幕上。        在Android 4.2以下，SecureRandom是基于老版的Bouncy Castle实现的。如果生成SecureRandom对象后马上调用setSeed方法。SecureRandom会用用户设置的seed代替默认的随机源。使得每次生成随机数时都是会使用相同的seed作为输入。从而导致生成的随机数是相同的。 该漏洞存在于Android系统随机生成数字串安全密钥的环节中。该漏洞的生成原因是对SecureRandom类的不正确使用方式导致的。 翻看Android的官方文档会发现。对于SecureRandom类的构造函数SecureRandom(byte[] seed)和SecureRandom#setSeed方法有一段安全性提醒：“Seeds this SecureRandom instance with the specified Seeding SecureRandom may be insecure”

```
总共检测SecureRandom【1】条 ： 

    const-string v9, "RSA"
    invoke-static {v9}, Ljava/security/KeyPairGenerator;->getInstance(Ljava/lang/String;)Ljava/security/KeyPairGenerator;
    move-result-object v1
    .line 162
    .local v1, "keygen":Ljava/security/KeyPairGenerator;
    new-instance v7, Ljava/security/SecureRandom;
    invoke-direct {v7}, Ljava/security/SecureRandom;-><init>()V
    .line 164
    .local v7, "random":Ljava/security/SecureRandom;
    const-wide/16 v10, 0x3e8    invoke-virtual {v7, v10, v11}, Ljava/security/SecureRandom;->setSeed(J)V
com/hisun/b2c/api/cipher/RSA.smali

```

- 解决方案

正确使用SecureRandom类不要使用自定义随机源代替系统默认随机源除非有特殊需求，在使用SecureRandom类时，不要调用以下函数：\r
 SecureRandom::SecureRandom(byte[] seed)\r
SecureRandom::setSeed(long seed) SecureRandom::setSeed(byte[] seed)

此问题中，问题出现在com/hisun/b2c/api/cipher/RSA这个类所在的插件中，初步看并不是官方插件，需要检查这个自定义插件中的用法，按照解决方法中的说明修改。

13. 检测项目：SSL证书使用规范检测

- 关键词

SSL、https、证书

- 详细报告

程序中在使用HTTPS 请求时开发者在封包传递时虽然使用了 SSL 加密链接，如果没有进行严格校验 SSL 证书，造成了可被抓包分析明文数据、修改封包重发的危险漏洞
```
总共检测资源文件中包含SSL关键类【7】条

检测代码中包含SSL关键类路径：
检测到包含ssl关键类路径
[/smali/com/amap/api/services/core/k$b.smali]
检测到包含ssl关键类路径
[/smali/com/baidu/lbsapi/auth/h.smali]
检测到包含ssl关键类路径
[/smali/org/zywx/wbpalmstar/platform/push/report/PushReportHttpClient$ESSLSocketFactory$1.smali]
检测到包含ssl关键类路径
[/smali/org/zywx/wbpalmstar/plugin/uexmultiHttp/HX509TrustManager.smali]
检测到包含ssl关键类路径
[/smali/com/baidu/location/ai.smali]
...
此处省略【2】条数据
...

```

- 解决方案

引擎：

2015年之后的Android引擎版本已经剔除了包含漏洞的代码，替换为可配置是否校验的安全https请求方式。需要更换最新版的引擎。

插件：

1. uexXmlHttpMgr插件，请使用最新版本（如果引擎没有升级4.x，插件就要用3.x的最新版本），以解决此问题。

2. 上述问题中还发现在高德地图、百度SDK处存在的漏洞，需要更新uexBaiduMap、uexGaodeMap、uexLocation这些插件版本为最新，尝试修复此问题。
