---
title: Android7-0中File-createTempFile接口异常的问题
categories:
  - Android折腾之路
excerpt: 一个小问题
date: 2017-03-25 19:59:00
tags:
---

# 问题现象

Logcat日志如下：
```
12-29 09:30:32.722: W/System.err(14750): java.io.IOException: Unable to create temporary file
12-29 09:30:32.723: W/System.err(14750): 	at java.io.File.generateTempFile(File.java:1773)
12-29 09:30:32.723: W/System.err(14750): 	at java.io.File.createTempFile(File.java:1860)
...
...
...
```

# 分析

也就是说，File.createTempFile这个方法有问题，无法写入文件。难道是Android 7.0增加了什么新权限？这么坑？
结果尝试换了一个方法，修改为File.createFile，竟然成功了，没有抛出任何异常。

# 结论

Android7.0中，File.createTempFile这个方法存在问题，创建文件时会抛出异常。

# 这两个接口有什么区别？

1. File.createTempFile生成的是一个可以自定义前缀和后缀的文件，当然你可以定义为tmp结尾，这样看起来就像是临时文件，此处没有深究是否有什么特别的地方；

2. File.createFile，就没啥特别的了，普通的创建文件；

3. 所以从表面上看，这两者没啥区别，倒是完全可以用File.createFile替代之。只不过为啥File.createTempFile会出现问题，就得去查一下Android7.0系统的源码，来比较一下实现方式的区别了。

记录一下，以备以后想要深究深究~
