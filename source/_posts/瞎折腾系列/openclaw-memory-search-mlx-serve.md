---
title: "折腾 OpenClaw Memory Search：从老 Mac 自编译 llama.cpp 到新 Mac 的 MLX Serve"
categories:
  - 瞎折腾系列
excerpt: "为了让 OpenClaw 能在本地搜索 Memory、Session 和历史资料，我先在老 Intel Mac mini 上运行 EmbeddingGemma，随后又经历了配置字段迁移、llama.cpp 与 Monterey 不兼容、自编译运行时和反复重建索引。最后，我把 Embedding 计算迁到新 Mac mini，通过 MLX Serve 与 New API 为老 Mac 提供服务。"
date: 2026-09-10 00:00:00
tags:
  - OpenClaw
  - MemorySearch
  - Embedding
  - llama.cpp
  - MLXServe
  - NewAPI
  - Macmini
---


> 本文主要记录了自己折腾OpenClaw的MemorySearch的经历。

## 一、为什么我要给 OpenClaw 配置 Memory Search

随着用OpenClaw用的越来越多，Markdown 记忆、Session 和整理后的历史工作资料越来越多。有时候很希望Agent可以随时回忆起之前合伙干的事情，但是也不希望大量内容全都塞进上下文中。而且，也希望 Agent 不只靠关键词或逐个文件翻找，而是能进行语义搜索。

那有大佬肯定会说，你这折腾这玩意干啥，现在有很多开源记忆工具啊，大厂搞得，比如飞书的OpenViking，腾讯的MemoryDB之类的，直接接入不就完事了么？

首先呢，我的OpenClaw主打的是学习功能，其次是管理我电脑的一些工作，查询我的知识库笔记等等，充当一个日常小助手。虽然确实积累的会话或者笔记也有几千篇了，但是还不至于接入这种开源大型框架，因为接入之后意味着他们很可能会打乱我的存储方式，建立它自己的索引结构，而且一旦他们哪天停止维护，或者维护更新缓慢了，更新OpenClaw时很可能导致无法启动，受制于第三方的感觉，懂得都懂。所以不到万不得已，我不想搞太多第三方插件，更希望直接使用OpenClaw项目组提供的官方功能。

## 二、老macmini本地运行 EmbeddingGemma

我先在老 Mac mini 上尝试直接使用 OpenClaw 自带的本地 Embedding 方案。先安装官方的 llama.cpp Provider：

```bash
openclaw plugins install @openclaw/llama-cpp-provider
```

然后把默认的 Memory Search Provider 切换到本地：

```bash
openclaw config set agents.defaults.memorySearch.provider local
```


默认模型应该是在provider设置为local之后自动进行初始化下载的，最后保存在 `~/.openclaw/models/llama.cpp/` 目录下，实际文件名是：

```text
hf_ggml-org_embeddinggemma-300m-qat-Q8_0.gguf
```

这个模型生成的是 768 维向量。为了让 `main` Agent 搜索到更多内容，我还打开了 Session Memory，并把搜索来源扩展为默认 Memory、Session 以及额外整理的历史资料目录。Session 相关配置放在 `main` Agent 自己的 `memorySearch` 配置里，额外目录则通过 `extraPaths` 指定，OpenClaw 会递归扫描这些目录中的 Markdown 文件。

配置结构大致如下，路径部分用占位符代替：

```json5
{
  "agents": {
    "defaults": {
      "memorySearch": {
        "provider": "local"
      }
    },
    "list": [
      {
        "id": "main",
        "memorySearch": {
          "experimental": {
            "sessionMemory": true
          },
          "sources": [
            "memory",
            "sessions"
          ],
          "extraPaths": [
            "/path/to/extra-memory"
          ]
        }
      }
    ]
  }
}
```

配置完成后，主要用到了下面三个命令：

```bash
# 用来做完整状态检查，`--deep` 会连 Provider、向量库和语义搜索一起检查
openclaw memory status --deep --agent main

# 主动触发索引的命令，在索引处于 dirty 状态时会真正启动索引
openclaw memory status --index --agent main

# 实际测试搜索效果
openclaw memory search "配置deepseek的key" --agent main
```

老 Mac mini 可以完成这套工作，只是第一次给 360 个文件建立索引时花了大约 2 个小时，没办法，毕竟是2014的老macmini了。之后新增的内容走增量索引，不需要每次都重新处理全部文件。

## 三、OpenClaw版本升级2026.8.1：Memory Search 的配置字段变了

升级到 OpenClaw 2026.8.1 后，我发现 Memory Search 又换了一套配置入口。之前写在 `agents.defaults.memorySearch` 里的公共配置，迁移到了顶层的 `memory.search`；单个 Agent 的配置也不再直接挂在旧的列表项上，而是放到 `agents.entries.<agentId>.memory.search` 下面。

我原来配置的本地 Provider、Session Memory、搜索来源和额外目录都需要保留，只是配置位置变了。迁移后的结构大致是这样，目录仍然用占位符表示：

```json5
{
  "memory": {
    "search": {
      "provider": "local",
      "extraPaths": [
        "/path/to/extra-memory"
      ]
    }
  },
  "agents": {
    "entries": {
      "main": {
        "memory": {
          "search": {
            "experimental": {
              "sessionMemory": true
            },
            "sources": [
              "memory",
              "sessions"
            ]
          }
        }
      }
    }
  }
}
```

这样配置之后，重建索引，之后进行memory search的时候，就可以顺利让agent找到之前很久远的历史对话了，效果还算不错。这时候，我以为万事大吉，坐享其成了。但是我高兴的太早了。

## 四、升级到 2026.9.1 后，官方 llama-server 在 Monterey 上启动失败

万万没想到，这次升级竟然是一次”史诗级升级“，官方称之为OpenClaw2.0。惊喜之余，也带来了各种”毁灭性“的兼容问题。我的MemorySearch又又不能用了。这次的原因是：

1. 新版 OpenClaw 的本地 Embedding 改为 Managed llama.cpp，由 OpenClaw 管理启动、健康检查和空闲停止，这意味着版本必须高度一致，否则可能会出现API兼容问题。
2. OpenClaw会自动下载llama.cpp的 `b10534 darwin-x64` 预编译包，但是这个包无法兼容在我的 MacOS 12 Monterey，悲剧啊！不过这也不怪人家，主要还是我这Macmini是在太老了，还硬要用。具体报错如下：

```text
dyld: Symbol not found: (_cblas_sgemm$NEWLAPACK$ILP64)
```

- GGUF 模型和 Memory 数据没有损坏。
- 根因是官方 binary 链接了老系统 Accelerate Framework 不提供的符号。

## 五、为了继续使用，硬着头皮在老 Mac 上重新编译 llama.cpp

- Apple Command Line Tools 和 Clang 可以继续使用。
- 真正先卡住的是旧 Homebrew 遗留的 CMake 3.7.2，而目标源码要求 CMake 3.14 以上。
- 使用 MacPorts 安装新版 CMake，因为HomeBrew也已经不支持MacOS 12了。（各种被抛弃啊！）
- 不编译 llama.cpp 最新版，精确对齐 OpenClaw 指定的 `b10534`这个commit。
- 第一版以稳定为先，关闭 Accelerate、BLAS、Metal 和不必要的动态依赖。
- 只编译 `llama-server`，并使用 `-j2` 控制老 Mac 的压力。

计划保留的核心命令：

```bash
cmake -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DGGML_ACCELERATE=OFF \
  -DGGML_BLAS=OFF \
  -DGGML_METAL=OFF \
  -DLLAMA_OPENSSL=OFF \
  -DBUILD_SHARED_LIBS=OFF \
  -DLLAMA_BUILD_NUMBER=10534 \
  -DLLAMA_BUILD_COMMIT=2b5621094

cmake --build build --target llama-server -j2
```

## 六、编译成功还不够，OpenClaw 对 binary 版本检查得很严格

- 默认自编译产物显示 `build 1`，无法通过 OpenClaw 校验。
- build number 必须显式对齐 `10534`。
- commit 写成完整 40 位 SHA 看起来更准确，却仍然无法匹配当时的校验规则。
- 最终必须使用九位 commit：`2b5621094`。
- 只把 `localService.command` 指向自编译文件仍可能被 Managed 安装检查拦截。
- 最终在保留官方坏 binary 备份的前提下，用软链接让 Managed 路径指向自编译兼容版本。

验证目标：

```text
version: 0.1.2-dev (build 10534, commit 2b5621094)
```

这时候我已经感觉有点烦了，我这是图啥呢？

## 七、这套方案跑通了，但升级维护越来越累

这套方案跑通以后并没有马上坏掉，后来升级到 OpenClaw 2026.9.2 也还能继续工作。真正让我觉得累的，是每次升级都要重新确认一遍：llama.cpp 的版本有没有变化，索引的 provenance classifier 和 chunking version 有没有变化，旧索引还能不能继续使用。

老 Mac mini 曾经完成过一万多个 chunks 的索引，但每次大规模重建都要消耗很长时间。如果 OpenClaw 后续把 Managed llama.cpp 从 `b10534` 换成新的 release，我还得重新编译、重新对齐版本，再处理一遍 Managed 路径。虽然本地 Embedding 不需要支付云 API 费用，但 CPU 时间和维护成本一点都没有消失，人的时间成本也是成本啊～

折腾到这里，我已经不太想长期维护这套专门适配 Monterey 的自编译运行时了。它适合在官方 binary 无法启动时救急，长期跟着 OpenClaw 升级就有点累。

## 八、把 Embedding 计算迁到新的 Mac mini 2024

既然老 Mac mini 主要的问题是算力和兼容性，我就开始考虑把 Embedding 计算迁到新的 Mac mini 2024。新 Mac 使用 Apple Silicon，运行 MLX 模型更节约系统资源；老 Mac 则继续负责做一些简单的工作吧，比如OpenClaw、Memory 文件和 SQLite 索引。

于是，保存记忆和计算向量就拆成了两件事。老 Mac 不用再自己编译和维护 llama.cpp，只需要通过 OpenAI-compatible 接口请求新 Mac 的 Embedding 服务。

目标链路：

```text
老 Mac mini
OpenClaw + Memory 文件 + SQLite 索引
              |
              | OpenAI-compatible Embeddings
              v
New API 中转站
              |
              v
新 Mac mini 2024
MLX Serve + Qwen3 Embedding
```

## 九、先尝试 LM Studio，结果模型类型识别出了问题

一开始我还是想先用 LM Studio，毕竟它有图形界面，在 Apple Silicon 上启动模型也比较方便。结果下载了千问以及其他 Embedding 模型之后，LM Studio 在我这次的实际环境里把它们识别成了普通 LLM，而不是 Embedding 模型。

这样就没办法按预期提供 `/v1/embeddings` 接口了。虽然也不好说LM Studio 以后永远就无法支持 Embedding，但是一直等着适配也满足不了当下需求啊。所以我换成了专门提供 MLX Embedding 服务的工具。

## 十、改用 MLX Serve

改用 MLX Serve，通过 `pipx` 安装。拉取模型时显式加上 `--type embedding`，从命令入口就把它和普通 LLM 区分开来。我使用的是 `Qwen3-Embedding-0.6B-4bit`，在新 Mac 上启动后，对外提供 OpenAI-compatible 接口。

服务启动后，我分别检查了 `/v1/models` 和 `/v1/embeddings`，确认模型能被识别，也确实能返回向量。这个 Qwen3 模型返回的是 1024 维向量。

记录一下关键命令：

```bash
pipx install git+https://github.com/menaje/mlx-serve.git

mlx-serve pull Qwen/Qwen3-Embedding-0.6B \
  --type embedding \
  --quantize 4

mlx-serve start \
  --host 127.0.0.1 \
  --port 2234 \
  --foreground
```

## 十一、通过 自己的 New API 服务把 Embedding 服务提供给 OpenClaw

MLX Serve 已经能工作之后，我没有让 OpenClaw 直接写死新 Mac 的局域网地址，而是继续复用已经在使用的 New API。这样 New API 负责统一入口、认证和模型路由，OpenClaw 只需要配置一个 `openai-compatible` Memory Provider。

大概验证顺序就是这样的：

```text
MLX Serve 本机接口
→ 老 Mac 直连 MLX Serve
→ New API 转发
→ OpenClaw embeddingProbe
→ OpenClaw 实际 memory search
```

这条链路中间确实遇到过认证问题，但后来已经处理完了。现在 OpenClaw 通过 New API 可以拿到新 Mac 上 MLX Serve 返回的 Embedding，整条链路算是走通了。接下来就是跟之前一样，修改配置为远端embedding服务，然后重建索引。

OpenClaw相关配置的修改片段如下：

```json5
{
  "memory": {
    "search": {
      "provider": "openai-compatible",
      "model": "Qwen3-Embedding-0.6B",
      "remote": {
        "baseUrl": "https://<New API 地址>/v1",
        "apiKey": "${NEW_API_API_KEY}"
      }
    }
  }
}
```

这里的 `baseUrl` 填 New API 对外提供的 `/v1` 地址，`model` 要和 New API 以及 MLX Serve 对外暴露的模型名保持一致。

## 十二、最终链路与实际效果

最后的链路变成了这样：OpenClaw、Memory 文件和 SQLite 索引继续放在老 Mac mini 上，Embedding 计算交给新 Mac mini 2024 上的 MLX Serve，New API 作为中间入口把服务提供给 OpenClaw。

这次折腾最后没有把所有东西都搬到新机器上，主要是把最容易拖慢老 Mac、也最容易受系统版本影响的那部分计算拆了出去，毕竟新的macmini我还想用作开发机写代码用呢，还是少占用内存吧，让老macmini继续发光发热吧！哈哈。

## 十三、折腾完之后的感受

这一波折腾也是够麻烦的了，也多亏那段时间比较上头，白天即使出门了，还要用手机远程控制codex remote继续干活，哈哈。

希望 OpenClaw 升级以后，Memory Search 不要再让我反复折腾这么多轮。估计应该也不会了吧，都已经做到这种地步了，还想咋滴？
