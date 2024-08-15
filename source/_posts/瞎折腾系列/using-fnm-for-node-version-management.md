---
title: 使用fnm来管理Node版本
categories:
  - 瞎折腾系列
excerpt: 之前在MacOS中用过nvm来切换node版本，但是在windows里我一直没有交给自动管理，因为他们自动管理一般都会放在C盘用户目录中，而我的Windows的C盘总是害怕满了，不想放太多东西，另外C盘放东西多了也不方便重装系统。
date: 2024-08-15 23:16:55
tags:
---
## 前言：手动切换Node版本其实也不麻烦

之前在MacOS中用过nvm来切换node版本，但是在windows里我一直没有交给自动管理，因为他们自动管理一般都会放在C盘用户目录中，而我的Windows的C盘总是害怕满了，不想放太多东西，另外C盘放东西多了也不方便重装系统。

最关键的是，在windows中我一直采取改目录文件名的方式来切换。即：

在某个NodeJS版本库目录（I:\0.DeveloperTools\NodeJS）中，我放了各种版本，以版本号命名目录，同时在目录内放一个txt文件，名字也是版本号，用来标识一下这个目录的node版本。然后系统环境变量PATH中配置了（I:\0.DeveloperTools\NodeJS\current）。这样想要用哪个版本的话，只需要把对应的版本目录名字改为current，原来的current改为版本号作为名字，甚至不用重启终端，就生效了，非常简单。

但是。。。（当然要有但是，否则就不会有这篇文章了）当我今天准备更新新版本Node时，发现官网会提供Windows环境下如何使用fnm下载Node版本，但是nvm却是灰色的。而且，每次手动下载，解压，命名，还是有点懒。干脆用上fnm吧！

![下载Node.js](image-download-nodejs.png)

## 迁移到fnm中管理（以Windows为例）

### 1. fnm下载安装

https://github.com/Schniz/fnm

直接在github仓库里下载最新Release版本，然后将这个exe文件配置到系统环境变量中即可。

### 2. 修改终端启动脚本，每次启动终端自动执行fnm环境变量的设置（以PowerShell为例，其他的看它的README吧）

用下面的命令打开这个启动脚本：

```
notepad $profile
```

如果你跟我一样记事本打开后提示文件不存在，那么执行下面的命令创建，然后再执行上面的命令打开：

```
if (!(Test-Path -Path $PROFILE)) {
    New-Item -Path $PROFILE -ItemType File -Force
}
```

启动脚本内写：

```
fnm env --use-on-cd --shell power-shell | Out-String | Invoke-Expression
```

** 它默认的目录是在C盘用户目录内，我要给它搬到我自定义的目录中，所以我写成如下内容： **

```
fnm --fnm-dir I:\0.DeveloperTools\NodeJS\fnm  env --use-on-cd --shell power-shell | Out-String | Invoke-Expression
```

### 3. 下载和使用指定版本，例如20

```
fnm use --install-if-missing 20
```

### 4. 迁移现有的Node版本

执行了上面的步骤后，fnm-dir中应该就有了内容了，可以看到它的目录结构就是v+版本号作为目录名，目录内就是node文件。那么我们就把现有的Node版本都按照这样的形式改名，放进来，就ok啦。

### 5. 列举fnm检查到的所有版本

```
fnm list
```

### 6. 设置全局默认Node版本，例如20

```
fnm default 20
```

### 其他

其他的就看一下Github仓库的README吧。


