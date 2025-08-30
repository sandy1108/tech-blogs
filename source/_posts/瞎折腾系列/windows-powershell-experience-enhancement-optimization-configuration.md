---
title: Windows Powershell体验增强优化配置
categories:
  - 瞎折腾系列
excerpt: 默认的Windows PowerShell自动补全功能比较基础，只能补全文件路径和基础命令，体验不够友好。通过安装一些增强模块，可以大幅提升PowerShell的自动补全体验，让命令行操作更加高效便捷。
date: 2025-07-05 23:00:00
tags:
  - PowerShell
  - 自动补全
  - oh-my-posh
  - Windows
  - 命令行
---

## 背景

默认的Windows PowerShell自动补全功能比较基础，只能补全文件路径和基础命令，体验不够友好。通过安装一些增强模块，可以大幅提升PowerShell的自动补全体验，让命令行操作更加高效便捷。

## 主要模块

1. [oh-my-posh](https://ohmyposh.dev/)：一个PowerShell主题管理工具，可以自定义PowerShell的显示样式。
2. [PSReadLine](https://github.com/PowerShell/PSReadLine)：一个PowerShell的输入提示和自动补全模块，可以提升PowerShell的输入体验。
3. [Zoxide](https://github.com/ajeetdsouza/zoxide)：一个用于快速跳转到目录的命令行工具，可以提升PowerShell的目录切换体验。
4. [PSFzf](https://github.com/kelleyma49/PSFzf)：一个PowerShell的模糊搜索模块，可以提升PowerShell的模糊搜索体验。

## 安装与配置

### oh-my-posh (2025更新)

oh-my-posh的安装，一开始走了个弯路。网上搜到的一些方法都是使用Install-Module安装，但是oh-my-posh的安装会失败，会提示oh-my-posh的二进制文件不存在，后来发现，官网的最新安装方法已经改变了。下面节选官网的安装方法的核心步骤，具体可以去官网查看（https://ohmyposh.dev/docs/installation/windows）：

1. 右键点开始菜单徽标，菜单中选择管理员终端，打开管理员模式的终端，执行命令：

https://ohmyposh.dev/docs/installation/windows

```
winget install JanDeDobbeleer.OhMyPosh --source winget --scope machine --force
```

2. 配置powershell中启用oh-my-posh：

https://ohmyposh.dev/docs/installation/prompt

```
notepad $PROFILE
```

上面这个命令会打开powershell的配置文件，在文件末尾添加以下内容：

```
oh-my-posh init pwsh | Invoke-Expression
```

如果$PROFILE文件不存在，给它new一个：

```
New-Item -Path $PROFILE -Type File -Force
notepad $PROFILE
```

3. 重启powershell，就可以看到oh-my-posh的样式了。

4. 如果样式有点错乱，可能是因为你的powershell终端的字体不太行，oh-my-posh要求终端使用Nerd Fonts，Nerd Fonts类型的字体是指包含特殊字符的字体，比如Fira Code、Maple Mono NF CN等。自行下载字体，并双击安装到系统中，然后进入Powershell的设置，外观，改字体，重启。

5. 配置主题：

https://ohmyposh.dev/docs/themes

比如我要使用unicorn主题，那么需要下载unicorn.omp.json文件，并保存到
C:\Users\{用户名}\AppData\Local\oh-my-posh\themes\unicorn.omp.json文件。

需要修改$PROFILE文件，将`oh-my-posh init pwsh | Invoke-Expression`改为`oh-my-posh init pwsh --config "$env:LOCALAPPDATA/themes/unicorn.omp.json" | Invoke-Expression`

这里使用了环境变量$env:LOCALAPPDATA，这个环境变量会自动指向C:\Users\{用户名}\AppData\Local目录。而且还要特别注意，字符串的引号要用双引号，否则变量不会被解析。

主题的config文件可以去官网github下载：https://ohmyposh.dev/docs/themes

也可以是直接指定官方主题的名称，会自动下载（但是影响终端启动速度）：`oh-my-posh init pwsh --config "your_theme_name" | Invoke-Expression`）

### PSReadLine

1. 安装PSReadLine：

```
Install-Module PSReadLine -Repository PSGallery -Scope CurrentUser -Force
```

2. 配置PSReadLine，下面配置仅供参考，拷贝到$PROFILE文件中：

```
#-------------------------------  Set Hot-keys BEGIN  -------------------------------
# 设置预测文本来源为历史记录
Set-PSReadLineOption -PredictionSource History

# 每次回溯输入历史，光标定位于输入内容末尾
Set-PSReadLineOption -HistorySearchCursorMovesToEnd

# 设置 Tab 为菜单补全和 Intellisense
Set-PSReadLineKeyHandler -Key "Tab" -Function MenuComplete

# 设置 Ctrl+d 为退出 PowerShell
Set-PSReadlineKeyHandler -Key "Ctrl+d" -Function ViExit

# 设置 Ctrl+z 为撤销
Set-PSReadLineKeyHandler -Key "Ctrl+z" -Function Undo

# 设置向上键为后向搜索历史记录
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward

# 设置向下键为前向搜索历史纪录
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
#-------------------------------  Set Hot-keys END    -------------------------------
```

### Zoxide

1. 安装Zoxide：

```
winget install ajeetdsouza.zoxide
```

2. 配置Zoxide，拷贝到$PROFILE文件中：

```
Invoke-Expression (& { (zoxide init powershell | Out-String) })
```

### PSFzf

1. 安装fzf：

https://github.com/junegunn/fzf#installation

```
	winget install fzf
```

2. （注意前置，需要先安装PSReadLine）安装PSFzf：

https://www.powershellgallery.com/packages/PSFzf/2.6.14

```
Install-Module -Name PSFzf
```

3. 配置PSFzf，拷贝到$PROFILE文件中：

```
Import-Module PSFzf
```

4. 这样，重启powershell，就可以使用PSFzf了。可以使用快捷键Ctrl+R进行模糊搜索，Ctrl+T进行模糊切换目录。

