---
title: 从AppCan的github仓库拉取插件代码
categories: 
  - 瞎折腾系列
excerpt: 换了个Mac mini，新系统啥也没有，装了好多开发工具之后终于可以用了，但是有时候需要查看AppCan原生插件的源码的时候，发现蛋疼了，因为我之前都clone好了，直接看；而现在一用到就要去找地址，然后打开终端，输入一长串指令、复制粘贴，真麻烦。看着重复的指令，我想，干脆写个脚本精简一下吧！
date: 2017-03-27 00:37:12
tags: 
---

# 怎么想起来搞这个呢？

换了个Mac mini，新系统啥也没有，装了好多开发工具之后终于可以用了，但是有时候需要查看AppCan原生插件的源码的时候，发现蛋疼了，因为我之前都clone好了，直接看；而现在一用到就要去找地址，然后打开终端，输入一长串指令、复制粘贴，真麻烦。看着重复的指令，我想，干脆写个脚本精简一下吧！

# shell脚本简介

--> Shell本身是一个用C语言编写的程序，它是用户使用Unix/Linux的桥梁，用户的大部分工作都是通过Shell完成的。Shell既是一种命令语言，又是一种程序设计语言。作为命令语言，它交互式地解释和执行用户输入的命令；作为程序设计语言，它定义了各种变量和参数，并提供了许多在高级语言中才具有的控制结构，包括循环和分支。

它虽然不是Unix/Linux系统内核的一部分，但它调用了系统核心的大部分功能来执行程序、建立文件并以并行的方式协调各个程序的运行。因此，对于用户来说，shell是最重要的实用程序，深入了解和熟练掌握shell的特性极其使用方法，是用好Unix/Linux系统的关键。

这篇文章具体就不再介绍具体怎么写shell脚本了（反正截至到撰文为止，我也只是写了这么一个完整的shell脚本，也就是说，其实我也是新手），直接切入主题了~

# 用法

复制脚本内容，放入一个文本文件中，可以保存为gitcloneappcan作为文件名。然后命令行中使用chmod增加运行权限：
```
chmod +x gitcloneappcan
```

# 脚本内容

```
#!/bin/sh
echo -n "开始运行gitcloneappcan"
#参数1($1)为插件名称，参数2($2)为平台名称。下面判断是否为空
if [ "$1" == "" ];
then
    echo -n "请输入插件名称，例如gitcloneappcan uexXmlHttpMgr android"
    exit 0
fi
if [ "$2" == "" ];
then
    echo -n "请输入平台名称(小写字母)，例如gitcloneappcan uexXmlHttpMgr ios"
    exit 0
fi

if [ "$2" == "ios" ]
then
platform="iOS"
elif [ "$2" == "android" ]
then
platform="Android"
else
echo -n "你输入的平台$2不存在"
exit 0
fi

pluginName=$1
gitPath="https://github.com/$2-plugin/$pluginName.git"

# 此处定义了需要clone的目标地址，用的话修改为自己的本地地址
clonePath="/Users/zhangyipeng/Desktop/AppCanWork/MyGithub/$platform/$pluginName"

echo -n "开始clone: $pluginName 插件，平台为$platform"
echo -n "Github目标地址为：$gitPath"
echo -n "即将把以上仓库代码克隆到：$clonePath"
read -p "Press Enter to continue." inputContent
echo -n $inputContent
git clone $gitPath $clonePath
echo -n "结束运行"

```
