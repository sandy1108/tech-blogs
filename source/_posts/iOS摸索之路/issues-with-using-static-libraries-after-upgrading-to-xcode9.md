---
title: Xcode9升级后使用静态库的问题
categories: 
  - iOS摸索之路
excerpt: 升级了xcode9beta版后，编译出来的.a静态库，放到xcode9以下的xcode中进行编译，结果出现了下面的错误：
date: 2017-11-16 17:02:28
tags: 
---

# 问题详情

升级了xcode9beta版后，编译出来的.a静态库，放到xcode9以下的xcode中进行编译，结果出现了下面的错误：

```
Framework not found IOSurface for architecture arm64
```

```
Framework not found FileProvider for architecture arm64
```

# 问题原因分析

通过网上查询，貌似很久以前就有IOSurface这个库的存在，只不过以前是iOS私有API，不允许使用的。一搜全都是说私有API的。

现在时代变啦！个人猜测，iOS11的SDK中，苹果重构了Foundation中的一些模块，然后放到了FileProvider和IOSurface这两个库中，并开放了出来。而xcode8以上是可以自动依赖系统库的，所以在开发者无感知的状态下，编译.a静态库的时候，就自动依赖了了这两个新的系统库，而这两个库在xcode8中可是没有的，自然就出现了这个问题。

# 问题解决

1. 换用xcode8编译静态库，提供给xcode8使用。

2. 统统使用xcode9编译。

3. 把xcode9中对应的这两个系统库拷贝到xcode8中，后面附带StackOverFlow帖子传送门（理论可行，但不推荐，还是别折腾了，毕竟苹果是爸爸）。https://stackoverflow.com/questions/44450673/framework-not-found-iosurface-for-architecture-arm64
