---
title: MacOS中设置环境变量
categories: 
  - iOS摸索之路
excerpt: 如果使用了iTerm，搭配zsh终端使用，上面的bash_profile环境变量就又没用了，因为那是给默认终端bash用的。
date: 2017-04-26 21:22:16
tags: 
---



:<PATH N>
```

# .bash_profile示例
```
export ANDROID_PLATFORM_TOOLS=/Users/zhangyipeng/Library/Android/sdk/platform-tools 
export ANDROID_BUILD_TOOLS=/Users/zhangyipeng/Library/Android/sdk/build-tools/19.1.0
export ANDROID_DECOMPILE_TOOLS=/Users/zhangyipeng/Library/Android/sdk/AndroidDecompileTools

export PATH=$PATH:$ANDROID_BUILD_TOOLS:$ANDROID_PLATFORM_TOOLS:$ANDROID_DECOMPILE_TOOLS

```

# 小技巧
1. 拷贝路径时，可以在Finder中，进入到目标目录，然后点击设置按钮（那个齿轮状的），里面有个拷贝为路径的选项，即可把路径拷贝到剪贴板里了；

2. 目录中不存在profile文件的时候，除了用其他方式，也可以使用命令行创建：
```
//格式为： touch [path]

touch ~/.bash_profile

```

3. 如果使用了iTerm，搭配zsh终端使用，上面的bash_profile环境变量就又没用了，因为那是给默认终端bash用的。zsh终端打开时，会自动加载```~/.zshrc```文件，因此我们只需要在此文件中添加上面的环境变量就可以了。另一个方案是，少些重复代码，让这个文件直接引用上面设置的bash环境变量文件。即.zshrc文件末尾增加一行代码：

```
source ~/.bash_profile
```
