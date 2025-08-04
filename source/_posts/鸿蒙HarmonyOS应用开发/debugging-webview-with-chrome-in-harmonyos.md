---
title: 鸿蒙使用Chrome调试WebView
categories: 
  - 鸿蒙HarmonyOS启程之路
excerpt: 
date: 2025-08-04 22:28:54
tags: 
---

## 参考

https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/web-debugging-with-devtools


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

