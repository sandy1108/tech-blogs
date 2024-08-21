---
title: iOS分析崩溃日志的一些途径
categories: 
  - iOS摸索之路
excerpt: 
date: 2018-01-12 19:30:44
tags: 
---

# 准备

1. 崩溃日志。实时日志的抓取或者是从设备导出的崩溃日志均可。
2. Mac系统电脑，打开终端备用（暂时不确定dwarfdump命令是否只有mac才有，后续跟进）。
3. 要分析的app的ipa包打包时生成的dSYM文件。

# 崩溃日志示例

我在控制台抓取到了一个崩溃日志

# 步骤和用到的终端命令

1. 查看DemoApp.ipa的UUID：

```
先将DemoApp.ipa解压，进入Payload，看到DemoApp.app

终端在当前目录执行：
dwarfdump --uuid DemoApp.app/DemoApp
```

2. 查看dSYM文件的UUID:

```
dwarfdump --uuid DemoApp.dSYM
```

3. 上面两步，得到的应该是相同的uuid，这样我们就可以判定他们是对应的，可以进行下一步分析了。

4. 根据id查询：

```
//lookup后面跟id，--arch后面跟要查询的id所在的CPU平台（在本文中指崩溃发生的平台），最后跟dSYM文件路径
dwarfdump --lookup 0x000cf358 --arch armv7 yourappname.app.dSYM
```

5.dSYM文件，如果在本地xcode运行的话，直接去products中右键show in finder 也能找到
