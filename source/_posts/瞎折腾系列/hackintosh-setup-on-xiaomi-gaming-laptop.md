---
title: 小米游戏本吃黑苹果的姿势记录
categories: 
  - 瞎折腾系列
excerpt: 
date: 2018-05-14 16:53:28
tags: 
---

# 写在前面

0. 我这是7代CPU的顶配版，不是后来8月份发布的8代CPU的增强版哦。8代CPU的版本，目前我这里是没有机器的，可以等待其他大佬适配。

1. 我的8G大小的山寨老U盘，经历过去年我的老华硕K550JK笔记本的“黑苹果之战”，功勋显著。虽然只是USB2.0，速度慢的不行，但还是继续让他继续“参战”，帮我完成小米游戏本的“黑苹果之战”。

2. 小米游戏本的购买故事：2018年4月13日幸运的抢到了首发，4月15日工作原因出差去了成都，4月16日发货，显示在成都发往北京（虽然特别期待，但是还是忍住没去顺丰拦截，随他去吧），4月22日别人帮忙收货，4月29日回京终于见到了期盼已久的小米游戏本。几经周折，还算好吧。

3. 拿到本本，转移老笔记本数据，然后玩了几次游戏，感觉有点空虚。还是觉得应该鼓捣鼓捣黑苹果，因为毕竟我买了这个本还要工作的，开发Android和iOS，这可以传说中“可以带去上班的游戏本”。工欲善其事，必先利其器啊。可是悲剧的是，现在全网也搜不到一个小米游戏本的黑苹果帖子。可能毕竟是用户太少了吧，想要买到雷布斯的产品还是看运气的。所以，赶紧自己搞起来吧。

4. 简书文章和Github近期还会持续更新，直到我觉得折腾烦了，或者够用了为止。

5. 哦，还有，我可不是大神，我只是一个小程序猿+黑苹果爱好者而已。所以重点来了，**欢迎大神指正，万一文章里有什么不对的，可以跟我说，以免误导别人，也算是帮帮小弟我。**

## Clover分享（伸手党福音）

断断续续折腾了一周多，我还是无私奉献出来了。强烈希望能够抛砖引玉，吸引更多大神帮忙一起解决问题和完善小米游戏本的黑苹果。

- Github: https://github.com/sandy1108/Clover-XiaoMi-GameLaptop
- QQ讨论群：756750452（是小米游戏本吃黑苹果的兴趣群，不是纯伸手党群哈，不喜勿加）
- 简书文章：https://www.jianshu.com/p/005712b26a35
- 远景帖子：http://bbs.pcbeta.com/viewthread-1785928-1-1.html

## 别人愿意帮你是情分，不愿意帮你是本分

**QQ群或帖子直接伸手求助的兄弟，我不反对。但是，如果你没有得到帮助，那是非常正常的，可别乱抱怨。因为每个人的时间都很宝贵，而黑苹果本身就是很耗费时间。非要做纯粹伸手党的，请移步万能的某宝。**

# 笔记本具体配置（主要关注的几个点）

- 小米游戏本（顶配版）
- CPU：i7-7700HQ
- 集显：HD630
- 独显：GTX1060移动版
- 16G内存
- 分辨率：1920*1080
- 无线网卡：Intel Dual Band Wireless-AC 8265

# MacOS安装镜像写入U盘

1. 先注意，此处由于我的U盘是“身经百战”的，因此已经分好了EFI分区。如果您使用的是没装过黑苹果的U盘，可能你还没分EFI分区，进行下面步骤之前，要先分出来EFI分区，留着以后作为引导区放Clover用。我在这里暂时不介绍了，有需要的可以自行查一下，不难。以后心情好也可能会补充一下。

2. 我从我的老华硕本的黑苹果系统（10.12.3）中，打开AppStore，搜索High Sierra，就能搜到新系统的入口，点击下载，等待，然后最新的Mac系统的安装app就下载到应用程序中了。目前我是下载了MacOS10.13.4。下载好即可，别安装。

3. 进入系统的应用程序目录，找到这个“安装 macOS High Sierra.app”的app，我先直接拷贝到别处备用，以免安装过程中造成文件的一些改动。

4. 用下面的终端命令执行安装盘创建操作。注意：下面是我的环境，其中的三个路径相关的参数都需要替换。第一个路径是找到上一步拷贝出来的app里的这个createinstallmedia，第二个路径是你的U盘分卷路径（特别说明，如果你想写入硬盘的一个分区，这里直接指定硬盘的分卷路径即可，分区空间大约8G足够），第三个路径是上一步的app包。

```
sudo /Users/zhangyipeng/Downloads/Mac软件/Install\ macOS\ High\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/Install\ macOS\ Sierra --applicationpath /Users/zhangyipeng/Downloads/Mac软件/Install\ macOS\ High\ Sierra.app --nointeraction
```

10.14之后，其中的--applicationpath可以删掉，过时了。如果10.14.4以后的话就必须删掉，否则制作会报错。即：

```
// ${installAppPath}替换为安装app的路径
// ${dstVolume}替换为目标安装盘分区卷标（即你的安装盘要制作到哪个分区的卷标）

sudo ${installAppPath}/Contents/Resources/createinstallmedia --volume ${dstVolume} ${installAppPath} --nointeraction
```

5. 如果是10.12以前的镜像，那就制作完成了，但是由于10.13系统有变化，需要再添加一个引导文件。具体可以参考这个帖子（ http://bbs.pcbeta.com/viewthread-1780071-1-1.html ），下载作者的文件替换即可。

6. 至此，镜像写入完成。

# Clover的安装和U盘引导配置

1. 在Mac系统中（这里我就直接用我的老华硕本的黑苹果），直接安装最近相对比较新的4452版本的Clover的pkg包，从远景论坛帖子中下载（http://bbs.pcbeta.com/viewthread-1740837-1-3.html ），这个帖子的作者还在持续更新的。

2. （重点）安装pkg的时候，首先要注意，选择安装位置。我是使用U盘安装，选择我的U盘。

3. （重点）选择安装位置后，如果你需要自动更新U盘EFI分区的Clover程序，要记得点击左下角自定义，进去看看，确保“安装Clover到EFI系统区”还有“Drivers64UEFI”都勾上了。“Drivers64UEFI”里面的一些引导驱动，并不都是有用的，但是刚开始安装，不知道缺什么，所以就丢进去好多个。后面我会写如何优化。

4. 然后一路下一步，Clover就安装到了U盘EFI分区中，完成。

# Clover引导进入安装界面探索

1. 进入clover，发现找不到U盘中的MacOS安装分区，识别不了，我的U盘安装分区是HFS+格式的，要在Clover的Drivers64UEFI加入：hfsplus.efi。另外，后续进入系统的话，因为我准备用APFS格式安装系统，可能还会用到apfs.efi，一并加上（这一步有的人可能不需要，主要是看你的Drivers64UEFI全不全）；（这一步现在升级了Clover后，有了一个ApfsDriverLoader-64.efi这个东西，这样就不需要apfs.efi了，也不用费心思去）

2. Clover核心配置文件：config.plist。因为小米游戏本与MacBookPro14,3是类似的，集显是HD630，所以应该比较好搞。配置文件直接去网上找的HD630移动版的模板，在此基础上进行修改应该会少趟点坑（其实坑还是不少）；

3. 进入安装界面前出现花屏+禁止图标：a. USB驱动的问题，将Rehabman大大的USBInjectAll.kext~~和GenericUSBXHCI.kext~~放到kexts对应目录下就过了。（https://bitbucket.org/RehabMan/os-x-generic-usb3/downloads/）（https://bitbucket.org/RehabMan/os-x-usb-inject-all/downloads/）。b. 我的老的CLOVER配置，虽然在进入系统后USB3.0驱动正常，但是在安装过程中，无法识别USB3.0，依然花屏禁止符号，用USB2.0就没事了。后来的CLOVER配置已经解决了此问题。

4. 尝试进行了上述步骤后，最终会进入一个漫长的-v日志输出，之后飞快的日志之后就自动重启，无限循环(最后提示Attempting system restart...MACH Reboot然后重启)。再次让我抓狂。

5. 因为想要根据最后的报错日志寻找解决办法，日志刷的太快看不清，所以尝试寻找-v日志的保存方法，无果。

6. 后来网上查了一下，原来这种现象其实是系统内核崩溃了，具体原因必须查日志看崩溃位置。所以回想起来，这个地方当时让我抓狂很久，但是实际上大家遇到这种问题不要方，想办法搞到崩溃前的日志，一般是有一串dependency开头的堆栈信息，这个咱们码农们应该能看着眼熟的。

7. 于是“机智”的我（其实是笨办法，实在没辙了）通过手机的高速相机功能录制视频，然后通过视频回放找到了崩溃前的信息，大致为（手抄了一下，以供参考）：com.apple.xbs/Sources/GPUDriversIntel/GPUDriversIntel-10.32.48/Common/IONDRV/Intel/KBL/AppleIntelFramebuffer/AppleIntelController.cpp:27617 后面重影看不清，大约有IntelKBLGraphicsFramebuffer字样，dependency里有com.apple.iokit.IOGraphicsFamily等等。大致推测是跟Intel集成显卡有关啦。

8. 于是又去网上查资料，最终确认目标：找到最新的Lilu.kext和IntelGraphicsFixup.kext，新增了一个IntelGraphicsDVMTFixup.kext(http://bbs.pcbeta.com/viewthread-1774121-1-1.html)(https://github.com/BarbaraPalvin/IntelGraphicsDVMTFixup/releases)，config.plist中，还需要简单配置一下，具体可以参见前面这个帖子。

9. 再重启引导，终于进入了安装界面，跨出第一大步。

10. 备注：第7步中，确实存在简单的办法，可以禁用Lilu输出的日志，并且clover中设置内核崩溃时不重启，就可以轻松看到崩溃堆栈信息了。

# Mac安装中问题探索

1. 进入界面后，发现磁盘管理工具中，只识别了我的固态硬盘，机械硬盘没有识别出来，估计是固态硬盘跟机械硬盘在主板上的接口类型不同，缺少驱动（机械是SATA的，固态是NVME的）。解决方案：添加SATA-100-series-unsupported.kext驱动（参考：http://bbs.pcbeta.com/viewthread-1692485-1-1.html）。

2. 安装前我已经在Windows10中分出了一个80GB的分区，未格式化的。然后在Mac安装界面打开磁盘工具，分区，把这个分区格式化为APFS（区分大小写），加密那个我没选，因为我用不到，怕有问题，没去尝试。

3. 直接安装，一切顺利。

# DSDT部分

## 20180521更新

经过 @黑果小兵 大大的指点，改用Hotpatch方式进行DSDT相关的配置，降低耦合性，提高补丁复用性。前期直接就可以应用R大的一些hotpatch补丁了。

# HotPatch

1. HotPatch修复大概意思是，我们不直接基于各种途径提取出来的DSDT和SSDT的文件直接修改，而是有针对性的制作一些补丁文件，通过Clover补进去，这样只需要总结出需要修补的部分，从而降低耦合行，提高补丁复用性。

2. 普通的通用型补丁，可以去R大的Github找到一些。地址：https://github.com/RehabMan/OS-X-Clover-Laptop-Config

# 进入Mac系统后的优化

## 语言

语言不知哪里没搞好，默认英文的，进去改中文，重启即可生效；

## 声卡（两种驱动方式）

1. 基于原生驱动，已经有了合适的，后续要仔细研究研究：https://github.com/vit9696/AppleALC
2. 省事点，用Voodoo通用驱动。但是目前我用的通用的，音乐貌似在某些音高上会无声音，表现就是断断续续。

## 调亮度

参考R大的文章（https://www.tonymacx86.com/threads/guide-laptop-backlight-control-using-applebacklightinjector-kext.218222/），边学习边鼓捣，终于于20180512完工。与此相关的帖子远景坛子里应该也有介绍，以后有需要的话我可以考虑总结一下我自己的调亮度的经历总结。

## 无线网卡

自带Intel8265貌似无解，而且没法换卡，不是插上去的那种。目前只能外接一个USB无线网卡了。

## CPU调频问题

1. CPU-S.app，这个应用挺方便的，使用这个检测出我的CPU频率只有三档，都在3000MHz以上，合着全都是睿频状态啊，这可有点不妙了。我直接偷了个懒，用这个app的生成SSDT文件的功能，生成了一个ssdt.aml，我改名为SSDT-CPUS.aml，放入clover，加入config.plist，然后就出现了10多档了，目前先这样吧，暂时标记已解决。

2. 后来遇到一个问题，增加了其他的SSDT文件后，变频失效了。后来得知，这个SSDT-CPUS.aml在SortedOrder中要排在SSDT-PNLF.aml之后，否则变频会失效。于是把SSDT-CPUS.aml放到了最后，变频重新生效。

## 硬盘温度总是很高问题（疑似）

其实我也不确定到底是高还是不高，使用HWMonitor.app检测温度发现硬盘温度总是40℃，红色字体，摸一下电脑背面还有点热。在Windows中鲁大师测试也是40℃左右，但是感觉摸起来没那么热呢，是心理作用？还是Mac的检测温度软件有bug呢。

## 触摸板和键盘（进行中）

1. ~~参考R大的Github：https://github.com/RehabMan/OS-X-Voodoo-PS2-Controller/wiki/How-to-Install~~，由于游戏本触摸板是接入主板的I2C总线的，是一个I2C类型的设备，因此PS2的驱动是无法生效了。

2. 我们采用VoodooI2C驱动：https://voodooi2c.github.io/，根据驱动作者的文档进行配置，以及对ACPI的一些简单学习后，修改了一些DSDT代码，加入VoodooI2C和VoodooI2CHID驱动，已经可以正常使用，各种原生手势支持的很到位，很给力。具体如何去做，比较啰嗦和复杂，等10.14正式版出了以后，我会考虑发个文章介绍一下我的详细做法。

3. 遗留问题：目前采用直接提取了DSDT进行修改，制作SSDT-hotpatch的方式没有成功，还有待排查原因（只是实现方式而已，用起来没差别）。

4. 键盘Fn键，对应的若干组合键已经可以正常工作，这个地方还是需要R大的PS2驱动的：[VoodooPS2Controller](https://github.com/RehabMan/OS-X-Voodoo-PS2-Controller)。放好驱动之后，还需要DSDT的修改，修改映射。我又进一步制作了对应的SSDT-Hotpatch补丁，这样就舒服了，没毛病了。

## USB3.0的U盘识别

1. 简单解决：

解除USB端口上限限制（这理论上不是最完美的方案，但是更无脑简单）。这里写的MatchOS是10.13.4，已经验证10.13.6是可用的。如果你比较激进的话，可以改为10.13.x，甚至加上10.14.x，那我就没有测试过了。

```
<dict>
        <key>Comment</key>
        <string>disable port limit in XHCI kext (credit PMHeart)</string>
        <key>Disabled</key>
        <false/>
        <key>Find</key>
        <data>
        g32UDw+DlwQAAA==
        </data>
        <key>InfoPlistPatch</key>
        <false/>
        <key>MatchOS</key>
        <string>10.13.4,10.13.5,10.13.6</string>
        <key>Name</key>
        <string>com.apple.driver.usb.AppleUSBXHCI</string>
        <key>Replace</key>
        <data>
        g32UD5CQkJCQkA==
        </data>
</dict>
```

2. 完美解决：

解除端口限制之后，再自定义SSDT补丁来屏蔽无用USB端口，并指定USB端口类型。具体操作可以参考R大的帖子。如果后期有空，或者需求较多，我会考虑写一个专门的文章做介绍（记录一下我的USB驱动方式）。

## Nvidia独立显卡驱动

### config.plist配置

1. Acpi/SortedOrder中，移除SSDT-DDGPU.aml，或者直接在DisabledAML列表中添加“SSDT-DDGPU.aml”;
2. Boot中，勾选nvda_drv=1（据说这个开关已经失效了，无用），取消勾选nv_disable=1（据说这个开关也失效了，无用）；
3. Graphics中，取消勾选Inject NVidia（我这边一开始勾选了，然后发现安装了WebDriver后，系统设置里看不出来独显）；
4. System Parameters中，勾选NvdiaWeb开关，为后面安装WebDriver做准备；
5. 注意：配置了这些之后，这个配置就不能用于安装黑苹果了，否则可能就进入不了安装界面哦。目前Github的master分支已经应用了此配置，所以，安装黑苹果的朋友不要用我的master分支的配置。安装之后在替换才行。

### 更新相关kext

根据http://bbs.pcbeta.com/viewthread-1781764-1-1.html 这个帖子的介绍，为了防患于未然，我更新了下面的三个kext文件，放于CLover中。

- NvidiaGraphicsFixup.kext
- Shiki.kext
- Lilu.kext

### 安装NVidia的WebDriver

1. 下载地址（此地址为tonymacx86论坛根据NVidia官方数据转换出来的页面，会理论上会自动保持最新）：https://www.tonymacx86.com/nvidia-drivers/
2. 下载地址中我们看到了有很多版本的WebDrivers，为了以防万一有不匹配的，我选择直接匹配的版本。可以看到其中很多同样是10.13.4，会有很多个版本，这是因为苹果有时会针对一个版本发布安全修复版本。那怎么查看自己的版本呢？打开终端，执行如下指令，即可看到：
```
cat /System/Library/CoreServices/SystemVersion.plist
```
3. 下载到对应的版本之后（我的是17E199），双击pkg文件进行安装操作。
4. 重启系统。

### 效果

1. HDMI口可以用，接入的外置显示器显示使用的是独显；
2. 打开系统信息可以看到集显和独显共存显示；
3. 但是默认情况下，内置显示器依然使用集显，暂时无法做到内置显示器使用独显；
4. 群里的@Tom菜菜 童鞋提供了一个可能的方法：淘宝搞一个HDMI的显卡欺骗器，插上之后，镜像外接的显示器，可以让独显正常工作在内置显示器里，如果有真的想要用的独显的，可以试试。后来买了一个验证，可行，但是感觉颜色偏白了一些，看起来不爽，暂时没需求，不想用了。

## 声卡驱动

### 简单方法

VoodooHDA.kext

### 效果好一些但是麻烦一些的办法

使用AppleALC。感谢群内的热心小伙伴 @頭糖吥給阣 已经提供了适配好小米游戏本的AppleALC.kext，以及配置方法。可以去我的帖子中爬楼找到。也已经更新到了Github仓库。

## 有线网卡

RealtekRTL8111.kext

## 睡眠立即自动唤醒问题

1. 跟踪控制台系统日志后发现一句日志：

```
Wake from Normal Sleep [CDNVA] due to GLAN XDCI XHC/HID Activity:
```

2. 结合之前查阅的资料，可知确实与USB无脑解除端口限制有关。按照上文中USB修复部分的介绍，完美驱动后即可解决本问题。

## 摄像头

后来测试了一下，貌似是没毛病。

## 蓝牙

禁用内置USB蓝牙后，即可使用外置USB蓝牙。如何禁用？可以通过修改UIAC那个SSDT实现，具体的，后面我也会再写的。

## 与Windows时间不同步问题

### 原因

1. 最近发现经常有人问我，为什么Mac下设置了时间，回到Windows就错了呢？看来，好多人还不了解原因。

2. Mac时间与Windows不同步的问题，主要是因为Windows计算时间的机制跟Mac的标准不同。Mac系统中会把主板设置中的系统时间，认为是UTC时间，也就是标准格林尼治时间（GMT），然后根据我们的北京时间的东八区（GMT+8），再进行增加时区转换（北京时间就+8小时）。而Windows会把主板设置的系统时间认为是进行过时区转换后的时间，所以拿过来就显示。因此造成了不同步。

3. 举个例子。假设我们使用北京时间，大家都知道，北京时间是根据格林尼治标准时间+8小时的，具体地球自转的地理问题我就不讲了哈。如果你在Mac中设置时间为早上8点，那么Mac系统只会按照标准时间记录，因此记录在主板程序内的时间就是凌晨0点。而回到Windows时，Windows会认为这个时间是加了8小时以后的时间，直接不换算就拿来显示出来了。

4. 既然如此，就是统一他们的换算标准呗。我是采用修改Windows注册表的方式，让Windows也按照Mac系统的方式换算时间，这样就OK了。修改方式如下：

### 解决方法一

1. 打开Windows系统，Win+R快捷键打开运行对话框，输入regedit，回车，打开注册表编辑器。

2. 左边能看到树形目录结构，按照下面的路径寻找，并点击：

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\
```

3. 点击上面的节点目录后，在右侧窗口新增一项DWORD，命名为RealTimeIsUniversal，并把值设为1即可。

### 解决方法二

上面操作步骤有点多，小白可能受不了，操作错了就会无效。下面是群友@未来 提供的通过命令行执行上述操作的方法：

Win+R打开运行，输入cmd，回车，打开命令提示行。然后输入：
```
Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```
执行它，可能会弹出管理员权限的请求对话框，允许即可。之后再设置时间时，Mac和Windows计算标准就相同了。

# 如何更新Clover以及配置

待续

# 现状（实时更新）

## 好的方面

1. 集成显卡基本驱动成功，显存显示2048MB
2. 屏幕显示效果很棒，色彩很舒服
3. 电量正常显示，充电状态正常显示
4. 外置USB鼠标、内置键盘可用
5. 集显独显双驱动，但是正常情况下，内置显示器无法真正使用独显
6. HDMI画面外接显示器可用，声音可能不可用，忘记测试；
7. CPU调频8个档，基本算是正常；
8. Airplay可用；
9. 系统设置中亮度调节正常；
10. USB3.0的U盘使用基本正常；
11. 有线网卡正常使用；
12. 声卡使用AppleALC驱动基本正常；（感谢群内的热心小伙伴 @頭糖吥給阣 ）
13. 关上盖子可以睡眠（不修改USB问题也可以）
14. 睡眠后立即唤醒的问题已经解决（USB使用自定义的SSDT修复，屏蔽无用端口）
15. 摄像头正常
16. 触摸板可用：除了双指缩放无法使用，其他Mac系统原生手势均可用
17. 内置蓝牙不可用，已经禁用内置蓝牙，以便可以使用USB外接蓝牙
18. Clover引导时，可以记忆上一次选择的系统并倒计时进入了
19. Fn快捷键组合功能，已经可以调节：声音、亮度、触摸板开关、键盘背光

## 坏的方面（后期慢慢优化）

- 内置无线网卡不工作（Intel板载网卡，无解。目前使用USB无线网卡）
- 偶现一次-v启动系统时卡在waiting for DSMOS无法进入系统，重启就好了，再也没有复现过，是一个隐患，待解决
- 内置蓝牙可以显示，但是搜不到设备，Airdrop不可用，故暂时禁用
- 开机最后阶段的花屏，仅偶尔会出现
