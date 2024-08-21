---
title: WOW插件开发初步探索总结
categories: 
  - 瞎折腾系列
excerpt: 
date: 2020-06-27 23:45:26
tags: 
---

## 参考资料

[老插件教材《Beginning Lua with World of Warcraft Add-Ons》一点粗浅的翻译](https://nga.178.com/read.php?tid=21788060)

[World of Warcraft API（好东西）](https://wowwiki.fandom.com/wiki/World_of_Warcraft_API)

[Lua语法教程](https://www.runoob.com/lua/lua-tutorial.html)

## 需要什么基础？

1. 有一定的英文基础
2. 有一定的编程基础
3. 了解魔兽世界

## 开发工具

1. 任意文本编辑器：随便找个文本编辑器也是可以的。
2. VSCode：毕竟这是我开发中用过的最舒服的文本编辑器，而且功能强大，可以安装很多插件。这时我发现，VSCode的插件库里，竟然真的有大佬写了个WOW插件开发的辅助插件，wow-bundle for VS Code。可以进行很多语法提示，WOW API的提示报错等，简介中号称是支持了8.0.1的客户端API。
3. AddOn Studio for World of Warcraft（力荐）：基于微软的VisualStudio的一个扩展功能，甚至还可以预览界面，真是方便啊。

![Studio启动图.webp](Studio启动图.webp)

![Studio内部界面.webp](Studio内部界面.webp)

## 魔兽插件插件目录

1. 怀旧服插件目录：World of Warcraft/_classic_/Interface/AddOns
2. 正式服插件目录：World of Warcraft/_retail_/Interface/AddOns

## 插件构成

- xxx.toc文件：用于标识插件名称等信息，以及插件包含哪些lua文件和xml文件，都要在此声明。
- xxx.lua文件：插件代码在这种类型的文件中，用于编写逻辑，以及动态创建界面等。
- xxx.xml文件：用于编写静态界面布局。适用于快速简便的创建静态插件界面。

## 插件开发小实战

下面以我练手开发的一个小插件为例，捋一下插件开发的要点。

代码Github共享：

https://github.com/sandy1108/QuickMessage

### TOC文件内容示例

```

## Interface: 11304
## Title: QuickMessage
## Notes: Some Message Listener
## Author: 洛阿神灵木事-炉石旅馆-加丁
## Version: 1.0.0
## eMail: sandy1108@163.com
## Notes-zhCN: 快捷发送组团信息、战场频道监听（不生效）、聊天提醒（暂无）等功能
## Title-zhCN: 快捷消息工具


QuickMessage.xml
QuickMessage.lua
AutoLeaveBattlefield.lua

```

1. 通过上面的文件内容，可以看出大致的格式。

2. 其中#开头的都是插件规定好的信息声明方式，可以修改冒号之后的内容，其中Title会作为游戏内的插件名称来显示（如果Title-zhCN那就是我们中文版游戏客户端会优先显示的名字了）。

3. 下面三个文件用来声明本插件涉及到的插件代码文件。

### xml编写

简单的说就是，我使用了前文提到的方便好用的IDE，可以拖拽UI，来构建，然后再进行微调，增加点击事件的绑定等。

时间关系，这里暂时不多赘述，以后再补充吧。

### lua编写tips

1. API查询：建议去文章开头的API文档链接，我认为还是很方便的查询各类API的。其中包括lua中使用的API，以及xml中的布局属性等。有点英文基础的慢慢看一下其中的API，相信你会有跟我一样的感叹：哎哟，插件还可以干这个！或者是：哦，原来xx插件是这样实现的！

2. 不过，只知道了插件API还不够。我们还要知道插件加载机制，以及插件界面的绘制方式，要不然也没法与用户交互（除非是一个自动监听什么事件然后自动干活的插件）

3. 一般为了规范，与xml文件相关的代码都会写在同名的lua文件中。



### 测试API的小技巧

在聊天窗口中，可以直接输入/script空格后面跟lua代码，就可以直接在游戏中执行代码了。

### 功能举例：发送聊天消息

> https://wowwiki.fandom.com/wiki/API_SendChatMessage

发送密语（其中，WHISPER代表密语类型）：

```
SendChatMessage("Hello Bob!", "WHISPER", "Common", "Bob");
```

给对应的频道号发送消息：

```
SendChatMessage("test","CHANNEL",nil,"1")
```

咦，频道号是什么鬼？我看到的都是大脚世界频道，谁知道是啥号啊？那么，我们可以根据频道名查找频道号，然后给这个频道号发消息：

```
local msg = "Hello"
local index = GetChannelName("大脚世界频道") -- It finds General is a channel at index 1
if (index~=nil) then 
  SendChatMessage(s , "CHANNEL", nil, index); 
end
```

然而在实际使用过程中我发现，这个GetChannelName的API获取频道号总是-1，但是单独运行这个代码是可以生效的，不知道是不是暴雪故意做了限制，还是我的用法不对？

于是我想了另一个办法，那就是获取所有已经加入了的频道列表数据：

```
function GetJoinedChannels()
    local channels = { }
    local chanList = { GetChannelList() }
    for i=1, #chanList, 3 do
        table.insert(channels, {
            id = chanList[i],
            name = chanList[i+1],
            isDisabled = chanList[i+2], -- Not sure what a state of "blocked" would be
        })
    end
    return channels
end
```

然后我们可以遍历来判断想要的频道，或者如我的插件中的逻辑，将所有的频道全部列举出来，每个频道生成一个对应的按钮：

```
function QMButtonInit:OnClick()
	print("QMButtonInit:OnClick")
	local aboveFrame = QMEditBoxInputMessage
	local channelYell = {
		id = -1,
		name = "喊话",
		isDisabled = false
    }
	aboveFrame = InitSendButton(1, aboveFrame, channelYell)
	local channelSay = {
		id = -2,
		name = "说话",
		isDisabled = false
	}
	aboveFrame = InitSendButton(2, aboveFrame, channelSay)
	local staticChannelCount = 2 -- 这里是指上面的喊话和说话这两种消息方式是固定的，所以后面的InitSendButton里携带的序号就是基础上再递增
	local channelObjList = QuickMessageFrame:GetJoinedChannels()
	for i=1, #channelObjList do
		local _channelObj = channelObjList[i]
		aboveFrame = InitSendButton(staticChannelCount + i, aboveFrame, _channelObj)
	end
end
```

这样就可以使用对应的频道ID来发消息了。

## 暂告段落

由于我们公会是G团，开团时拍装备也是必备的，为了满足大家的需要，我优化了RaidLedger插件的提示语，并增加了开关用于控制通知频率等功能，方便开团拍装备和记账等。另外我还修改了BuffWatcher插件的提示语，方便开团分配Buff。当然还是要感谢原作者了，相比原作者的开发，我只是修改了皮毛而已。通过这些修改，也更多的学习了插件机制，以后有机会还可以进一步学习和开发新功能。

后续，还想再添加一下配置界面，并加入启动插件的快捷入口，或者指令。然后再添加一下聊天消息关键字的监听啊，声音提醒啊什么的。

## 关于我

我是怀旧服，一区加丁，炉石旅馆公会，洛阿神灵木事，有兴趣的也可以加入我们公会游戏内进行交流。 [公会详情直接进NGA的这个帖子](https://nga.178.com/read.php?tid=21839151&_ff=666)

