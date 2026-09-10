---
title: 鸿蒙使用Chrome调试WebView
categories: 
  - 鸿蒙HarmonyOS启程之路
excerpt: 在HarmonyOS开发中，我遇到了开启WebView调试后，`chrome://inspect`页面无法发现设备的问题。最新的DevEco Studio提供了一个便捷方法：在运行配置中勾选“Auto WebView Debug”即可自动转发端口。如果该方法失效或在旧版环境中，则需要通过`hdc shell`手动查找调试端口，并使用`hdc fport`命令将其转发至PC，从而解决调试问题。
date: 2025-08-04 22:28:54
tags:
  - HarmonyOS
  - ArkWeb
  - WebView
  - ChromeDevTools
  - 远程调试
  - DevEcoStudio
---

## 参考

https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/web-debugging-with-devtools

https://developer.huawei.com/consumer/cn/forum/topic/0208190002851211041?fid=0109140870620153026


## 代码层面前提

给WebView配置调试模式开启：

```
import { webview } from '@kit.ArkWeb';

...

webview.WebviewController.setWebDebuggingAccess(true);
```

打开chrome://inspect发现没有找到手机正在开启的app的webview页面。

## 步骤

0. **最新更新**：省流，如果你使用DevEco开发中进行调试，可以自动开启转发，后面的命令行步骤就可以省略了

方法：Edit Configurations--> Auto WebView Debug 打勾

然后再运行调试应用即可省略下面的步骤。但是这个方法启动应用后，如果一段时间不调试，调试端口就关闭了，需要重新运行。

1.  hdc shell进入需要调试的手机或者模拟器的shell环境；


```
hdc shell
```

2.  查看调试端口

```
cat /proc/net/unix | grep devtools
exit

```

3.  转发到电脑上

```
hdc fport tcp:9222 localabstract:webview_devtools_remote_38532
```

4.  如果还未发现设备，chrome中添加端口监听

```
localhost:9222
```

## 特殊情况记录更新

2025年8月初我的华为P70pro升级了鸿蒙5.1系统，结果发现开启多个WebView的时候，调试模式就会自动关闭，于是提了官方论坛询问，最后从回复中看意思是系统补丁中的小bug，可能会在后面修复。给了一个临时解决方案，就是代码层面pageBegin的时候先关后开：

```
webview.WebviewController.setWebDebuggingAccess(false);
webview.WebviewController.setWebDebuggingAccess(true);
```

详见论坛帖子：https://developer.huawei.com/consumer/cn/forum/topic/0208190002851211041?fid=0109140870620153026
