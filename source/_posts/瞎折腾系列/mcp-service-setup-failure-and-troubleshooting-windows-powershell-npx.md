---
title: MCP服务配置失败的踩坑记录（Windows + PowerShell + VSCode + npx）
categories:
  - 瞎折腾系列
excerpt: 同一台机器，PowerShell 里 npx 一切正常，MCP 配置里却启动失败？最后发现是“命令跑在了 cmd，而我的 npx 依赖 PowerShell Profile/fnm 初始化”。这篇把定位过程、坑点清单、以及稳妥的解决方案整理一下。
date: 2025-08-13 00:30:00
tags:
  - MCP
  - PowerShell
  - Windows
  - VSCode
  - npx
  - 环境变量
---

## 背景

- 系统：Windows
- 平时用 PowerShell，Node 由 fnm 管理不同版本
- PowerShell 启动时通过 Profile 脚本自动注入 PATH/初始化 fnm
- 我在VSCode的AugmentCode中MCP 服务配置里，默认用的是 cmd 去跑 npx

现象：我在 独立的PowerShell 里跑 npx 没问题；MCP 配置里一运行就跪（找不到 npx 或执行异常）。

---

## 问题现象

- 手动 PowerShell 中：`npx -v` 正常
- MCP 配置（`command: "cmd"` + `npx ...`）：服务起不来，报错或找不到 npx

后来一看，命令是在 cmd 下执行，而我所有 Node 环境都绑在 PowerShell 的 Profile 里。cmd 不会加载 PowerShell 的 Profile，这就对不上了。

---

## 快速定位过程

在 PowerShell 中确认 npx 到底指向哪儿：

```
Get-Command npx
```

输出是 `ExternalScript  npx.ps1`，路径位于 `fnm_multishell` 目录，说明我的 npx 是靠 PowerShell 环境（Profile + fnm 初始化）来生效的。

再确认 Profile内容：

```
notepad $PROFILE
```

内容如下：

```
fnm --fnm-dir I:\0.DeveloperTools\NodeJS\fnm  env --use-on-cd --shell power-shell | Out-String | Invoke-Expression
```

说明平时的 PowerShell 会话里 Profile 配置了自动注入当前fnm管理的node版本。之前没有注意过fnm的工作原理，原来是这样，它并没有将 node/npm/npx 加入到 系统全局的PATH，而是通过 PowerShell 的 Profile 来注入。

最后换到 cmd 验证：

- `cmd /c npx -v` 往往不行（说明 cmd 环境里没有 npx）
- `cmd /c powershell -NoLogo -Command ". $PROFILE; npx -v"` 是可以的（还是走Powershell，指定预先加载Profile，npx就通）

---

## 根因分析

- VSCode中，MCP配置运行时会使用cmd，cmd 不会自动加载 PowerShell 的 Profile
- 如果是常规的Node安装，环境变量配置在系统环境变量的PATH中，cmd也可以顺利识别。
- 然而我的 Node/npm/npx 是使用fnm管理，通过 PowerShell Profile 中 使用 fnm 初始化脚本注入到当前会话的 PATH 中，而默认cmd不会执行这个初始化脚本

---

## 解决方案


让 MCP初始化时 用 PowerShell，并在执行前加载 Profile（推荐）
   - 把 command 改为 `powershell`
   - 在 `-Command` 里先 dot-source `. $PROFILE`，再执行 `npx`
   - 好处：沿用现有 PowerShell 的初始化（fnm 等），改动最小

修改示例如下：

```json
{
  "mcpServers": {
    "Context 7": {
      "command": "powershell",
      "args": [
        "-NoLogo",
        "-Command",
        ".",
        "$PROFILE;",
        "npx",
        "-y",
        "@upstash/context7-mcp@latest"
      ]
    }
  }
}
```

如果你的机器 ExecutionPolicy 较严格，可能需要在 `args` 里加上 `-ExecutionPolicy Bypass`（本文不使用）。

---


