---
title: Chrome64之后出的一个“坑”？pointerdown事件与touchstart事件的xy坐标不一致
categories: 
  - AppCan经验分享
excerpt: 
date: 2018-02-28 21:10:14
tags: 
---

# 这是Chrome64之后出的一个“坑”吗？

# 问题

首先介绍问题背景：此项目的App是基于AppCan混合方式开发，大量使用了WebView。

然后，有人反馈问题说，他带着自己的手机出了一趟国，回国后发现App用不了了，各种点击无反应。

# 分析

初看这个问题，真是一脸懵逼。这还跟出国有啥关系~还好作为一个喜欢折腾的程序猿，我已经把我的小米5splus刷入了谷歌服务，闲的蛋疼的时候，科学上网后去GooglePlay看了看，加入了Android WebView的beta组，然后就升级了WebView内核。咦，我也复现了问题。这样，就好理解了。客户出国就是国外的环境了，自然可以上GooglePlay，然后就自动升级了WebView内核。

查看了一下版本，原来我的系统默认WebView版本是62，升级后变为65beta版，也出现了总是点击无响应的问题。

# 看代码

追根溯源，我们看到，页面中使用了appcan.js，而appcan.js引用并改写了**zepto.js**，这里的点击事件都是通过监听Event来处理实现的。

节选代码如下：

```
$(document).on(touchstart, function(e){
    ...
})
```

1. 监听touchstart，记录初始触摸位置x1，y1；
2. 监听touchmove，不断记录移动后位置x2，y2；
3. 监听touchend，计算x1-x2和y1-y2的值，如果超过一定的大小，就认为发生了滑动，返回滑动事件；否则返回点击事件；

# 找到了问题

1. 经过日志发现，touchstart和touchend事件被触发了两次，有时候还会触发touchmove事件。第一次touchstart的坐标x1y1与最终touchend的x2y2的差距几乎忽略不计（因为我就是一个点击事件，手指没有移动，这是对的）。但是第二次的touchstart事件触发的x1y1十分离谱，小了好几百像素，但是target指向的却是同一个element。这样，后面计算两点的偏移量时，就会被认为是滑动事件，从而不会触发点击事件了。

2. 细看这个重复触发的事件，我发现，里面的e.type是不同的，第一次是pointerdown类型，第二次是touchstart类型。仔细看前面appcan.js的代码中，touchstart是个变量，它的值最终是"touchstart MSPointerDown pointerdown"，也就是监听了这三个事件，恐怕是为了兼容不同浏览器，但是却造成了事件都揉和在了一起处理。这样，当触发多种类型事件并且xy坐标却不同时，就在后来的touchend时值的比对造成了混乱，因而导致事件判断错误。

3. 什么是Pointer Event呢？最初是由微软定义的标准，希望能够统一所有的事件类型，并使用在了IE10中。后来GoogleChrome在55版本中也跟进兼容了这一个方案，而众所周知，Android4.4之后系统的WebView内核采用了Blink内核，与Chrome一致，并且可以做到跟随GooglePlay进行单独更新，所以新版Android的WebView中也基本上支持了这个事件。 

4. 通过回退WebView版本并比对现象发现，旧版WebView同样会监听到pointerdown事件和touchstart事件。 但是pointerdown事件的xy坐标是与touchstart事件相同的，因此没有暴露出这个问题。

# 解决方案

1. 就pointer的支持情况来看（https://caniuse.com/#search=pointer），iOS其实是不支持的，而在移动端，touch事件是很早以前就支持了。所以综合考虑，我们采取将touchstart等事件的监听中，只保留touchstart、touchmove、touchend事件，不再监听其他事件，防止混乱产生。
2. 另外考虑了一下，如果多种类型事件都想要监听的话，可以单独保存变量，单独处理，不要混用，以免出现这种不好排查的问题。
3. 至于pointer坐标监听为何现在跟touch坐标不一致了，暂时没有搞清楚原因，不知道是不是Blink内核的bug呢？还是说难道是pointer的坐标系有变化么？不得而知，有待观望。如果有大神知道，还请指点迷津，谢谢~
