---
title: 用 shields.io 给项目加点好看的徽章
categories: 
  - 瞎折腾系列
excerpt: 原来是一个叫 **shields.io** 的网站做的，而且用法很简单。这里赶紧记下来，免得以后忘了，也顺便分享给可能需要的朋友。
date: 2025-08-10 15:22:00
tags: 
---

# 今天学了个新东西：用 shields.io 给项目加点好看的徽章

我最近在逛 GitHub 的时候，总看到别人的项目 `README` 文件里有一排排很漂亮的小徽章，像这样：

![image-shield-demo.png](image-shield-demo.png)

感觉还挺好看，简洁明了，而且很多项目都在用。这似乎也不是一个图片，是动态生成的。正好闲来无事具体研究了一下，发现原来是一个叫 **shields.io** 的网站提供出来的这个徽章生成服务，而且用法很简单。于是记录下来，免得以后忘了，也顺便分享给可能需要的朋友。

## 所以，Shields.io 是个啥？

简单说，它就是一个能帮你在线生成小徽章的网站。

而且，它**完全免费，而且不用注册登录，直接改 URL 链接就能生成**！它会动态生成一张矢量图（SVG 格式），所以图片很清晰，加载也快。

## 核心玩法：改 URL 就行

最基础的格式长这样：

```
https://img.shields.io/badge/<标签>-<信息>-<颜色>

```

我把它拆开来看：

*   **`<标签>` (Label):** 就是徽章左边那块，背景一般是灰色。
*   **`<信息>` (Message):** 徽章右边的文字。
*   **`<颜色>` (Color):** 徽章右边的背景颜色。

比如我想做一个 `HarmonyOS` 的徽章，链接就可以这么写：

`https://img.shields.io/badge/HarmonyOS-NEXT-blue.svg`

效果就是这样：

![image-shield-harmonyos](https://img.shields.io/badge/HarmonyOS-NEXT-blue.svg)

是不是一下就明白了？把链接里的文字和颜色换成自己想要的就行。

## 给徽章换个颜色

默认的颜色可能看腻了，换颜色也超简单。

#### 1. 用它自带的颜色名

它内置了一些常见的颜色名，比如 `green`, `yellow`, `orange`, `red`, `blue`，直接写在 URL 最后就行。

`https://img.shields.io/badge/status-ok-green`

效果是这样：

![image-shield-green](https://img.shields.io/badge/status-ok-green)

#### 2. 用十六进制颜色码（更自由）

如果想用一些特别的颜色，可以直接用十六进制颜色代码（Hex Code），这样选择就多太多了。

**有一点要注意：不需要加 `#` 号！**

比如，我想用一个粉色 `ff69b4`：

`https://img.shields.io/badge/made%20with-love-ff69b4`

效果是这样：

![image-shield-pink](https://img.shields.io/badge/made%20with-love-ff69b4)

（小提示：如果你的文字里有空格，最好用 `%20` 来代替，这是 URL 编码的规范写法。）

## 总结一下

总之，今天又 get 了一个新技能。以后自己的小项目也能用这些好看的徽章装点一下门面了。

这次只是记录了最基础的用法，其实官网上还有很多高级玩法，比如可以动态显示 GitHub 的 star 数、npm 包的下载量等等。如果你也觉得有意思，可以去 [Shields.io](https://shields.io/) 官网看一看。
