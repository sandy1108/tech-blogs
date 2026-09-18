---
title: 用 Android 手机作为 Mac mini 的麦克风：AndroidMic + BlackHole 使用分享
categories:
  - 瞎折腾系列
excerpt: Mac mini是没有麦克风的，所以这篇笔记记录如何把 Android 手机的麦克风声音传到 Mac mini，并让系统识别为麦克风输入源，从而直接在macmini中使用各路语音输入法等应用。
date: 2026-07-12 10:04:00
tags:
  - Android
  - macOS
  - Macmini
  - AndroidMic
  - BlackHole
---

> 众所周知，Mac mini是没有麦克风的，所以这篇笔记记录如何把 Android 手机的麦克风声音传到 Mac mini，并让系统识别为麦克风输入源，从而直接在macmini中使用各路语音输入法等应用。

## 写在前面

那么问题来了，我为啥不直接连接个蓝牙耳机或者蓝牙麦克风呢？嘿嘿，那当然是完全可以了。只不过搞这玩意的时候，我暂时没有闲置的蓝牙耳机，常用的蓝牙耳机已经跟两个手机配对了，可以快速连接，如果再连上macmini的话，就会导致手机连不上了，影响快捷的体验。但是如果有富余的蓝牙耳机的话，我这个需求必然还是蓝牙耳机或者蓝牙麦克风是最优解了。

## 使用的工具介绍

这里使用的开源项目是 [AndroidMic](https://github.com/teamclouday/AndroidMic)。Mac 端还需要一个虚拟音频设备，本文使用 [BlackHole](https://existential.audio/blackhole/) 2ch，把 AndroidMic 接收到的声音转成 MacOS 上其他应用可以选择的输入设备。

大概就是这么个意思：

```text
Android 手机麦克风
  -> AndroidMic Android 端
  -> Wi-Fi UDP
  -> AndroidMic Mac 端
  -> BlackHole 2ch
  -> 目标应用的麦克风输入
```

AndroidMic 负责采集并传输声音，但它传到 Mac 后还需要一个系统能够识别的音频设备。BlackHole 在这里扮演虚拟音频线的角色：AndroidMic Mac 端把声音输出到 BlackHole，Zoom、Discord、OBS、Audacity 或语音转文字工具再把 BlackHole 当作麦克风输入。

这里的 `2ch` 指两声道，不是“一路输入、一路输出”。对我这种只需要把一部手机接成麦克风的场景，2 声道版本已经够用，也没有必要用 64 声道的复杂路由，否则可能会占用资源过多。多声道的情况比较适用于多设备，或者配置了多个声音源，例如人声、游戏直播、背景音乐等等分别控制的情况。

## 工具的安装

### 1. BlackHole

可以用 Homebrew 安装：

```bash
brew install blackhole-2ch
```

也可以从 [BlackHole 官方下载页](https://existential.audio/blackhole/) 安装。安装完成后，可以在“音频 MIDI 设置”或系统声音设置里确认 `BlackHole 2ch` 已经出现。

### 2. AndroidMic Mac 端

打开 [AndroidMic Releases](https://github.com/teamclouday/AndroidMic/releases/latest)，下载与 Mac 架构对应的 DMG 文件并安装。

如果 macOS 阻止打开未签名应用，先确认 DMG 来自官方 Release，再按项目 README 的方式清除应用的隔离属性：

```bash
xattr -c /Applications/AndroidMic.app
```

然后重新打开应用。应用实际安装位置如果不是 `/Applications/AndroidMic.app`，需要把命令中的路径改成实际路径。

### 3. AndroidMic Android 端

从 [AndroidMic Releases](https://github.com/teamclouday/AndroidMic/releases/latest) 下载 APK，或使用 [F-Droid 上的 AndroidMic 页面](https://f-droid.org/packages/io.github.teamclouday.AndroidMic/) 安装。

## 我最后使用的连接方式

AndroidMic 支持 Wi-Fi TCP/UDP、USB ADB 和 USB Serial。本文实际使用的是 Wi-Fi UDP：手机和 Mac mini 连接到同一个局域网，Mac 端 AndroidMic 打开 Wi-Fi 监听，手机端填写 Mac 的 IP 和端口。

我一开始试过 TCP。手机为了省电主动断开后，电脑端监听也会停止；再次连接时还要重新处理连接状态。换成 UDP 后，Mac 端保持监听，手机端用的时候连接、用完断开即可，重新使用时直接发起连接，因为UDP协议不需要连接状态，所以实际上就是按需发包，服务端一直收包就可以了，对我这种偶尔用手机说几句话进行VibeCoding的场景更适合一些。

## 具体配置过程

打开 Mac 端 AndroidMic，选择 `TCP/UDP (WiFi)`，再切换到 UDP 模式。应用日志会显示监听的 IP 和端口，端口以当前版本实际显示的内容为准，不要照抄固定值：

```text
Mac mini IP：<当前局域网 IP>
端口：<AndroidMic Mac 端日志显示的端口>
```

如果 Mac 同时连接了 Wi-Fi、网线或虚拟网卡，要确认填写的 IP 是手机能够访问的那个地址。

在 Mac 端 AndroidMic 的 **Output Audio Device** 中选择：

```text
BlackHole 2ch
```

这里要注意，不要选择 Mac mini 的扬声器、显示器扬声器或耳机，否则手机声音会被直接播放出来，目标应用收不到 BlackHole 的输入。

高级音频参数可以先使用默认值。遇到杂音、断续或无声时，再尝试让 Android 端和 Mac 端的采样率、声道数、位深保持一致；常见组合是 `44.1 kHz` 或 `48 kHz`、单声道、16 bit 或 24 bit。它的高级参数里还包含了降噪之类的功能，相当全面，感兴趣的可以试试。

然后在 Android 手机端打开 AndroidMic，选择同样的 Wi-Fi UDP 模式，填写刚才记录的 Mac IP 和端口，授予麦克风权限和通知权限，再启动录音并连接。

最后在 Zoom、Discord、OBS、Audacity 或语音转文字工具中，把麦克风输入选择为 `BlackHole 2ch`。有的软件并没有选择声音输入源的功能，比如我用的微信输入法和豆包输入法之类的，那就可以去系统设置-声音，修改输入源为BlackHole即可。

对着手机说话，观察目标应用的输入音量条有变化，基本就说明链路已经通了。

## 最后的使用感受

这套方案的优点是不用额外买麦克风，手机也不用长期插在 Mac 上。Mac mini 负责运行 AndroidMic 和 BlackHole，手机需要说话时再连上即可。我实际用它接语音转文字和 AI 对话，效果已经够用了；如果要做正式录音、直播或长时间通话，还是应该使用更稳定的专用麦克风和有线连接。

它更像是一个解决临时需求的小方案：Mac mini 没有麦克风，但手边有 Android 手机时，花一点时间配置，就能把现有设备利用起来。

## 官方资料与下载入口

- [AndroidMic GitHub 仓库](https://github.com/teamclouday/AndroidMic)
- [AndroidMic 最新 Release](https://github.com/teamclouday/AndroidMic/releases/latest)
- [AndroidMic F-Droid 页面](https://f-droid.org/packages/io.github.teamclouday.AndroidMic/)
- [BlackHole 官方网站](https://existential.audio/blackhole/)
- [BlackHole GitHub 仓库](https://github.com/ExistentialAudio/BlackHole)
- [Homebrew：blackhole-2ch](https://formulae.brew.sh/cask/blackhole-2ch)
- [AndroidMic 官方 README：安装与连接说明](https://github.com/teamclouday/AndroidMic#setup-guide)
