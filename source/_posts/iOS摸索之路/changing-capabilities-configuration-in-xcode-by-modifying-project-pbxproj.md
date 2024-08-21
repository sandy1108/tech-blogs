---
title: Xcode通过修改project-pbxproj来更改Capabilities配置
categories: 
  - iOS摸索之路
excerpt: 
date: 2017-04-13 20:15:00
tags: 
---

# 遇到的问题

项目中遇到了一个iOS的问题，需要调试。拿到了工程之后，直接打开，开启自动配置证书，随便写了个我调试常用的BundleID，竟然迎来了一个个的红色叹号：


![1745248-1231b14f78e6676e.webp](1745248-1231b14f78e6676e.webp)


```
The 'Apple Push Notification' feature is only available to users enrolled in Apple Developer Program. Please visit 
https://developer.apple.com/programs/ to enroll.

Provisioning profile "iOS Team Provisioning Profile: com.wsgh.test2" doesn't support the Push Notifications capability.

Provisioning profile "iOS Team Provisioning Profile: com.wsgh.test2" doesn't include the aps-environment entitlement.
```

第一个错应该是根本原因。可是为什么呢？

# 排查和解决

字面意思是，推送功能只对注册开发者开放。仔细一想，这可以理解，因为我用的是我自己的个人开发者Team，穷鬼一个，并没有花99美元。

- 解决办法：

1.  切换到企业的开发Team中；

2.  继续使用自己的Team，但是关闭推送功能；

我比较喜欢用自己的，所以选择方法2。打开Xcode工程的配置界面，切换到Capabilities标签，找到Push。。。What？为啥没有？合着我现在不想用推送都不行了？

可是，作为一个爱折腾的程序狗，我表示不服：配置这个东西一定是有配置文件的，大家开发的时候因为xcode做的比较强大，各种配置都直接点UI了，一般也懒得关注配置文件在哪。

好，思路确定了，找吧。为了图省事，我把工程直接创建了个git仓库，用来比较工程文件的变动。

先切换到公司的Team，点开推送开关配置，然后git commit；

然后关闭推送开关，再看看文件变动。

发现两处不同：

1. 工程根目录的xcodeproj包中，右键显示包内容，找到了project.pbxproj，其中有个com.apple.push的值，开启推送时是1，关闭变成了0。估计这个是那个开关的配置文件；


![1745248-80adcfa6fd7df000.webp](1745248-80adcfa6fd7df000.webp)


2. Code Signing Entitlements配置的那个entitlements文件中，打开推送开关配置时，会自动生成一个APS Environment键值，这是iOS10推出之后，SDK新要求的一个配置权限的地方，这个键值就是推送的配置，关闭的时候，这个被删掉了。

![1745248-47c14b9ee0e0da12.webp](1745248-47c14b9ee0e0da12.webp)


至此，根据这个修改之后，这个工程就能用了。

# 延伸

从这个例子中，可以得知，project.pbxproj文件就是存放xcode的工程基本配置的地方，关于本工程的基本配置变更时，都会体现在这里；而iOS新增的权限配置，是在entitlements文件中规定的，而entitlements文件的路径，也是在project.pbxproj文件中指定了（键为：CODE_SIGN_ENTITLEMENTS）。

所以，以此我们可以在某些特殊情况下，直接查看工程文件就可以临时查看和修改工程的配置信息，从而达到装逼的效果，哦不对，是解决一些燃眉之急的效果~
