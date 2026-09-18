---
title: 写了一个让 OpenClaw 调用 Agnes 生成图片和视频的 Provider 插件
categories:
  - 瞎折腾系列
excerpt: 听说 Agnes 提供了限量免费的图片和视频模型，白嫖谁不喜欢，于是我想把它接进 OpenClaw。直接配置原生媒体模型不能用，临时做成 Skill 又不够顺手，最后干脆让 Codex 补了一个 Agnes Provider。
date: 2026-09-02 00:00:00
tags:
  - OpenClaw
  - Agnes
  - 开源项目
  - 插件
  - AI生图
  - AI生视频
---

其实在几个月前，我就听说了 Agnes 这个 AI 平台，Flash 系列的文本、图片和视频模型都可以限量免费调用。虽然有额度限制，但能白嫖那不是挺好的，更何况很多平台即使送一些免费额度，也很少连图片和视频生成一起开放。

所以我第一反应就是：拿它来给我的 OpenClaw 生成图片和视频，岂不是正合适？

## OpenClaw 已经有生成图片和视频的工作流

OpenClaw 本身已经提供了原生的图片和视频生成工作流，可以分别配置默认的 Image Model 和 Video Model。既然 Agnes 也提供对应模型，我就想当然地把它填进了配置：

```json
{
  "models": {
    "providers": {
      "agnes": {
        "baseUrl": "https://apihub.agnes-ai.com/v1",
        "apiKey": "${AGNES_API_KEY}"
      }
    }
  },
  "agents": {
    "defaults": {
      "mediaModels": {
        "image": {
          "primary": "agnes/agnes-image-2.5-flash"
        },
        "video": {
          "primary": "agnes/agnes-video-2.5-flash",
          "timeoutMs": 300000
        }
      }
    }
  }
}
```

看起来模型地址、API Key、图片模型和视频模型都配好了，结果一试，不能用。

原因也不复杂：OpenClaw 的配置里虽然可以写一个叫 `agnes` 的模型服务，但它的图片和视频生成流程里并没有 Agnes Provider。只有配置，没有负责实际调用 Agnes 接口的适配代码，OpenClaw 当然不知道该怎样完成请求、怎样等待视频生成，又该怎样接收最后的图片和视频。

## 先做了一个能用的 Skill

既然原生方式暂时走不通，我就先想了一个比较直接的办法：做成 Skill，让 Agent 自己调用 Agnes API。

这条路确实很快就走通了。需要生成图片或视频时，Agent 按照 Skill 的说明组织参数、请求接口，再把结果取回来。

不过实际用了一段时间以后，我发现它并不算顺手。因为整个过程依赖 Agent 按照说明一步一步执行，偶尔会出现调用不稳定的情况；尤其是生成完成以后，怎样拿回文件、怎样在对话里展示图片或视频，效果并不总是理想。

它更像是“教 Agent 临时完成一项任务”，而不是 OpenClaw 原生认识的图片和视频模型：

```text
OpenClaw → Agent 阅读 Skill → 调用 Agnes API → 自己处理和展示结果
```

能用是能用，但总感觉绕了一圈。

## 官方没有，那就自己补一个 Provider

既然 OpenClaw 已经有标准的图片和视频生成流程，只是缺少 Agnes Provider，那能不能直接补一个？

我把这个想法交给 Codex，让它按照 OpenClaw 的 Provider 接口和 Agnes API 做一个适配。结果发现这件事并没有想象中复杂，很快就做出了第一版。

改成 Provider 以后，调用链路就变成了：

```text
OpenClaw → 原生图片/视频生成能力 → Agnes Provider → Agnes API
```


这个插件的大部分代码都是 Codex 写的。我主要负责说明需求、在实际环境中安装试用、反馈问题和确认最终效果，大概逻辑就是，在 OpenClaw 和 Agnes 之间补上了缺失的那一层Provider，利用的是OpenClaw提供的插件机制，注册对应的agnes的provider，处理相关请求，转换之后发送给agnes，收到响应之后，转换为OpenClaw可以识别的实体，然后OpenClaw内部直接进行统一的图片和视频工作流，再呈现给会话，或者发送到渠道。

## 目前可以做什么

插件当前版本是 `0.1.0`，明确验证过的模型是：

- 图片：`agnes-image-2.5-flash`
- 视频：`agnes-video-2.5-flash`

现在可以通过 OpenClaw 原生能力完成文生图和文生视频，也支持视频生成中的首尾帧与参考图：

- 文生图；
- 文生视频；
- 使用首帧和尾帧生成视频；
- 使用最多 5 张参考图生成视频。

第一版只做了我自己会用到的功能。图片编辑、视频转视频、音频、水印等能力暂时没有支持，也没必要为了显得功能多而全部塞进去，够用就好了，说不定哪天官方就出了自己的provider。

## 如果其他人也想用的话：安装和配置

项目目前没有发布到 npm，可以直接从 GitHub 获取源码：

```bash
git clone https://github.com/sandy1108/openclaw-agnes-provider.git
cd openclaw-agnes-provider

npm install
npm run typecheck
npm test
npm run build
```

构建完成后，把插件目录加入 OpenClaw 的 `plugins.load.paths`，将 `agnes` 加入 `plugins.allow`，再启用插件。Agnes 的地址、API Key 和默认媒体模型仍然使用前面的 OpenClaw 配置；真实 API Key 最好通过环境变量注入，不要写进仓库。

最后可以用下面几条命令检查 Provider 是否已经被 OpenClaw 识别：

```bash
openclaw plugins inspect agnes
openclaw config validate
openclaw gateway health
openclaw infer image providers
openclaw infer video providers
```

完整安装、配置和调用示例都放在项目 README 里。OpenClaw 和 Agnes 还在更新，实际使用时仍然要注意版本变化。

## 顺手开源一下

做完以后，我顺手把项目放到了 GitHub，采用 MIT License：

[sandy1108/openclaw-agnes-provider](https://github.com/sandy1108/openclaw-agnes-provider)

它目前没有发布到 npm，直接从源码安装即可。这个项目本身并不复杂，也谈不上什么厉害的插件，就是刚好补上了 OpenClaw 和 Agnes 之间缺失的那一层，让我可以更方便地白嫖图片和视频生成额度。而且OpenClaw的插件机制和Agnes的API全都有非常明确的文档说明，对于Codex来说，没有比这更明确的任务了，因此直接一步到位就可以用了。

最开始写的那个 Skill 也已经被我暂时禁用了，平时统一使用原生 Provider。能少绕一层，而且用起来更稳定，这次小折腾的目的也就达到了。
