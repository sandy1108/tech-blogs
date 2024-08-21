---
title: iOS开发的另类神器：libimobiledevice开源包
categories: 
  - iOS摸索之路
excerpt: 
date: 2017-04-04 01:54:30
tags: 
---

# 简介

libimobiledevice又称libiphone，是一个开源包，可以让Linux支持连接iPhone/iPod Touch等iOS设备。由于苹果官方并不支持Linux系统，但是Linux上的高手绝对不能忍受因为要连接iOS设备就换用操作系统这个事儿。因此就有人逆向出iOS设备与Windows/Mac Host接口的通讯协议，最终成就了横跨三大桌面平台的非官方版本USB接口library。经常用Linux系统的人一定对libimobiledevice不陌生，但是许多Windows和Mac用户也许就不知道了。事实上，它同iTools一样，都是可以替代iTunes，进行iOS设备管理的工具。因为源码是开放的，可以自行编译，所以对很多开发者而言可以说更为实用。

官方github地址：https://github.com/libimobiledevice/libimobiledevice

最后还有一点，作为一个前Android开发，习惯使用adb命令各种调试，转了iOS怎么能没有这种工具，而去使用iTunes和iTools呢？对此零容忍！

# 快速直接安装libmobiledevice的方法

- 在MacOS下安装可以使用brew，类似Ubuntu中的apt-get

```
# 高版本brew会警告，不允许使用sudo执行brew，有风险。

brew update
brew install libimobiledevice

# libimobiledevice中并不包含ipa的安装命令，所以还需要安装
brew install ideviceinstaller
```

- Ubuntu下安装需要添加一个新的软件库，里面包含了libimobiledevice

```
sudo add-apt-repository ppa:pmcenery/ppa
sudo apt-get update
apt-get install libimobiledevice-utils
sudo apt-get install ideviceinstaller
```

# 常用功能

1. 安装ipa包，卸载应用
```
//命令安装一个ipa文件到手机上，如果是企业签名的，非越狱机器也可以直接安装了。
ideviceinstaller -i xxx.ipa

//命令卸载应用，需要知道此应用的bundleID
ideviceinstaller -U [bundleID]
```

2. 查看系统日志
```
idevicesyslog
```

3. 查看当前已连接的设备的UUID
```
idevice_id --list
```
4. 截图
```
idevicescreenshot
```
5. 查看设备信息
```
ideviceinfo
```
6. 获取设备时间
```
idevicedate
```
7. 设置代理（也好像是端口转发的工具，具体能利用它干啥还没试过）
```
iproxy
```
8. 挂载DeveloperDiskImage，用于调试
```
ideviceimagemounter
```
9. 获取设备名称
```
idevicename
```
10. 调试程序（需要预先挂载DeveloperImage）
```
idevicedebug
```
11. 查看和操作设备的描述文件
```
ideviceprovision list
```

# ideviceinstaller安装ipa报错（已经支持iOS12）

## 错误1

```
"Could not connect to lockdownd. Exiting."
```

出现这个问题一般是因为新版操作系统的通信协议可能有些微调，参考下面的stackoverflow的帖子，可以通过下面的命令尝试更新使用最新的libimobiledevice构建版本。

>http://stackoverflow.com/questions/39035415/ideviceinstaller-fails-with-could-not-connect-to-lockdownd-exiting

>The best solution here is to get the latest libimobiledevice, which has a fix for this particular issue:

```
brew uninstall ideviceinstaller
brew uninstall libimobiledevice
brew install --HEAD libimobiledevice
brew link --overwrite libimobiledevice
brew install ideviceinstaller
brew link --overwrite ideviceinstaller
```

## 错误2，进行上面步骤时出现

```
A recent change to libimobiledevice bumped the constraint on libusbmuxd to >= version 1.1.0. The current usbmuxd homebrew package is version 1.0.10.
As a result, homebrew --HEAD installs of libimobiledevice no longer build without a --HEAD install of usbmuxd.

```

从意思上看，应该是其中用到的usbmuxd库更新了，我们系统中的不够新。那咱们就给他升级一下呗：

```
brew update
brew uninstall --ignore-dependencies usbmuxd
brew install --HEAD usbmuxd
brew link --overwrite usbmuxd
```

更新了usbmuxd库之后，可以重复错误1中的解决办法，重新安装libmobiledevice即可正常使用。

## 错误3，可能会提示你需要配置github的accessToken，按照提示操作即可

## 错误4，可能会提示man手册目录的权限问题，按照提示操作即可

# 挂载文件系统工具：ifuse

- ifuse是一个依赖libimobiledevice库的工具，所以必须首先安装libimobiledevice

- 首先去 https://osxfuse.github.io/ 下载fuse for macos的库。

- 然后github上clone下载ifuse最新源码到本地（自己决定放哪）：
```
//cd 到要安装的目标路径，然后：

git clone https://github.com/libimobiledevice/ifuse.git
```

- 进入clone好的目录，执行：
```
//将源码在本机编译：

./autogen.sh
./configure
make

//执行脚本ifuse到系统终端（其实也可以不用，直接去src中运行也可以）
sudo make install
```

- 挂载媒体文件目录：
```
//注意，此处的挂载点必须要真实存在，需要预先创建好目录，否则挂载失败

ifuse [挂载点]
```
- 挂载某应用的documents目录
```
ifuse --documents [要挂载的应用的bundleID] [挂载点]

//注意，iOS 8.3之后要求应用的UIFileSharingEnabled权限要开启，否则可能没有权限访问，会有如下的错误提示

ERROR: InstallationLookupFailed
The App 'com.wsgh.test' is either not present on the device, or the 'UIFileSharingEnabled' key is not set in its Info.plist. Starting with iOS 8.3 this key is mandatory to allow access to an app's Documents folder.
```
- 挂载某应用的整个沙盒目录
```
ifuse --container [要挂载的应用的bundleID] [挂载点]
```

- 获取bundleID
```
ideviceinstaller -l
```

- 卸载挂载点
```
fusermount -u [挂载点]
```

- 如果是越狱的设备，并且配置好了，可以使用下面命令挂载整个iphone文件系统（暂时没试过，还没有开始研究越狱设备）
```
ifuse --root [挂载点]
```

- 详细说明，可以进入ifuse的github主页查看原版文档
```
https://github.com/libimobiledevice/ifuse
```

# 学习交流群

想加入libimobiledevice交流群的话，简书私信找我。
