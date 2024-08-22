---
title: 安装使用oh-my-zsh（Win10+WSL或Ubuntu）
categories: 
  - 瞎折腾系列
excerpt: Windows Subsystem for Linux（简称WSL）是一个为在Windows 10上能够原生运行Linux二进制可执行文件（ELF格式）的兼容层。
date: 2019-05-18 13:39:20
tags: 
---


## 参考了若干地方

https://www.jianshu.com/p/b147735ff3f2

https://blog.csdn.net/weixin_33955681/article/details/87525155

https://www.cnblogs.com/xishuai/p/mac-iterm2.html

## 什么是WSL

Windows Subsystem for Linux（简称WSL）是一个为在Windows 10上能够原生运行Linux二进制可执行文件（ELF格式）的兼容层。

说白了，意思就是，Windows10里可以用Linux的终端了。

## 启用WSL（Ubuntu）

打开Win10应用商店，搜索Ubuntu，进行安装。安装成功后，在开始菜单中输入Ubuntu找到并启动linux终端。启动后，为了以后使用方便，我把他在任务栏上的图标，右键锁定在任务栏中了。

### 透露一个小技巧

如何在当前目录打开PowerShell或者WSL终端？了解的可以跳过此处。

在资源管理器的目标目录的空白位置，按住Shift键，点击鼠标右键，菜单里就会有两个终端的快捷启动方式，启动后，路径自动前进到当前目录。很方便很实用的。

### WSL安装后的目录位置

```
C:\Users\用户名\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs
```

### WSL初步配色调料包

- 字体：consolas

- 字体大小：24

- 屏幕文字颜色：220 220 220

- 屏幕背景颜色：50 50 50

- 不透明度：95%

## WSL采用Solarized配色调料包

背景rgb（0，43，53），文字rgb(147,161,161)

## WSL的主题配色临时工具：ColorTool

1. 微软官方出品，github开源。看样子应该是Windows未来的新终端里会带的工具。可能未来的新终端里就会集成主题功能了。这个工具貌似是可以使用那些iTerm2上的漂亮主题，挺不错的。

2. github源码：https://github.com/Microsoft/Terminal/tree/master/src/tools/ColorTool

3. github下载：https://github.com/microsoft/terminal/releases/tag/1904.29002

4. 下载后解压缩得到一个ColorTool.exe以及一个schemes文件夹。schemes文件夹内包含很多主题文件。比如我想要solarized_dark主题，然后使用WSL终端在目录里执行（PowerShell也行）：

```
./ColorTool.exe -d schemes/solarized_dark.itermcolors
```

终端中显示：Wrote selected scheme to the defaults. 表示已经成功。重启WSL即可。

## 分割线：从此处往后Ubuntu和Win10均适用

## 主角登场：安装zsh终端

1. 使用Ubuntu包管理工具安装

```
sudo apt-get install zsh
```

2. 重启WSL，或者直接输入命令切换到ZSH

```
zsh
```

3. 网上有人说安装后重启WSL会自动回到bash终端，还需要设置默认终端才可以。不过我没有遇到这种情况。


## 分割线：从此处往后，Win10、Ubuntu、Mac应该均适用

## 安装Oh-My-Zsh

这应该是针对zsh终端的一个扩展框架吧，其中可以支持配置很多插件、主题，比较方便。

### 通过在线脚本安装

```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### 配置文件

配置文件是在用户目录下的.zshrc文件，可以按照本文前面说的目录位置直接去找到它进行修改，也可以vi修改：

```
vi ~/.zshrc
```

### 配置主题agnoster

查找配置Key字符串ZSH_THEME，将等号后面引号里面的改为“agnoster”。即：

```
ZSH_THEME="agnoster"
```

### 配置隐藏终端前缀的用户名和主机名

```
DEFAULT_USER="zhangyipeng"
```

### （WSL）安装字体 FiraCode

https://raw.githubusercontent.com/tonsky/FiraCode/master/distr/ttf/FiraCode-Retina.ttf

下载，打开，点击安装。

安装完成后，进入终端设置（WSL的话，点击窗口左上角，然后属性），修改字体为FiraCode Retina。


## 在custom目录中安装插件和修改主题

~/.oh-my-zsh中有个custom目录，其中可以存放自定义插件和主题。为了不影响原有的安装目录，我决定把新增的插件放到这里面。这里面有themes和plugins目录，看名字应该知道，其实是跟根目录下的对应文件夹功能是一致的。所以应该知道怎么用了吧。

### （WSL）修改agnoster主题

由于WSL中默认的agnoster主题，当前目录的高亮配色有点蛋疼，故进行修改：

执行命令将原主题拷贝过来，并改名加上后缀_wsl：

```
cp ~/.oh-my-zsh/theme/agnoster.zsh-theme ~/.oh-my-zsh/custom/theme/agnoster_wsl.zsh-theme
```

找到下面的部分：
```
# Dir: current working directory
prompt_dir() {
  prompt_segment blue $CURRENT_FG '%~'
}
```
将其中blue修改为075，这样颜色会更容易辨认了。（此处参考文章：https://www.cnblogs.com/Ricky81317/p/9062590.html）

由于我们修改了主题名称，还需要去刚才的.zshrc配置文件修改主题为agnoster_wsl。这样，重启WSL就大功告成了。

### 配置主题后光标看不到了？

> 此处参考（但是原文中的字符写错了，都要改为半角字符才可以）：https://blog.csdn.net/Judy_Angella/article/details/84980427

显示光标闪烁：

```
echo -e "\033[?25h"
```

隐藏光标闪烁：

```
echo -e "\033[?25l"
```

### 安装语法高亮插件

> 注意：将目录切换到~/.oh-my-zsh/custom/plugins中，然后按照下面步骤进行操作即可。

Clone this repository in oh-my-zsh's plugins directory:

```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

Activate the plugin in ~/.zshrc:

```
plugins=( [plugins...] zsh-syntax-highlighting)
```

Restart zsh (such as by opening a new instance of your terminal emulator).

### 安装自动提示插件

> 注意：将目录切换到~/.oh-my-zsh/custom/plugins中，然后按照下面步骤进行操作即可。

Clone this repository into $ZSH_CUSTOM/plugins (by default ~/.oh-my-zsh/custom/plugins)

```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

Add the plugin to the list of plugins for Oh My Zsh to load (inside ~/.zshrc):

```
plugins=(zsh-autosuggestions)
```

Start a new terminal session.

## 总结

经过了这么一顿操作后，WSL终于像样了。面对这样的友好的终端，工作心情会好很多。

![美化后的WSL.webp](美化后的WSL.webp)
