---
title: 鸿蒙开发里各种Context
categories: 
  -  鸿蒙HarmonyOS启程之路
excerpt: 我在这篇文章中梳理了 HarmonyOS 中不同的 Context 类型。从总基类 `BaseContext`，到提供应用信息的 `Context` 和处理生命周期的 `ApplicationContext`，再到 `AbilityStageContext` 和功能丰富的 `UIAbilityContext`。最后，我还单独介绍了用于显示弹窗等 UI 操作的 `UIContext` 工具类，并说明了它们之间如何相互获取。
date: 2025-06-05 23:45:12
tags: 
---



## 总体概述图

<https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V14/application-context-stage-V14>

## 具体列举

### BaseContext

总基类，获取stageMode

### Context

> https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-context-stage

二级基类，应用信息，目录信息，eventHub

### ApplicationContext

on各种事件，比如abilityLifeCycle

### AbilityStageContext

AbilityStage中的Context，只能获取一些应用基本信息，或者屏幕方向等。可以通过getApplicationContext进一步获取ApplicationContext。

### UIAbilityContext

1.  通过UIAbility中的this.context获取。
2.  通过UIAbility内Page或Component内使用getContext(this)

可以startAbility，terminateSelf等等，可以通过getApplicationContext进一步获取ApplicationContext。

### UIContext（@ohos.arkui.UIContext），单独一种。

> https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-arkui-uicontext#uicontext


是ohos.window中的getUIContext()获取，跟上面的Context没有直接关系，类似一个工具类的样子。

也可以通过getHostContext()进一步获取Context实例，即上面BaseContext的子类们的类型。具体类型要看在什么样的Ability中执行。

UIContext可以监听一些事件，显示一些dialog等
