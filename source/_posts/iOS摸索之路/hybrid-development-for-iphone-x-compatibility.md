---
title: iPhoneX适配之Hybrid开发
categories: 
  - iOS摸索之路
excerpt: 
date: 2018-03-28 10:17:13
tags: 
---

# Hybrid开发针对iPhoneX的适配之路

2017年9月苹果发布了iPhoneX。它的“新发型-齐刘海”很让程序猿们揪心。如何适配呢？由于AppCan的客户们逐渐的开始遇到iPhoneX适配问题了，那这里就以AppCan平台为例子，记录和分享一下适配过程。

Hybrid开发是指应用内既有原生也有H5页面（本地或在线）。H5页面通常使用系统提供的WebView来呈现。

# 适配前状况

1. 打包使用Xcode8，不支持iOS11新特性，也无法支持iPhoneX等新机型；
2. App运行在iPhoneX上的表现是：系统可能识别到app使用了iOS10的SDK打包，因此自动留下了比普通的安全区更“安全”（更宽）的黑边，极其难看；
3. 等等。

# 基本方面适配

1. AppIcon：（不过这跟iPhoneX适配无关）图标需要加入1024×1024的图标了，要不然上AppStore会被拒；

2. 启动图：需要加入1125×2436分辨率的针对iPhoneX分辨率的启动图。加入正确分辨率的启动图之后，运行在iPhoneX时，可以看到不会有那种更“安全”（更宽）的黑边了，但是还是有黑边（也不是黑边，其实就是露出了后面的View背景），不是全屏的，而是似乎系统自动将WebView显示的内容放在了正常的安全区域内（即上空44px，下空34px）；

# WebView适配（以UIWebView为例）

有两种思路，一种是将UIWebView本身放在安全区域内，另一种是UIWebView铺满全屏，H5代码中对安全区进行适配。

## UIWebView本身放置在安全区域内

- 其实不用刻意去做，上文中已经提到，发现系统已经把WebView的大小调整在了安全区内，可能算是一种自动适配的手段吧。那既然如此，我们只需要调整一下WebView后面的View的背景颜色，让它与WebView的内容中上边缘和下边缘的颜色风格一致就看起来没问题了。

- 但是，在AppCan框架中，我们发现打开插件和浮动窗口时，H5代码中传入的x和y都是相对于WebView页面内容的，而实际上addView时却是相对于外层的全屏View框架，这就导致了addView错位上移。于是我们采取统一给x和y添加偏移量来自动适配安全区位置。

- 这样的适配方式，简单粗暴，强行留边，适用于一般情况。不过如果H5代码中需要一些花哨的适配，或者一些更加灵活的适配方式，那就需要下面的方式了。

## UIWebView铺满全屏，H5进行进行适配

- 由于目前UIWebView看起来好像是自动调整了自身的大小来把自己放在安全区内，所以，为了让它铺满全屏，我们还需要取消它的自动适配功能。方法是：在所有的H5页面中的meta标签加入下面的viewport-fit=cover属性：

```
<meta name="viewport" content="...,viewport-fit=cover" />
```

- 可是，如果我们想要默认把所有页面都应用viewport-fit=cover的效果，就必须挨个页面改一遍么？我想默认就铺满全屏可以么？有个办法：我们在自定义的UIWebView初始化时加入下面代码即可：

```
    //设置webView自带的scrollView，使得view充满屏幕
    if(@available(iOS 11.0, *)){
        [self.scrollView setContentInsetAdjustmentBehavior:UIScrollViewContentInsetAdjustmentNever];
    }
```

- 现在UIWebView铺满全屏了，H5代码如何处理呢？好心的苹果良心发现，在iOS11系统中的最新的WebKit内核中加入了一些常量，定义了what is 安全区，可以用于css中做padding使用，从而把内容限定在安全区内。示例如下：

```
        .iphonex-compat {
            /* iOS11.0-iOS11.1 */
            padding: constant(safe-area-inset-top) constant(safe-area-inset-right) constant(safe-area-inset-bottom) constant(safe-area-inset-left);
            /* iOS11.2+ */
            padding: env(safe-area-inset-top) env(safe-area-inset-right) env(safe-area-inset-bottom) env(safe-area-inset-left);
        }
```

- 如果此时UIWebView没有使用开始的那两种方式进行铺满全屏的操作，则上述的constant或者env拿到的这些变量则均为0。而如果不是iOS11的系统，上述CSS function则是未定义状态，不起效果。

- 另外，iOS 11.2 新增了 CSS function: min() 和 max()，用于横竖屏切换时，动态修改上下左右的padding（因为可能自己的页面本身就有padding，这是就要看谁大谁小了）。例如：

```
padding-left: max(12px, env(safe-area-inset-left));
```

# 参考文章

极速适配 iPhone X 秘笈：https://mp.weixin.qq.com/s/sCfNNxiX6v8Ak9JaOwljuw
