---
title: 我的华硕K550JK笔记本黑苹果之路
categories: 
  - 瞎折腾系列
excerpt: 
date: 2017-03-21 11:14:13
tags: 
---

# 远景帖子
http://bbs.pcbeta.com/viewthread-1735666-1-1.html

# 吐槽
作为一个搞了三年Android开发的，对于苹果的东西，本来我是拒绝的。我原来的想法是，我宁愿花时间去学习React、Vue、Angular这些前端框架，或者node.js、Python、PHP这些后端框架，也不会去浪费时间鼓捣iOS。
然而公司突然有需求了，作为一个哪里需要哪里填的螺丝钉，再仔细想想：多学个东西，也没啥坏处，iOS虽然终归要死，但是至少没这么快，算了算了，学学吧（苹果爱好者和iOS开发者不要骂我，我躲起来了）~
有段话说得好：
```
PC上用Windows的，那是普通青年；
Mac电脑上用MacOS的，那是文艺青年；
Mac上用Windows的，那是2B青年；
那么最后，PC上用MacOS的，是什么？牛逼青年！
```
那么，作为一个爱折（zhuang）腾（bi）的好青年，黑苹果搞起来咯！

**吐槽最后说一句：本文仅仅是分享一下一个小白的经验而已，而且完美度并不是100%，因为目前已经不影响自己使用了，我得赶紧学iOS开发去了。经过这一番折腾，也对苹果系统的套路有了简单的了解，学习了。大神看起来肯定low的很，欢迎大神指点！**

# 材料
1. 华硕K550JK笔记本电脑，集成显卡是HD4600
2. 参考了一下远景前辈们的帖子：
 - http://bbs.pcbeta.com/viewthread-1716416-1-1.html
 - http://bbs.pcbeta.com/viewthread-1723942-1-1.html

# 安装过程中的一些坑

1. 进入系统后要记得关闭睡眠功能，否则可能无法开机（网上说的，估计是因为怕电源管理没驱动好，我暂时还没有开睡眠功能。一旦无法开机，可以使用Clover解除睡眠锁定状态即可，不可怕）；
2. 引导时出现错误，修改DSDT文件可能会解决问题；
3. 安装时提示请插入电源，右上角始终显示0电量，未充电状态，则需要配置DSDT文件驱动电源管理；
4. 各种帖子搞来的Clover的EFI，尽管型号相同，配置相同，但是也可能会出现各种问题（我就遇到了各种问题），但是至少能少踩点坑，也是个好事。如果你想直接伸手党，搞来就用，100%纯自动，那你只能跪求神明保佑了。否则还是提升自己动手能力比较靠谱~

# Clover config
特别配置：SSDT DropOEM true
不配置这个貌似无法生效SSDT里的一些改动（改了白改，我就被坑了）。

# DSDT
1. 从Windows10导出原始文件，用MaciASL打开
2. Fix ADBG
3. Fix PARSEOP_ZERO 
4. Remove _DSM methods
5. Rename B0D3 to HDAU

# SSDT

- SSDT
1. Remove _DSM methods
2. unexpected $end and premature End-Of-File
3. 2错误解决不了了，所以只好不进行1操作（2操作是1操作引起的，本来是无错的）

- SSDT1
1. 有编译错误，搞不定，直接不放了。

- SSDT2
1. 无改动

- SSDT3
1. 无改动

- SSDT4
1. Remove _DSM methods
2. Rename GFX0 to IGPU
3. Rename B0D3 to HDAU
4. Brightness Fix (Haswell)
5. haswell显卡补丁

- SSDT5
1. Rename GFX0 to IGPU

- SSDT6
1. Remove _DSM methods
2. Rename GFX0 to IGPU补丁
3. 编译会有syntax error,Unexpected PARSEOP_NAMESEG, expecting“(”的错误，删除后面的MUID REVI SFNC XRG0即可解决编译问题。

# 下面开始屏蔽独立显卡
1. 为何要屏蔽独立显卡？因为不屏蔽的话，前面折腾那一堆显卡补丁都是没用的，最终我这里还是显存只识别7MB，系统画面没有透明效果，打开launchpad都会卡半天。而屏蔽了独立显卡，加上之前的操作，驱动起来就完美了。

2. 为了确认一下要从哪个文件下手，我在控制台中执行了如下指令。结果如下：
```
zhangyipengdeMacBook-Pro:dslwin zhangyipeng$ grep -l Method.*_INI *.dsl
dsdt.dsl
ssdt4.dsl
ssdt5.dsl
ssdt6.dsl
zhangyipengdeMacBook-Pro:dslwin zhangyipeng$ grep -l Method.*_OFF *.dsl
ssdt5.dsl
ssdt6.dsl
zhangyipengdeMacBook-Pro:dslwin zhangyipeng$ grep -l GFX0 *.dsl
dsdt.dsl
ssdt4.dsl
ssdt5.dsl
ssdt6.dsl
zhangyipengdeMacBook-Pro:dslwin zhangyipeng$ grep -l B0D3 *.dsl
dsdt.dsl
ssdt4.dsl
zhangyipengdeMacBook-Pro:dslwin zhangyipeng$ grep -l -B3 _ADR.*0x00030000 *.dsl
ssdt4.dsl
zhangyipengdeMacBook-Pro:dslwin zhangyipeng$ grep -l -B3 _ADR.*0x00020000 *.dsl
ssdt4.dsl
```

3. 于是发现ssdt5和ssdt6这两个文件同时包含_INI和_OFF，因此根据R大大的帖子，我们知道，我们需要从这两个文件下手了；

4. dsdt需要修改_REG，ssdt5需要修改_OFF，ssdt6需要修改_INI。具体修改方法需要参考后面链接中翻译的R大的文章；

5. 整体编译生成aml，也可以挨个另存为ACPI Machine Language文件；

6. 拷贝到/EFI/CLOVER/ACPI/patched；

7. 重启，依然不行，怀疑可能是ssdt1文件所致，因为他曾经编译错误，删除之；

8. 重启，依然不行，查到需要修改config中SSDT DropOEM true；

9. 重启，it works!；

# 总结
折腾的我差点怀疑了人生，不过还是整完了，完结撒花。

# 目前剩余的没有正常驱动的组件
- [x] 安装过程中无法识别充电状态
```
20170219 15:00
修改了DSDT，打了Haswell的电源补丁就勉强通过了。其实，后来才知道，可以把电池拔了。。。
```
- [ ] 电量显示问题，无法识别充电状态，无法获取电量信息。
```
一开始没搞定，后来基本没有时间了，况且不影响使用，暂时没有动力去搞了，过一阵子再搞~（拖延症又犯了）
```
- [x] 声卡
```
20170219 19:00
增加了SSDT文件后，声卡貌似是完美了。
```
- [x] 无线网卡
```
更换了BCM43224，淘宝买的，免驱。
```
- [x] 显卡（显存显示7MB，有问题）
```
20170221 03:00
参考了http://www.cnblogs.com/eaglexmw/p/4908877.html 转载翻译的R大的帖子，然后结合实际情况修改DSDT和SSDT，详见上文。
```
- [ ] 内存（内存显示只有一根内存条，其实是两根）
```
20170219 19:12
发现config文件中配置的内存条信息中，8G的那根信息有误，修改正确重启，然而问题没有解决。
```
- [x] clover没有安装到硬盘中，目前需要U盘引导
```
挂载主硬盘的EFI分区，将Clover整体拷贝过去，更新一下EFI目录就OK了。
```
- [x] clover引导时，啰嗦（-v）模式无法关闭
```
发现并不是-v无法关闭，而是kext的Debug开关没关，所以Debug开关一共有三个，加上-v，一共四个地方。
```
- [x] 键盘按键需要调整一下（好像识别的不太对）
```
误判，挺对的，按键一切正常。驱动放了个Fn的kext驱动已经可以通过快捷键调节声音和屏幕亮度（貌似不需要Fn，直接按，这点有点奇怪，估计是因为macbook原版键盘上面就是直接的调整亮度和音量的）。
```
- [ ] 触摸板不灵敏
```
确实有点蛋疼，不支持mac触摸板的各种手势，进入偏好后触摸板是白板无内容。不过我本来就是个鼠标党，可以用鼠标，所以没继续研究触摸板。后期遗留任务，慢慢搞吧~
```
- [x] 屏幕显示偏蓝，需要调整色差
```
参考了http://bbs.pcbeta.com/viewthread-1723942-1-1.html 这位大哥的帖子，修改之后感觉区别不是特别大，也不知道x和y都是干啥的，为啥只修改y。后来直接去系统偏好设置里的显示器-->颜色，直接自定义颜色值效果，还可以增加个类似iOS中NightShift的颜色效果配置，挺不错的了，就算解决了。
```
