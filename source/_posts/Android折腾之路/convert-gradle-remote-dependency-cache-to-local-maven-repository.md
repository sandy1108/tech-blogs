---
title: 将Gradle远程依赖缓存转换为本地Maven目录仓库
categories: 
  - Android折腾之路
excerpt: 公司自己的Android打包服务设计为离线使用，虽然环境可以联网，但是部署在某些客户的环境可能是不联网的。因此必须适应离线打包。
date: 2021-03-04 12:05:30
tags: 
---

## 问题起源

公司自己的Android打包服务设计为离线使用，虽然环境可以联网，但是部署在某些客户的环境可能是不联网的。因此必须适应离线打包。以前的升级都是直接从AndroidStudio的安装包里挖出来m2repository目录，其中就会包含一个当前版本推荐使用的gradle依赖库和gradle编译插件的相关资料，打包时直接依赖即可。但是自从去年开始AS中去掉了这个m2repository，似乎全部转为了在线，不再维护离线了。我还去安卓的官网看了一下，存在一个离线maven的下载链接，2G多，号称是最新的maven依赖库，让我高兴了半个小时，下载完毕了，然后发现其实里面的文件最后更新是2019年。告辞。

## 解决

### 参考文章

我全网搜索了各种各样的依赖库，发现谷歌确实是没有再提供新版的离线依赖包。只发现了一个跟我有相似需求的人，写了一篇文章（https://www.jianshu.com/p/050dd9fc2438），使用了python写了一个脚本，将gradle远程依赖的缓存，转换为了maven本地目录。虽然我曾经也这么想过，但是总觉得这个办法有点太笨了。但是现在看来，确实没办法了，有先行者了，那就跟随吧。

### 我的方案

由于我的python水平比较菜，而工作需求比较紧迫，不想再耽误时间了，所以我决定直接用我熟悉的java写一个小程序来完成这次批量转换。（为啥不用人家现成的py脚本？因为我怕一旦出了问题，或者跟我的需求有偏差，我还是要去研究他的python代码）。

### 两边目录结构对比

Gradle缓存目录结构：
```
~/.gradle/caches/modules-2/files-2.1/com.android.tools.build/gradle/4.1.2/d56e2eaa0cd496e8e0a2fc833be09fe7b9f1e0e6/gradle-4.1.2.jar
```
Maven目录结构：
```
~/LocalMavenConverted/com/android/tools/build/gradle/4.1.2/gradle-4.1.2.jar
```

对比发现，主要区别有两点：

1. Group名的部分，Gradle缓存直接用全名作为目录，而Maven则把每个分割点都拆成了目录
2. Gradle缓存的最后一步，把每个文件装到了一个随机数目录里。

### 思路步骤

遍历缓存目录内的所有目录，找到其中的每一个文件，并且通过字符串分割，拼接出新的输出路径，将文件拷贝过去。

### 代码

https://github.com/sandy1108/GradleCache2MavenDir
