---
title: Transporter上传ipa出错
categories: 
  - iOS摸索之路
excerpt: 
date: 2021-03-05 11:09:12
tags: 
---

## 问题

现在iOS应用上传AppStore的工具，不再集成在Xcode中了，需要单独去AppStore下载Transporter应用。

然而有一次，在某电脑上使用Transporter上传AppStore应用，结果报错，日志如下：

```
[2021-01-21 16:28:00 CST] <main>  INFO: ParseError at [row,col]:[5588,37]
Message: XML document structures must start and end within the same entity. Exception's name: javax.xml.stream.XMLStreamException, Exception's message: ParseError at [row,col]:[5588,37]
Message: XML document structures must start and end within the same entity.
java.lang.reflect.InvocationTargetException
 at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
 at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
 at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
 at java.base/java.lang.reflect.Method.invoke(Unknown Source)
 at com.apple.transporter.launcher.Application.start(Application.java:453)
 at com.apple.transporter.launcher.Application.main(Application.java:950)
Caused by: com.apple.transporter.bootstrap.BundleNotFoundException: bundle=[org.osgi.util.function] version=[1.1.0,2.0.0) not found.
 at com.apple.transporter.bootstrap.BootstrapperPhase1.downloadNeededBundles(BootstrapperPhase1.java:267)
 at com.apple.transporter.bootstrap.BootstrapperPhase1.bootstrap(BootstrapperPhase1.java:97)
 at com.apple.transporter.bootstrap.BootstrapperPhase1.bootstrap(BootstrapperPhase1.java:59)
 at com.apple.transporter.launcher.Launcher.launchBootstrapper(Launcher.java:37)
 ... 6 more
```

从日志来看，苹果的Transporter竟然其中还有Java代码，有点意思。

但是就日志来看，应该是缺少依赖库，无法运行，但是这跟我们的ipa没有什么直接关系。后来发现很多人也遇到了类似问题，可能是Transporter初始化的时候会下载一些远程依赖库，但是貌似网络出错之类的导致加载失败了，之后就无限出错，闪退，应该属于bug吧，以后应该会修复的。

## 目前解决办法

用户目录下/.itmstransporter 目录（这应该是它的缓存目录）改个名，或者丢到废纸篓。然后重新启动Transporter。
