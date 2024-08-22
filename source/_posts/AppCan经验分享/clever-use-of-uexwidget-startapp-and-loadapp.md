---
title: uexWidget的startApp和loadApp的妙用
categories: 
  - AppCan经验分享
excerpt: 最初的uexWidget.loadApp，在Android中对应的是拼装Intent，iOS对应的是openURL。懂原生开发的应该知道，这样就可以做各种各样的事情，扩展性很强。
date: 2017-12-20 19:27:12
tags: 
---

# uexWidget.startApp和uexWidget.loadApp接口介绍

最初的uexWidget.loadApp，在Android中对应的是拼装Intent，iOS对应的是openURL。懂原生开发的应该知道，这样就可以做各种各样的事情，扩展性很强。

后来觉得针对Android会有利用包名类名启动等方式，为了更清晰方便使用，提供了uexWidget.startApp接口。

# 两个接口的文档说明

Github地址：

https://github.com/AppCanOpenSource/appcan-docs-v2/tree/master/%E5%BA%94%E7%94%A8%E5%BC%95%E6%93%8E/uexWidget

# 常用方法举例说明

列举了一些常用的，以后有需要可以再补充。

## 打开应用市场

```
uexWidget.loadApp("android.intent.action.VIEW","","market://details?id=com.tencent.mm");

uexWidget.startApp("1", "android.intent.action.VIEW", '{"data":{"scheme":"market://details?id=com.tencent.mm"}}');

function openMarket(pkgName){
    uexWidget.startApp("1", "android.intent.action.VIEW", '{"data":{"scheme":"market://details?id='+pkgName+'"}}');
}
```

## 回到桌面(Android)

```
var info = '{"category":["android.intent.category.HOME"]}';
uexWidget.startApp(1,"android.intent.action.MAIN",info);
```

## 打开APN

```
uexWidget.loadApp("android.settings.APN_SETTINGS","","");
```
## 打开设置页面

```
uexWidget.startApp(0, "com.android.settings","com.android.settings.Settings");
```

## 打开浏览器

```
uexWidget.loadApp("android.intent.action.VIEW","","http://www.appcan.cn");

var main = "android.intent.action.VIEW";
var add = '{"data":{"scheme":"http://www.appcan.cn/"}}';
uexWidget.startApp(1, main, add);
```

## 安装本地apk

```
uexWidget.loadApp("android.intent.action.VIEW","application/vnd.android.package-archive","file:///sdcard/1123.apk");
```

## 调起百度导航

- iOS

```
uexWidget.loadApp("android.intent.action.VIEW","","bdapp://map/direction?origin=latlng:34.264642646862,108.95108518068|name:我家&destination=大雁塔&mode=driving&region=西安&src=yourCompanyName|yourAppName#Intent;scheme=bdapp;package=com.baidu.BaiduMap;end");
```

- Android

```
var main = "android.intent.action.VIEW";
var add = '{"data":{"scheme":"bdapp://map/direction?origin=latlng:34.264642646862,108.95108518068|name:我家&destination=大雁塔&mode=driving&region=西安&src=yourCompanyName|yourAppName#Intent;scheme=bdapp;package=com.baidu.BaiduMap;end"}}';
uexWidget.startApp(1, main, add);
```


