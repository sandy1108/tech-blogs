---
title: Chrome56之后出的一个“坑”？Treat-Document-Level-Touch-Event-Listeners-as-Passiv
categories: 
  - AppCan经验分享
excerpt: 近期有很多开发者反馈说，在一些新版的Android系统中，AppCan应用的下拉刷新不管用了，拉不动。于是进行了排查，发现在下拉时WebView出现了下面的警告：
date: 2018-02-05 17:23:00
tags: 
---

# 遇到的问题

1. 近期有很多开发者反馈说，在一些新版的Android系统中，AppCan应用的下拉刷新不管用了，拉不动。于是进行了排查，发现在下拉时WebView出现了下面的警告：

 ```
 [Intervention] Unable to preventDefault inside passive event listener due to target being treated as passive. See https://www.chromestatus.com/features/5093566007214080
 ```

2. 查看了一下下面的堆栈信息，定位在了appcan.js中有个处理手势事件分发的逻辑位置，调用了preventDefault方法来拦截事件分发。但是看了上面的警告看来是失败了。那就点那个链接进去看看呗。

# 查到了原因

Chrome56+的新特性：Treat Document Level Touch Event Listeners as Passive。为了让页面滚动变得更为流畅，在DOM上处理touch事件时，会默认为是 passive:true。浏览器会忽略preventDefault()的JS请求，并弹出警告。而Android7.0之后，各大厂商跟进升级系统，内部都采用了56版本以上的WebView内核（这里补充一点，可能有些开发者不太了解：从Android4.4+开始，WebView内核已经跟系统独立了，Android用户如果拥有Google服务的话，可以通过GooglePlay更新GoogleWebView，而其内部的内核也采用了谷歌的Blink内核，将与Chrome同步更新。不过，大部分国内用户没有Google服务的话，只能跟随系统厂商一起更了。于是出现了一些用户反馈说，小米手机升级了新系统出现了XXX问题，华为手机升级了新系统出现了XXX问题。）

# 解决办法

说到底，其实就是谷歌Chrome为了增强性能，减少滑动时的卡顿问题而做的处理。然而我们这种的问题如何解决呢？由于此处问题是以AppCan应用为例子的，所以我们找到appcan.js文件中的add方法中修改下面代码：

```
if ('addEventListener' in element)
                element.addEventListener(realEvent(handler.e), handler.proxy, eventCapture(handler, capture))
```

改为：

```
if ('addEventListener' in element){
    var captureTemp = eventCapture(handler, capture) || {}
    if(!captureTemp.passive)
        captureTemp.passive = false
    element.addEventListener(realEvent(handler.e), handler.proxy, captureTemp)
}
```

然后重新集成并打包app，发现下拉刷新等手势操作已经没有问题了。

另外：AppCan JS SDK的修正版（1.1.3+）应该已经在路上了。

如果是跟AppCanJS无关的项目，也是一样，原理就是addEventListener的时候，要加第三个参数，其中要包含{passive:false}。

# 参考资料

1. Chrome官网说明：https://www.chromestatus.com/features/5093566007214080
2. 谷歌开发者网站说明：https://developers.google.cn/web/updates/2017/01/scrolling-intervention
3. 分析文章：http://blog.csdn.net/tengxy_cloud/article/details/52858742
