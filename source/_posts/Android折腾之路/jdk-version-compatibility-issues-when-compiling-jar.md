---
title: 编译生成jar包时，jdk版本兼容问题
categories: 
  - Android折腾之路
excerpt: 需要生成jar包，提供给安卓编译使用，但是使用的安卓编译工具（buildTools）较低（小于19），不支持jdk7的jar包，此时可以通过如下几种方式解决。
date: 2017-06-13 15:55:55
tags: 
---

# 问题

需要生成jar包，提供给安卓编译使用，但是使用的安卓编译工具（buildTools）较低（小于19），不支持jdk7的jar包，此时可以通过如下几种方式解决。

# javac命令

```
javac -source 1.6 -target 1.6 xxx.java
```

# ant脚本

```
//修改javac标签中的 source 和 target为 1.6

<javac executable="xxx/javac" source="1.6" target="1.6"/>
```

# gradle脚本

```
//脚本中增加如下代码

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_6
        targetCompatibility JavaVersion.VERSION_1_6
    }
```

# 总结

这个问题只是举了个例子，事实上，今后可能还会遇到类似的，比如jdk8，jdk9，安卓buildTools还没升这么快，也不支持，这种情况下，同上。感觉应该还是挺有用的，总结了一下，做个记录。
