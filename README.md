# GASDocumentation

[English](README_en.md) | **简体中文**

> **译者说明**
> - 本文是 [tranek/GASDocumentation](https://github.com/tranek/GASDocumentation) README 的非官方中文译文，基于上游 commit [`8f76c57`](https://github.com/tranek/GASDocumentation/commit/8f76c57)（对应 UE 5.3）。
> - 原文作者 Dan Kestranek（tranek），以 [MIT 许可](https://github.com/tranek/GASDocumentation/blob/master/LICENSE)发布；本译文同样遵循 MIT 许可，并保留原版权声明。
> - 术语约定：类名、类型名、函数名、资产名保留英文（如 `GameplayEffect`、`AttributeSet`）；通用概念译为中文，首次出现时括注英文。完整对照见文末[附录：术语对照表](#translation-glossary)。
> - 代码块采用中英对照：代码与英文注释保持原样，中文注释紧随其后另起一行。
> - 译文如有疏漏，以英文原文为准。

这是我对虚幻引擎 5 的 GameplayAbilitySystem 插件（GAS）的理解，并附带一个简单的多人联机示例项目。本文并非官方文档，本项目和我本人都与 Epic Games 没有任何关联，我也不对这些信息的准确性做任何保证。

本文档的目标是讲解 GAS 中的主要概念和类，并结合我的使用经验补充一些说明。社区用户之间流传着大量关于 GAS 的“口口相传的经验”（tribal knowledge），我希望在这里把自己掌握的全部分享出来。

示例项目与文档目前对应 **Unreal Engine 5.3**（UE5）。本文档也有适用于旧版虚幻引擎的分支，但它们已不再维护，可能存在 bug 或过时信息。请使用与你的引擎版本相匹配的分支。

[GASShooter](https://github.com/tranek/GASShooter) 是一个姊妹示例项目，演示了在多人 FPS/TPS 中使用 GAS 的进阶技巧。

最好的文档永远是插件源码。

<a name="table-of-contents"></a>
## 目录

> 1. [GameplayAbilitySystem 插件简介](#intro)
> 1. [示例项目](#sp)
> 1. [在项目中配置 GAS](#setup)
> 1. [GAS 概念](#concepts)  
>    4.1 [能力系统组件（ASC）](#concepts-asc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.1.1 [复制模式](#concepts-asc-rm)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.1.2 [创建与初始化](#concepts-asc-setup)  
>    4.2 [GameplayTag](#concepts-gt)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.2.1 [响应 GameplayTag 的变化](#concepts-gt-change)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.2.2 [从插件的 .ini 文件加载 GameplayTag](#concepts-gt-loadfromplugin)  
>    4.3 [属性（Attribute）](#concepts-a)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.1 [属性的定义](#concepts-a-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.2 [BaseValue 与 CurrentValue](#concepts-a-value)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.3 [元属性（Meta Attribute）](#concepts-a-meta)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.4 [响应属性变化](#concepts-a-changes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.5 [派生属性（Derived Attribute）](#concepts-a-derived)  
>    4.4 [属性集（AttributeSet）](#concepts-as)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.1 [属性集的定义](#concepts-as-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2 [属性集的设计](#concepts-as-design)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.1 [各自带有独立属性的子组件](#concepts-as-design-subcomponents)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.2 [在运行时添加和移除 AttributeSet](#concepts-as-design-addremoveruntime)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3 [物品属性（武器弹药）](#concepts-as-design-itemattributes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.1 [在物品上使用普通浮点数](#concepts-as-design-itemattributes-plainfloats)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.2 [在物品上使用 `AttributeSet`](#concepts-as-design-itemattributes-attributeset)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.3 [在物品上使用 `ASC`](#concepts-as-design-itemattributes-asc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.3 [定义属性](#concepts-as-attributes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.4 [初始化属性](#concepts-as-init)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.5 [PreAttributeChange()](#concepts-as-preattributechange)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.6 [PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.7 [OnAttributeAggregatorCreated()](#concepts-as-onattributeaggregatorcreated)  
>    4.5 [GameplayEffect](#concepts-ge)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.1 [GameplayEffect 的定义](#concepts-ge-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.2 [施加 GameplayEffect](#concepts-ge-applying)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.3 [移除 GameplayEffect](#concepts-ga-removing)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4 [GameplayEffect 修改器（Modifier）](#concepts-ge-mods)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4.1 [乘法与除法修改器](#concepts-ge-mods-multiplydivide)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4.2 [修改器上的 GameplayTag](#concepts-ge-mods-gameplaytags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.5 [GameplayEffect 的堆叠](#concepts-ge-stacking)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.6 [授予的能力](#concepts-ge-ga)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.7 [GameplayEffect 的标签](#concepts-ge-tags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.8 [免疫](#concepts-ge-immunity)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.9 [GameplayEffectSpec](#concepts-ge-spec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.9.1 [SetByCaller](#concepts-ge-spec-setbycaller)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.10 [GameplayEffectContext](#concepts-ge-context)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.11 [修改器幅度计算（MMC）](#concepts-ge-mmc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12 [GameplayEffect 执行计算（ExecCalc）](#concepts-ge-ec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1 [向执行计算传递数据](#concepts-ge-ec-senddata)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.1 [SetByCaller](#concepts-ge-ec-senddata-setbycaller)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.2 [后备数据为属性的计算修改器](#concepts-ge-ec-senddata-backingdataattribute)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.3 [后备数据为临时变量的计算修改器](#concepts-ge-ec-senddata-backingdatatempvariable)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.4 [GameplayEffectContext](#concepts-ge-ec-senddata-effectcontext)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.13 [自定义施加条件（Custom Application Requirement）](#concepts-ge-car)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.14 [消耗 GameplayEffect](#concepts-ge-cost)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15 [冷却 GameplayEffect](#concepts-ge-cooldown)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.1 [获取冷却 GameplayEffect 的剩余时间](#concepts-ge-cooldown-tr)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.2 [监听冷却的开始与结束](#concepts-ge-cooldown-listen)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.3 [预测冷却](#concepts-ge-cooldown-prediction)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.16 [修改激活中 GameplayEffect 的持续时间](#concepts-ge-duration)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.17 [在运行时创建动态 GameplayEffect](#concepts-ge-dynamic)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.18 [GameplayEffect 容器](#concepts-ge-containers)  
>    4.6 [GameplayAbility](#concepts-ga)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1 [GameplayAbility 的定义](#concepts-ga-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.1 [复制策略（Replication Policy）](#concepts-ga-definition-reppolicy)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.2 [服务器响应远程能力取消（Server Respects Remote Ability Cancellation）](#concepts-ga-definition-remotecancel)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.3 [直接复制输入（Replicate Input Directly）](#concepts-ga-definition-repinputdirectly)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.2 [将输入绑定到 ASC](#concepts-ga-input)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.2.1 [绑定输入但不激活能力](#concepts-ga-input-noactivate)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.3 [授予能力](#concepts-ga-granting)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4 [激活能力](#concepts-ga-activating)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4.1 [被动能力](#concepts-ga-activating-passive)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4.2 [激活失败标签（Activation Failed Tags）](#concepts-ga-activating-failedtags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.5 [取消能力](#concepts-ga-cancelabilities)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.6 [获取激活中的能力](#concepts-ga-definition-activeability)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.7 [实例化策略（Instancing Policy）](#concepts-ga-instancing)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.8 [网络执行策略（Net Execution Policy）](#concepts-ga-net)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.9 [能力标签](#concepts-ga-tags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.10 [GameplayAbilitySpec](#concepts-ga-spec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.11 [向能力传递数据](#concepts-ga-data)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.12 [能力的消耗与冷却](#concepts-ga-commit)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.13 [能力升级](#concepts-ga-leveling)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.14 [能力集（Ability Set）](#concepts-ga-sets)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.15 [能力批处理（Ability Batching）](#concepts-ga-batching)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.16 [网络安全策略（Net Security Policy）](#concepts-ga-netsecuritypolicy)  
>    4.7 [AbilityTask](#concepts-at)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.1 [AbilityTask 的定义](#concepts-at-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.2 [自定义 AbilityTask](#concepts-at-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.3 [使用 AbilityTask](#concepts-at-using)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.4 [根运动源（Root Motion Source）AbilityTask](#concepts-at-rms)  
>    4.8 [GameplayCue](#concepts-gc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.1 [GameplayCue 的定义](#concepts-gc-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.2 [触发 GameplayCue](#concepts-gc-trigger)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.3 [本地 GameplayCue](#concepts-gc-local)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.4 [GameplayCue 参数](#concepts-gc-parameters)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.5 [GameplayCue 管理器（GameplayCueManager）](#concepts-gc-manager)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.6 [阻止 GameplayCue 触发](#concepts-gc-prevention)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7 [GameplayCue 批处理](#concepts-gc-batching)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7.1 [手动 RPC](#concepts-gc-batching-manualrpc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7.2 [一个 GE 上挂多个 GC](#concepts-gc-batching-gcsonge)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.8 [GameplayCue 事件](#concepts-gc-events)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.9 [GameplayCue 的可靠性](#concepts-gc-reliability)  
>    4.9 [能力系统全局对象（AbilitySystemGlobals）](#concepts-asg)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.9.1 [InitGlobalData()](#concepts-asg-initglobaldata)  
>    4.10 [预测（Prediction）](#concepts-p)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.1 [预测键（Prediction Key）](#concepts-p-key)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.2 [在能力中创建新的预测窗口](#concepts-p-windows)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.3 [预测性地生成 Actor](#concepts-p-spawn)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.4 [GAS 中预测的未来](#concepts-p-future)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.5 [网络预测插件（Network Prediction Plugin）](#concepts-p-npp)  
>    4.11 [目标选择（Targeting）](#concepts-targeting)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.1 [目标数据（Target Data）](#concepts-targeting-data)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.2 [目标 Actor（Target Actor）](#concepts-targeting-actors)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.3 [目标数据过滤器（Target Data Filter）](#concepts-target-data-filters)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.4 [GameplayAbility 世界准星（World Reticle）](#concepts-targeting-reticles)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.5 [GameplayEffect 容器的目标选择](#concepts-targeting-containers)  
> 1. [常见能力与效果的实现](#cae)  
>    5.1 [眩晕（Stun）](#cae-stun)  
>    5.2 [冲刺（Sprint）](#cae-sprint)  
>    5.3 [开镜瞄准（Aim Down Sights）](#cae-ads)  
>    5.4 [吸血（Lifesteal）](#cae-ls)  
>    5.5 [在客户端和服务器上生成随机数](#cae-random)  
>    5.6 [暴击（Critical Hit）](#cae-crit)  
>    5.7 [不堆叠的 GameplayEffect，且只有幅度最大的那个真正作用于目标](#cae-nonstackingge)  
>    5.8 [游戏暂停时生成目标数据](#cae-paused)  
>    5.9 [单键交互系统](#cae-onebuttoninteractionsystem)  
> 1. [调试 GAS](#debugging)  
>    6.1 [showdebug abilitysystem](#debugging-sd)  
>    6.2 [Gameplay 调试器（Gameplay Debugger）](#debugging-gd)  
>    6.3 [GAS 日志](#debugging-log)  
> 1. [优化](#optimizations)  
>    7.1 [能力批处理](#optimizations-abilitybatching)  
>    7.2 [GameplayCue 批处理](#optimizations-gameplaycuebatching)  
>    7.3 [AbilitySystemComponent 的复制模式](#optimizations-ascreplicationmode)  
>    7.4 [属性代理复制（Attribute Proxy Replication）](#optimizations-attributeproxyreplication)  
>    7.5 [ASC 延迟加载](#optimizations-asclazyloading)  
> 1. [易用性建议](#qol)  
>    8.1 [GameplayEffect 容器](#qol-gameplayeffectcontainers)  
>    8.2 [用蓝图 AsyncTask 绑定 ASC 委托](#qol-asynctasksascdelegates)  
> 1. [疑难排查](#troubleshooting)  
>    9.1 [`LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`](#troubleshooting-notlocal)  
>    9.2 [`ScriptStructCache` 错误](#troubleshooting-scriptstructcache)  
>    9.3 [动画蒙太奇没有复制到客户端](#troubleshooting-replicatinganimmontages)  
>    9.4 [对蓝图 Actor 执行 Duplicate（创建副本）会把 AttributeSet 置为 nullptr](#troubleshooting-duplicatingblueprintactors)  
>    9.5 [无法解析的外部符号 UEPushModelPrivate::MarkPropertyDirty(int,int)](#troubleshooting-unresolvedexternalsymbolmarkpropertydirty)  
>    9.6 [枚举名现在以路径名表示](#troubleshooting-enumnamesarenowpathnames)  
> 1. [常见 GAS 缩写](#acronyms)
> 1. [其他资源](#resources)  
>    11.1 [Epic Games 的 Dave Ratti 问答](#resources-daveratti)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11.1.1 [社区问题（一）](#resources-daveratti-community1)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11.1.2 [社区问题（二）](#resources-daveratti-community2)  
> 1. [GAS 更新日志](#changelog)  
>    * [5.3](#changelog-5.3)  
>    * [5.2](#changelog-5.2)  
>    * [5.1](#changelog-5.1)  
>    * [5.0](#changelog-5.0)  
>    * [4.27](#changelog-4.27)  
>    * [4.26](#changelog-4.26)  
>    * [4.25.1](#changelog-4.25.1)  
>    * [4.25](#changelog-4.25)  
>    * [4.24](#changelog-4.24)
>
> [附录：术语对照表](#translation-glossary)

<a name="intro"></a>
## 1. GameplayAbilitySystem 插件简介
摘自[官方文档](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)：
>Gameplay Ability System 是一个高度灵活的框架，用于构建 RPG 或 MOBA 游戏中常见的那类能力与属性。你可以为游戏中的角色构建可供使用的动作或被动能力；构建因这些动作而累积或削减各种属性的状态效果；实现“冷却”计时器或资源消耗来限制这些动作的使用；修改能力的等级以及每一级对应的效果；激活粒子或音效；等等。简单来说，这套系统可以帮助你设计、实现并高效地联网同步各种游戏内能力——简单的如跳跃，复杂的如任何现代 RPG 或 MOBA 游戏里你最喜欢的角色的整套技能。

GameplayAbilitySystem 插件由 Epic Games 开发，随虚幻引擎一同提供。它已经在《Paragon》《Fortnite》等 AAA 商业游戏中经受过实战检验。

该插件为单人和多人游戏提供了开箱即用的解决方案，可用于：
* 实现基于等级的角色能力或技能，并可选配消耗和冷却（[GameplayAbilities](#concepts-ga)）
* 操作 Actor 所拥有的数值型 `Attributes`（[属性](#concepts-a)）
* 向 Actor 施加状态效果（[GameplayEffects](#concepts-ge)）
* 向 Actor 添加 `GameplayTags`（[GameplayTags](#concepts-gt)）
* 生成视觉或声音效果（[GameplayCues](#concepts-gc)）
* 上述所有内容的网络复制

在多人游戏中，GAS 支持对以下内容进行[客户端预测](#concepts-p)：
* 能力激活
* 播放动画蒙太奇（Montage）
* `Attributes` 的变化
* 添加 `GameplayTags`
* 生成 `GameplayCues`
* 通过与 `CharacterMovementComponent` 相连的 `RootMotionSource` 函数实现的移动。

**GAS 必须在 C++ 中配置**，但 `GameplayAbilities` 和 `GameplayEffects` 可以由策划在蓝图中创建。

GAS 目前存在的问题：
* `GameplayEffect` 的延迟校正（latency reconciliation）：无法预测能力冷却，导致对于冷却很短的能力，高延迟玩家的射速会比低延迟玩家低。
* 无法预测 `GameplayEffects` 的移除。不过我们可以预测性地添加效果相反的 `GameplayEffects`，从而在效果上抵消它们。但这种做法并不总是合适或可行，所以这仍然是个问题。
* 缺少样板模板、多人联机示例和文档。希望本文能在这方面有所帮助！

**[⬆ 返回目录](#table-of-contents)**

<a name="sp"></a>
## 2. 示例项目
本文档附带一个多人联机第三人称射击示例项目，面向刚接触 GameplayAbilitySystem 插件、但并非虚幻引擎新手的读者。读者应当已经掌握 C++、蓝图、UMG、网络复制以及 UE 中的其他中级主题。本项目示范了如何搭建一个基础的、支持多人联机的第三人称射击项目：玩家/AI 控制的英雄把 `AbilitySystemComponent`（`ASC`）放在 `PlayerState` 类上，AI 控制的小兵则把 `ASC` 放在 `Character` 类上。

本项目的目标是在保持简单的同时展示 GAS 的基础知识，并以注释详尽的代码演示一些大家经常问到的能力。由于面向初学者，项目没有涉及[预测投射物](#concepts-p-spawn)之类的进阶主题。

演示的概念：
* `ASC` 放在 `PlayerState` 上与放在 `Character` 上的对比
* 被复制的 `Attributes`
* 被复制的动画蒙太奇
* `GameplayTags`
* 在 `GameplayAbilities` 内部及外部施加和移除 `GameplayEffects`
* 施加经护甲减免后的伤害，改变角色的生命值
* `GameplayEffectExecutionCalculations`
* 眩晕效果
* 死亡与重生
* 在服务器上由能力生成 Actor（投射物）
* 开镜瞄准和冲刺时，预测性地修改本地玩家的速度
* 冲刺时持续消耗耐力
* 消耗法力来施放能力
* 被动能力
* 堆叠 `GameplayEffects`
* 选择目标 Actor
* 在蓝图中创建的 `GameplayAbilities`
* 在 C++ 中创建的 `GameplayAbilities`
* 每个 `Actor` 一个实例（Instanced per Actor）的 `GameplayAbilities`
* 非实例化（Non-Instanced）的 `GameplayAbilities`（跳跃）
* Static `GameplayCues`（FireGun 投射物命中的粒子效果）
* Actor `GameplayCues`（冲刺和眩晕的粒子效果）

英雄类拥有以下能力：

| 能力                         | 输入绑定             | 是否预测    | C++ / 蓝图      | 说明                                                                                     |
| ---------------------------- | -------------------- | ----------- | --------------- | ---------------------------------------------------------------------------------------- |
| 跳跃（Jump）                 | 空格键               | 是          | C++             | 让英雄跳起来。                                                                           |
| 开枪（Gun）                  | 鼠标左键             | 否          | C++             | 从英雄的枪中发射一枚投射物。动画会被预测，但投射物不会。                                 |
| 开镜瞄准（Aim Down Sights）  | 鼠标右键             | 是          | 蓝图            | 按住按键期间，英雄走得更慢，镜头拉近，以便用枪更精确地射击。                             |
| 冲刺（Sprint）               | 左 Shift             | 是          | 蓝图            | 按住按键期间，英雄跑得更快，同时消耗耐力。                                               |
| 前冲（Forward Dash）         | Q                    | 是          | 蓝图            | 英雄消耗耐力向前冲刺。                                                                   |
| 被动护甲层（Passive Armor Stacks） | 被动           | 否          | 蓝图            | 英雄每 4 秒获得一层护甲，最多 4 层。受到伤害会移除一层护甲。                             |
| 陨石（Meteor）               | R                    | 否          | 蓝图            | 玩家选定一个位置，向敌人砸下一颗陨石，造成伤害并使其眩晕。目标选择会被预测，但陨石的生成不会。 |

`GameplayAbilities` 用 C++ 还是蓝图创建并不重要。这里两种方式混合使用，是为了分别示范在两种语言中如何实现。

小兵没有任何预定义的 `GameplayAbilities`。红色小兵的生命回复更快，而蓝色小兵的初始生命值更高。

在 `GameplayAbility` 的命名上，我用后缀 `_BP` 表示该 `GameplayAbility` 的逻辑是在蓝图中编写的；没有后缀则表示逻辑是用 C++ 编写的。

**蓝图资产命名前缀**

| 前缀        | 资产类型            |
| ----------- | ------------------- |
| GA_         | GameplayAbility     |
| GC_         | GameplayCue         |
| GE_         | GameplayEffect      |

**[⬆ 返回目录](#table-of-contents)**

<a name="setup"></a>
## 3. 在项目中配置 GAS
配置一个使用 GAS 的项目，基本步骤如下：
1. 在编辑器中启用 GameplayAbilitySystem 插件
1. 编辑 `YourProjectName.Build.cs`，把 `"GameplayAbilities", "GameplayTags", "GameplayTasks"` 加入 `PrivateDependencyModuleNames`
1. 刷新/重新生成 Visual Studio 工程文件
1. 从 4.24 到 5.2，必须调用 `UAbilitySystemGlobals::Get().InitGlobalData()` 才能使用 [`TargetData`](#concepts-targeting-data)。示例项目在 `UAssetManager::StartInitialLoading()` 中完成这一步。从 5.3 开始，这一调用会自动完成。详见 [`InitGlobalData()`](#concepts-asg-initglobaldata)。

启用 GAS 只需要做这些。接下来，为你的 `Character` 或 `PlayerState` 添加 [`ASC`](#concepts-asc) 和 [`AttributeSet`](#concepts-as)，然后就可以开始制作 [`GameplayAbilities`](#concepts-ga) 和 [`GameplayEffects`](#concepts-ge) 了！

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts"></a>
## 4. GAS 概念

#### 本章内容

> 4.1 [能力系统组件（ASC）](#concepts-asc)  
> 4.2 [GameplayTag](#concepts-gt)  
> 4.3 [属性（Attribute）](#concepts-a)  
> 4.4 [属性集（AttributeSet）](#concepts-as)  
> 4.5 [GameplayEffect](#concepts-ge)  
> 4.6 [GameplayAbility](#concepts-ga)  
> 4.7 [AbilityTask](#concepts-at)  
> 4.8 [GameplayCue](#concepts-gc)  
> 4.9 [能力系统全局对象（AbilitySystemGlobals）](#concepts-asg)  
> 4.10 [预测（Prediction）](#concepts-p)

<a name="concepts-asc"></a>
### 4.1 能力系统组件（ASC）
`AbilitySystemComponent`（`ASC`）是 GAS 的核心。它是一个 `UActorComponent`（[`UAbilitySystemComponent`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemComponent/index.html)），负责处理与该系统的所有交互。任何想要使用 [`GameplayAbilities`](#concepts-ga)、拥有 [`Attributes`](#concepts-a) 或接收 [`GameplayEffects`](#concepts-ge) 的 `Actor`，都必须挂载一个 `ASC`。这些对象都存放在 `ASC` 内部，并由 `ASC` 管理和复制（`Attributes` 例外，它们由各自的 [`AttributeSet`](#concepts-as) 负责复制）。开发者通常会子类化 `ASC`，但这不是必需的。

挂载 `ASC` 的那个 `Actor` 称为该 `ASC` 的 `OwnerActor`；`ASC` 在场景中的实体表现 `Actor` 称为 `AvatarActor`。`OwnerActor` 和 `AvatarActor` 可以是同一个 `Actor`，例如 MOBA 游戏中简单的 AI 小兵；也可以是不同的 `Actor`，例如 MOBA 游戏中玩家控制的英雄，其 `OwnerActor` 是 `PlayerState`，`AvatarActor` 是英雄的 `Character` 类。大多数 `Actor` 会把 `ASC` 挂在自己身上。如果你的 `Actor` 会重生，并且需要在两次生成之间保留 `Attributes` 或 `GameplayEffects`（比如 MOBA 里的英雄），那么 `ASC` 最理想的位置是 `PlayerState`。

**注意：** 如果 `ASC` 放在 `PlayerState` 上，就需要调高 `PlayerState` 的 `NetUpdateFrequency`。`PlayerState` 上这个值默认非常低，可能导致 `Attributes`、`GameplayTags` 等内容的变化要延迟一段时间才到达客户端，让人感觉卡顿。一定要启用[`自适应网络更新频率（Adaptive Network Update Frequency）`](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/Properties/index.html#adaptivenetworkupdatefrequency)，Fortnite 就用了它。

如果 `OwnerActor` 和 `AvatarActor` 是不同的 `Actor`，两者都应实现 `IAbilitySystemInterface`。该接口只有一个必须重写（override）的函数 `UAbilitySystemComponent* GetAbilitySystemComponent() const`，它返回指向自身 `ASC` 的指针。系统内部各个 `ASC` 之间正是通过查找这个接口函数来相互交互的。

`ASC` 把当前处于激活状态的 `GameplayEffects` 保存在 `FActiveGameplayEffectsContainer ActiveGameplayEffects` 中。

`ASC` 把已授予的 `Gameplay Abilities` 保存在 `FGameplayAbilitySpecContainer ActivatableAbilities` 中。每当你打算遍历 `ActivatableAbilities.Items` 时，务必在循环上方加上 `ABILITYLIST_SCOPE_LOCK();`，锁住列表，防止它在遍历期间（因移除能力而）发生变化。作用域内每个 `ABILITYLIST_SCOPE_LOCK();` 都会让 `AbilityScopeLockCount` 加一，离开作用域时再减一。不要在 `ABILITYLIST_SCOPE_LOCK();` 的作用域内移除能力（清除能力的函数内部会检查 `AbilityScopeLockCount`，列表被锁定时会阻止移除）。

<a name="concepts-asc-rm"></a>
### 4.1.1 复制模式
`ASC` 为 `GameplayEffects`、`GameplayTags` 和 `GameplayCues` 的复制定义了三种复制模式（Replication Mode）：`Full`、`Mixed` 和 `Minimal`。`Attributes` 则由各自的 `AttributeSet` 负责复制。

| 复制模式           | 适用场景                                | 说明                                                                                         |
| ------------------ | --------------------------------------- | -------------------------------------------------------------------------------------------- |
| `Full`             | 单人游戏                                | 每个 `GameplayEffect` 都会复制到每个客户端。                                                 |
| `Mixed`            | 多人游戏，玩家控制的 `Actor`            | `GameplayEffects` 只复制到拥有者客户端（owning client）。只有 `GameplayTags` 和 `GameplayCues` 会复制给所有人。 |
| `Minimal`          | 多人游戏，AI 控制的 `Actor`             | `GameplayEffects` 不会复制给任何人。只有 `GameplayTags` 和 `GameplayCues` 会复制给所有人。   |

**注意：** `Mixed` 复制模式要求 `OwnerActor` 的 `Owner` 是 `Controller`。`PlayerState` 的 `Owner` 默认就是 `Controller`，而 `Character` 的不是。如果在 `OwnerActor` 不是 `PlayerState` 的情况下使用 `Mixed` 复制模式，就需要用一个有效的 `Controller` 对 `OwnerActor` 调用 `SetOwner()`。

从 4.24 开始，`PossessedBy()` 会把 `Pawn` 的 Owner 设置为新的 `Controller`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-asc-setup"></a>
### 4.1.2 创建与初始化
`ASC` 通常在 `OwnerActor` 的构造函数中创建，并显式标记为可复制。**这一步必须在 C++ 中完成**。

```c++
AGDPlayerState::AGDPlayerState()
{
	// Create ability system component, and set it to be explicitly replicated
	// 创建能力系统组件，并显式设置为可复制
	AbilitySystemComponent = CreateDefaultSubobject<UGDAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
	AbilitySystemComponent->SetIsReplicated(true);
	//...
}
```

`ASC` 需要在服务器和客户端两端都用它的 `OwnerActor` 和 `AvatarActor` 进行初始化。初始化应当在 `Pawn` 的 `Controller` 设置好之后（即控制（Possess）之后）进行。单人游戏只需要关心服务器这条路径。

对于 `ASC` 位于 `Pawn` 上的玩家控制角色，我通常在服务器端的 `Pawn::PossessedBy()` 中初始化，在客户端的 `PlayerController::AcknowledgePossession()` 中初始化。

```c++
void APACharacterBase::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->InitAbilityActorInfo(this, this);
	}

	// ASC MixedMode replication requires that the ASC Owner's Owner be the Controller.
	// ASC 的 Mixed 复制模式要求 ASC Owner 的 Owner 是 Controller。
	SetOwner(NewController);
}
```

```c++
void APAPlayerControllerBase::AcknowledgePossession(APawn* P)
{
	Super::AcknowledgePossession(P);

	APACharacterBase* CharacterBase = Cast<APACharacterBase>(P);
	if (CharacterBase)
	{
		CharacterBase->GetAbilitySystemComponent()->InitAbilityActorInfo(CharacterBase, CharacterBase);
	}

	//...
}
```

对于 `ASC` 位于 `PlayerState` 上的玩家控制角色，我通常在服务器端的 `Pawn::PossessedBy()` 中初始化，在客户端的 `Pawn::OnRep_PlayerState()` 中初始化。这样可以确保客户端上的 `PlayerState` 已经存在。

```c++
// Server only
// 仅在服务器上执行
void AGDHeroCharacter::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// Set the ASC on the Server. Clients do this in OnRep_PlayerState()
		// 在服务器上设置 ASC。客户端在 OnRep_PlayerState() 中做这件事
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// AI won't have PlayerControllers so we can init again here just to be sure. No harm in initing twice for heroes that have PlayerControllers.
		// AI 没有 PlayerController，所以这里再初始化一次以确保万无一失。对拥有 PlayerController 的英雄来说，初始化两次也没有坏处。
		PS->GetAbilitySystemComponent()->InitAbilityActorInfo(PS, this);
	}
	
	//...
}
```

```c++
// Client only
// 仅在客户端上执行
void AGDHeroCharacter::OnRep_PlayerState()
{
	Super::OnRep_PlayerState();

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// Set the ASC for clients. Server does this in PossessedBy.
		// 为客户端设置 ASC。服务器在 PossessedBy 中做这件事。
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// Init ASC Actor Info for clients. Server will init its ASC when it possesses a new Actor.
		// 为客户端初始化 ASC 的 Actor Info。服务器会在控制新的 Actor 时初始化它的 ASC。
		AbilitySystemComponent->InitAbilityActorInfo(PS, this);
	}

	// ...
}
```

如果你看到错误信息 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`，说明你没有在客户端初始化 `ASC`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gt"></a>
### 4.2 GameplayTag
[`FGameplayTags`](https://docs.unrealengine.com/en-US/API/Runtime/GameplayTags/FGameplayTag/index.html) 是形如 `Parent.Child.Grandchild...` 的层级化名称，注册在 `GameplayTagManager` 中。这些标签在对对象的状态进行分类和描述时极其有用。例如，角色被眩晕时，我们可以在眩晕持续期间给它一个 `State.Debuff.Stun` `GameplayTag`。

你会发现，过去用布尔值或枚举处理的东西，渐渐都换成了 `GameplayTags`，而布尔逻辑则变成了判断对象是否拥有某些 `GameplayTags`。

给对象添加标签时，如果它有 `ASC`，我们通常会把标签加到 `ASC` 上，这样 GAS 才能与这些标签交互。`UAbilitySystemComponent` 实现了 `IGameplayTagAssetInterface`，提供了访问其所拥有 `GameplayTags` 的函数。

多个 `GameplayTags` 可以存放在一个 `FGameplayTagContainer` 中。相比 `TArray<FGameplayTag>`，更推荐使用 `GameplayTagContainer`，因为 `GameplayTagContainers` 内置了一些提升效率的机制。标签本身是标准的 `FName`，但如果在项目设置中启用了 `Fast Replication`（快速复制），它们在 `FGameplayTagContainers` 中就能被高效地打包在一起进行复制。`Fast Replication` 要求服务器和客户端拥有相同的 `GameplayTags` 列表。这通常不成问题，所以你应该启用这个选项。`GameplayTagContainers` 也可以返回一个 `TArray<FGameplayTag>` 用于遍历。

存放在 `FGameplayTagCountContainer` 中的 `GameplayTags` 有一个 `TagMap`，记录该 `GameplayTag` 的实例数量。`FGameplayTagCountContainer` 里可能仍然存在某个 `GameplayTag`，但它的 `TagMapCount` 已经是零。调试时如果发现 `ASC` 还“拥有”某个 `GameplayTag`，可能就是这种情况。`HasTag()`、`HasMatchingTag()` 以及类似函数都会检查 `TagMapCount`，当 `GameplayTag` 不存在或其 `TagMapCount` 为零时返回 false。

`GameplayTags` 必须预先在 `DefaultGameplayTags.ini` 中定义。虚幻编辑器在项目设置中提供了一个界面，让开发者无需手动编辑 `DefaultGameplayTags.ini` 就能管理 `GameplayTags`。`GameplayTag` 编辑器可以创建、重命名、搜索引用以及删除 `GameplayTags`。

![项目设置中的 GameplayTag 编辑器](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaytageditor.png)

搜索 `GameplayTag` 引用会在编辑器中打开熟悉的 `Reference Viewer`（引用查看器）图表，显示所有引用该 `GameplayTag` 的资产。不过它不会显示引用该 `GameplayTag` 的 C++ 类。

重命名 `GameplayTags` 会创建一个重定向（redirect），使仍然引用原 `GameplayTag` 的资产能够重定向到新的 `GameplayTag`。如果可能，我更倾向于新建一个 `GameplayTag`，手动把所有引用更新到新的 `GameplayTag`，然后删除旧的 `GameplayTag`，以免产生重定向。

除了 `Fast Replication`，`GameplayTag` 编辑器还有一个选项，可以填入经常被复制的 `GameplayTags`，对它们做进一步优化。

由 `GameplayEffect` 添加的 `GameplayTags` 会被复制。`ASC` 还允许你添加 `LooseGameplayTags`（松散标签），它们不会被复制，必须手动管理。示例项目对 `State.Dead` 使用了 `LooseGameplayTag`，这样拥有者客户端在自身生命值降到零时能立即做出响应。重生时会手动把 `TagMapCount` 重新设为零。只有在处理 `LooseGameplayTags` 时才应手动调整 `TagMapCount`。比起手动调整 `TagMapCount`，更推荐使用 `UAbilitySystemComponent::AddLooseGameplayTag()` 和 `UAbilitySystemComponent::RemoveLooseGameplayTag()` 函数。

在 C++ 中获取一个 `GameplayTag` 的引用：
```c++
FGameplayTag::RequestGameplayTag(FName("Your.GameplayTag.Name"))
```

对于更高级的 `GameplayTag` 操作，比如获取父级或子级 `GameplayTags`，请查看 `GameplayTagManager` 提供的函数。要访问 `GameplayTagManager`，需包含 `GameplayTagManager.h`，并通过 `UGameplayTagManager::Get().FunctionName` 调用。`GameplayTagManager` 实际上把 `GameplayTags` 存储为关系节点（父、子等），比反复进行字符串操作和比较的处理速度更快。

`GameplayTags` 和 `GameplayTagContainers` 可以使用可选的 `UPROPERTY` 说明符 `Meta = (Categories = "GameplayCue")`，在蓝图中过滤标签，只显示父标签为 `GameplayCue` 的 `GameplayTags`。如果你明确知道某个 `GameplayTag` 或 `GameplayTagContainer` 变量只应该用于 `GameplayCues`，这个说明符就很有用。

另外，还有一个单独的结构体 `FGameplayCueTag`，它封装了一个 `FGameplayTag`，并且在蓝图中会自动过滤 `GameplayTags`，只显示父标签为 `GameplayCue` 的标签。

如果想过滤函数中的 `GameplayTag` 参数，可以使用 `UFUNCTION` 说明符 `Meta = (GameplayTagFilter = "GameplayCue")`。函数中的 `GameplayTagContainer` 参数无法被过滤。如果你想修改引擎以支持这一点，可以参考 `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagGraphPin.cpp` 中的 `SGameplayTagGraphPin::ParseDefaultValueData()`：它调用 `FilterString = UGameplayTagsManager::Get().GetCategoriesMetaFromField(PinStructType);`，并在 `SGameplayTagGraphPin::GetListContent()` 中把 `FilterString` 传给 `SGameplayTagWidget`。而 `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagContainerGraphPin.cpp` 中这些函数的 `GameplayTagContainer` 版本并不会检查元数据字段属性，也不会传递过滤条件。

示例项目大量使用了 `GameplayTags`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gt-change"></a>
### 4.2.1 响应 GameplayTag 的变化
`ASC` 提供了一个在 `GameplayTags` 被添加或移除时触发的委托（delegate）。它接受一个 `EGameplayTagEventType` 参数，可以指定只在 `GameplayTag` 被添加/移除时触发，或者在该 `GameplayTag` 的 `TagMapCount` 发生任何变化时都触发。

```c++
AbilitySystemComponent->RegisterGameplayTagEvent(FGameplayTag::RequestGameplayTag(FName("State.Debuff.Stun")), EGameplayTagEventType::NewOrRemoved).AddUObject(this, &AGDPlayerState::StunTagChanged);
```

回调函数的参数是该 `GameplayTag` 以及新的 `TagCount`。
```c++
virtual void StunTagChanged(const FGameplayTag CallbackTag, int32 NewCount);
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gt-loadfromplugin"></a>
### 4.2.2 从插件的 .ini 文件加载 GameplayTag
如果你创建的插件带有自己的、包含 `GameplayTags` 的 .ini 文件，可以在插件的 `StartupModule()` 函数中加载该插件的 `GameplayTag` .ini 目录。

例如，虚幻引擎自带的 CommonConversation 插件就是这样做的：

```c++
void FCommonConversationRuntimeModule::StartupModule()
{
	TSharedPtr<IPlugin> ThisPlugin = IPluginManager::Get().FindPlugin(TEXT("CommonConversation"));
	check(ThisPlugin.IsValid());
	
	UGameplayTagsManager::Get().AddTagIniSearchPath(ThisPlugin->GetBaseDir() / TEXT("Config") / TEXT("Tags"));

	//...
}
```

如果该插件已启用，引擎启动时就会查找 `Plugins\CommonConversation\Config\Tags` 目录，把其中所有包含 `GameplayTags` 的 .ini 文件加载到你的项目中。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-a"></a>
### 4.3 属性（Attribute）

<a name="concepts-a-definition"></a>
#### 4.3.1 属性的定义
`Attributes` 是由结构体 [`FGameplayAttributeData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayAttributeData/index.html) 定义的浮点数值。它们可以表示任何东西：角色的生命值、角色的等级、一瓶药水剩余的使用次数等等。只要是属于某个 `Actor`、与游戏玩法相关的数值，都应该考虑用 `Attribute` 来表示。`Attributes` 通常只应由 [`GameplayEffects`](#concepts-ge) 修改，这样 ASC 才能[预测](#concepts-p)这些变化。

`Attributes` 由 [`AttributeSet`](#concepts-as) 定义并存放在其中。`AttributeSet` 负责复制那些被标记为需要复制的 `Attributes`。关于如何定义 `Attributes`，请参阅 [`AttributeSets`](#concepts-as) 一节。

**提示：** 如果不希望某个 `Attribute` 出现在编辑器的 `Attributes` 列表中，可以使用 `Meta = (HideInDetailsView)` 这个`属性说明符（property specifier）`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-a-value"></a>
#### 4.3.2 BaseValue 与 CurrentValue
一个 `Attribute` 由两个值组成：`BaseValue`（基础值）和 `CurrentValue`（当前值）。`BaseValue` 是 `Attribute` 的永久值，而 `CurrentValue` 是 `BaseValue` 加上来自 `GameplayEffects` 的临时修改。例如，你的 `Character` 可能有一个移动速度 `Attribute`，其 `BaseValue` 为 600 单位/秒。由于目前还没有 `GameplayEffects` 修改移动速度，`CurrentValue` 也是 600 单位/秒。如果她获得一个临时的 +50 单位/秒移动速度增益（buff），`BaseValue` 仍保持 600 单位/秒不变，而 `CurrentValue` 变为 600 + 50，即 650 单位/秒。当移动速度增益到期后，`CurrentValue` 会恢复为 `BaseValue` 的 600 单位/秒。

GAS 初学者经常把 `BaseValue` 误当作 `Attribute` 的最大值来使用，这种做法是错误的。可能变化的最大值，或者会在能力、UI 中被引用的最大值，应该作为独立的 `Attributes` 来处理。对于写死的最大值和最小值，可以定义一个使用 `FAttributeMetaData` 的 `DataTable` 来设置最大值和最小值，但 Epic 在该结构体上方的注释中称其为“尚在开发中（work in progress）”。更多信息请参阅 `AttributeSet.h`。为避免混淆，我建议：会在能力或 UI 中被引用的最大值做成独立的 `Attributes`；仅用于钳制（clamp）`Attributes` 的写死的最大/最小值，则在 `AttributeSet` 中定义为写死的浮点数。关于 `Attributes` 的钳制：针对 `CurrentValue` 的变化，参见 [PreAttributeChange()](#concepts-as-preattributechange)；针对 `GameplayEffects` 引起的 `BaseValue` 变化，参见 [PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)。

对 `BaseValue` 的永久修改来自 `Instant`（瞬时）`GameplayEffects`，而 `Duration`（持续）和 `Infinite`（无限）`GameplayEffects` 修改的是 `CurrentValue`。周期性（Periodic）`GameplayEffects` 被当作瞬时 `GameplayEffects` 处理，修改的是 `BaseValue`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-a-meta"></a>
#### 4.3.3 元属性（Meta Attribute）
有些 `Attributes` 被用作临时值的占位符，专门用来与其他 `Attributes` 交互，这类属性称为 `Meta Attributes`（元属性）。例如，我们通常把伤害定义为一个 `Meta Attribute`。`GameplayEffect` 不直接修改生命值 `Attribute`，而是借助一个名为“伤害”的 `Meta Attribute` 作为占位符。这样，伤害值可以先在 [`GameplayEffectExecutionCalculation`](#concepts-ge-ec) 中被各种增益和减益（buff/debuff）修正，然后在 `AttributeSet` 中做进一步处理，例如先从当前的护盾 `Attribute` 中扣除伤害，最后再把剩余部分从生命值 `Attribute` 中扣除。伤害 `Meta Attribute` 不会在不同 `GameplayEffects` 之间保留，每一个 `GameplayEffect` 都会覆盖它。`Meta Attributes` 通常不进行复制。

`Meta Attributes` 为伤害、治疗之类的东西提供了良好的逻辑分离：把“我们造成了多少伤害？”和“这些伤害该怎么处理？”分开。这种逻辑分离意味着我们的 `Gameplay Effects` 和 `Execution Calculations` 不需要知道目标如何处理伤害。继续以伤害为例：`Gameplay Effect` 决定造成多少伤害，然后由 `AttributeSet` 决定如何处理这些伤害。并非所有角色都拥有相同的 `Attributes`，尤其是使用了 `AttributeSets` 子类时。基础 `AttributeSet` 类可能只有生命值 `Attribute`，而它的子类可能额外加了一个护盾 `Attribute`。带有护盾 `Attribute` 的子类在分配所受伤害时，做法会与基础 `AttributeSet` 类不同。

虽然 `Meta Attributes` 是一种不错的设计模式，但并非必须使用。如果你所有的伤害都只用一个 `Execution Calculation`，所有角色也共用同一个 `Attribute Set` 类，那么直接在 `Execution Calculation` 内部把伤害分配到生命值、护盾等属性上并直接修改这些 `Attributes`，或许也没问题。你牺牲的只是灵活性，而这对你来说也许可以接受。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-a-changes"></a>
#### 4.3.4 响应属性变化
要监听 `Attribute` 的变化以更新 UI 或其他游戏逻辑，可以使用 `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`。该函数返回一个委托，你可以绑定它，每当 `Attribute` 发生变化时就会自动调用。该委托提供一个 `FOnAttributeChangeData` 参数，其中包含 `NewValue`、`OldValue` 和 `FGameplayEffectModCallbackData`。**注意：** `FGameplayEffectModCallbackData` 只会在服务器上被设置。

```c++
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(AttributeSetBase->GetHealthAttribute()).AddUObject(this, &AGDPlayerState::HealthChanged);
```

```c++
virtual void HealthChanged(const FOnAttributeChangeData& Data);
```

示例项目在 `GDPlayerState` 上绑定了 `Attribute` 值变化委托，用来更新 HUD，并在生命值降到零时响应玩家死亡。

示例项目中包含一个自定义蓝图节点，它把上述功能封装成了一个 `ASyncTask`。`UI_HUD` UMG 控件用它来更新生命值、法力值和耐力值。这个 `AsyncTask` 会一直存活，直到手动调用 `EndTask()`，我们在 UMG 控件的 `Destruct` 事件中调用它。请参阅 `AsyncTaskAttributeChanged.h/cpp`。

![监听属性变化的蓝图节点](https://github.com/tranek/GASDocumentation/raw/master/Images/attributechange.png)

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-a-derived"></a>
#### 4.3.5 派生属性（Derived Attribute）
如果要让某个 `Attribute` 的部分或全部数值从一个或多个其他 `Attributes` 派生而来，可以使用一个 `Infinite` `GameplayEffect`，并为它配置一个或多个 `Attribute Based`（基于属性）或 [`MMC`](#concepts-ge-mmc) 类型的[`修改器（Modifiers）`](#concepts-ge-mods)。当它所依赖的 `Attribute` 更新时，`Derived Attribute`（派生属性）也会自动更新。

派生属性上所有 `Modifiers` 的最终计算公式与 `Modifier Aggregators`（修改器聚合器）的公式相同。如果你需要计算按特定顺序进行，就把它们全部放在一个 `MMC` 中完成。

```
((CurrentValue + Additive) * Multiplicitive) / Division
```

**注意：** 如果在 PIE 中以多个客户端运行，需要在编辑器偏好设置中关闭 `Run Under One Process`（在单进程下运行），否则在第一个客户端之外的其他客户端上，被依赖的 `Attributes` 更新时，`Derived Attributes` 不会随之更新。

在下面这个例子中，我们用一个 `Infinite` `GameplayEffect` 根据 `Attributes` `TestAttrB` 和 `TestAttrC` 来派生 `TestAttrA` 的值，公式为 `TestAttrA = (TestAttrA + TestAttrB) * ( 2 * TestAttrC)`。只要其中任何一个 `Attribute` 的值发生更新，`TestAttrA` 就会自动重新计算。

![派生属性示例](https://github.com/tranek/GASDocumentation/raw/master/Images/derivedattribute.png)

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-as"></a>
### 4.4 属性集（AttributeSet）

<a name="concepts-as-definition"></a>
#### 4.4.1 属性集的定义
`AttributeSet` 负责定义、持有并管理 `Attributes` 的变化。开发者应当从 [`UAttributeSet`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAttributeSet/index.html) 派生子类。在 `OwnerActor` 的构造函数中创建 `AttributeSet`，会自动把它注册到该 Actor 的 `ASC` 上。**这一步必须在 C++ 中完成**。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-as-design"></a>
#### 4.4.2 属性集的设计
一个 `ASC` 可以拥有一个或多个 `AttributeSets`。AttributeSet 的内存开销微乎其微，所以使用多少个 `AttributeSets` 只是一个代码组织上的选择，由开发者自行决定。

可以让游戏中的每个 `Actor` 都共用一个庞大的单体 `AttributeSet`，按需使用其中的属性，不用的属性直接忽略即可。

或者，你也可以使用多个 `AttributeSet`，每个代表一组 `Attributes`，根据需要有选择地添加到 `Actors` 上。例如，可以有一个存放生命相关 `Attributes` 的 `AttributeSet`，一个存放法力相关 `Attributes` 的 `AttributeSet`，以此类推。在 MOBA 游戏中，英雄可能需要法力，而小兵可能不需要。因此英雄会获得法力 `AttributeSet`，而小兵不会。

此外，子类化 `AttributeSets` 也是有选择地决定 `Actor` 拥有哪些 `Attributes` 的另一种方式。在内部，`Attributes` 以 `AttributeSetClassName.AttributeName` 的形式被引用。当你子类化一个 `AttributeSet` 时，来自父类的所有 `Attributes` 仍然以父类的类名作为前缀。

虽然可以拥有多个 `AttributeSet`，但同一个 `ASC` 上不应该有多个同类的 `AttributeSet`。如果存在多个同类的 `AttributeSet`，系统将不知道该用哪一个，只会随便挑一个。

<a name="concepts-as-design-subcomponents"></a>
##### 4.4.2.1 各自带有独立属性的子组件
如果一个 `Pawn` 上有多个可受伤害的组件，比如可以各自受损的护甲部件，并且你知道一个 `Pawn` 最多可能有多少个可受伤害的组件，我建议在一个 `AttributeSet` 上创建同样数量的生命值 `Attributes`——DamageableCompHealth0、DamageableCompHealth1 等等——用来表示这些可受伤害组件的逻辑“槽位”。在可受伤害组件的类实例中记录槽位编号，让 `GameplayAbilities` 或[`执行计算（Executions）`](#concepts-ge-ec)读取它，以便知道应该对哪个 `Attribute` 施加伤害。可受伤害组件少于最大数量甚至为零的 `Pawns` 也没有问题。`AttributeSet` 中有某个 `Attribute`，并不意味着你必须使用它。未使用的 `Attributes` 只占用微不足道的内存。

如果你的每个子组件都需要很多 `Attributes`，或者子组件的数量可能没有上限，或者子组件可以拆下来给其他玩家使用（例如武器），又或者出于其他任何原因这种方法不适合你，我建议不要再用 `Attributes`，而是直接在组件上存储普通的浮点数。参见[物品属性](#concepts-as-design-itemattributes)。

<a name="concepts-as-design-addremoveruntime"></a>
##### 4.4.2.2 在运行时添加和移除 AttributeSet
`AttributeSets` 可以在运行时添加到 `ASC` 或从中移除，但移除 `AttributeSets` 可能有危险。例如，如果某个 `AttributeSet` 在客户端上先于服务器被移除，而此时有一个 `Attribute` 值的变化被复制到客户端，这个 `Attribute` 就会找不到它的 `AttributeSet`，导致游戏崩溃。

武器加入背包时：
```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().AddUnique(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```

武器从背包移除时：
```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().Remove(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```
<a name="concepts-as-design-itemattributes"></a>
##### 4.4.2.3 物品属性（武器弹药）
有几种方式可以实现带 `Attributes` 的可装备物品（武器弹药、护甲耐久度等）。这些方式都把数值直接存储在物品上。对于在生命周期内可能被多个玩家装备的物品，这是必要的。

> 1. 在物品上使用普通浮点数（**推荐**）
> 1. 在物品上使用独立的 `AttributeSet`
> 1. 在物品上使用独立的 `ASC`

<a name="concepts-as-design-itemattributes-plainfloats"></a>
###### 4.4.2.3.1 在物品上使用普通浮点数
不使用 `Attributes`，而是在物品类实例上存储普通的浮点数值。Fortnite 和 [GASShooter](https://github.com/tranek/GASShooter) 都是这样处理枪械弹药的。对于一把枪，直接把最大弹匣容量、弹匣内当前弹药、备用弹药等作为被复制的浮点数（`COND_OwnerOnly`）存储在枪的实例上。如果多把武器共享备用弹药，就把备用弹药作为一个 `Attribute` 移到角色身上，放在一个共享的弹药 `AttributeSet` 中（换弹能力可以使用 `Cost GE`，从备用弹药中扣除并补充到枪上以浮点数存储的弹匣弹药中）。由于弹匣当前弹药没有使用 `Attributes`，你需要重写 `UGameplayAbility` 中的一些函数，以便针对枪上的浮点数检查并扣除消耗。授予能力时，把枪设置为 [`GameplayAbilitySpec`](https://github.com/tranek/GASDocumentation#concepts-ga-spec) 中的 `SourceObject`，这样在能力内部就能访问到授予该能力的那把枪。

为了防止在自动射击期间，枪把弹药数量复制回来、覆盖掉本地的弹药数量，可以在 `PreReplication()` 中，当玩家拥有 `IsFiring` `GameplayTag` 时禁用复制。本质上，你是在这里自己实现本地预测。

```c++
void AGSWeapon::PreReplication(IRepChangedPropertyTracker& ChangedPropertyTracker)
{
	Super::PreReplication(ChangedPropertyTracker);

	DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, PrimaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
	DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, SecondaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
}
```

优点：
1. 避免了使用 `AttributeSets` 的限制（见下文）

限制：
1. 无法使用现有的 `GameplayEffect` 工作流（例如用 `Cost GEs` 扣除弹药）
1. 需要额外工作，重写 `UGameplayAbility` 上的关键函数，以便针对枪上的浮点数检查并扣除弹药消耗

<a name="concepts-as-design-itemattributes-attributeset"></a>
###### 4.4.2.3.2 在物品上使用 `AttributeSet`
在物品上使用一个独立的 `AttributeSet`，并在物品加入玩家背包时[将其添加到玩家的 `ASC` 上](#concepts-as-design-addremoveruntime)，这种方式可行，但有一些重大限制。我在早期版本的 [GASShooter](https://github.com/tranek/GASShooter) 中就用这种方式实现过武器弹药。武器把最大弹匣容量、弹匣内当前弹药、备用弹药等 `Attributes` 存放在武器类上的一个 `AttributeSet` 中。如果多把武器共享备用弹药，就把备用弹药移到角色身上的一个共享弹药 `AttributeSet` 中。当武器在服务器上加入玩家背包时，武器会把自己的 `AttributeSet` 添加到玩家的 `ASC::SpawnedAttributes` 中，然后服务器再把它复制到客户端。如果武器从背包中移除，它会把自己的 `AttributeSet` 从 `ASC::SpawnedAttributes` 中移除。

当 `AttributeSet` 不在 `OwnerActor` 上（比如在武器上）时，一开始你会在 `AttributeSet` 中遇到一些编译错误。解决方法是：在 `BeginPlay()` 而不是构造函数中创建 `AttributeSet`，并在武器上实现 `IAbilitySystemInterface`（在武器加入玩家背包时设置指向 `ASC` 的指针）。

```c++
void AGSWeapon::BeginPlay()
{
	if (!AttributeSet)
	{
		AttributeSet = NewObject<UGSWeaponAttributeSet>(this);
	}
	//...
}
```

你可以查看这个[旧版本的 GASShooter](https://github.com/tranek/GASShooter/tree/df5949d0dd992bd3d76d4a728f370f2e2c827735)，了解它的实际用法。

优点：
1. 可以使用现有的 `GameplayAbility` 和 `GameplayEffect` 工作流（例如用 `Cost GEs` 扣除弹药）
1. 对于数量很少的物品，配置起来很简单

限制：
1. 你必须为每种武器类型创建一个新的 `AttributeSet` 类。在功能上，一个 `ASC` 中每个类只能有一个 `AttributeSet` 实例，因为对 `Attribute` 的修改会在 `ASC` 的 `SpawnedAttributes` 数组中查找其 `AttributeSet` 类的第一个实例。同一 `AttributeSet` 类的其他实例会被忽略。
1. 出于上一条原因（每个 `AttributeSet` 类只能有一个 `AttributeSet` 实例），玩家背包中每种类型的武器只能有一把。
1. 移除 `AttributeSet` 很危险。在 GASShooter 中，如果玩家被自己的火箭炸死，玩家会立即从背包中移除火箭发射器（包括从 `ASC` 中移除它的 `AttributeSet`）。当服务器把火箭发射器弹药 `Attribute` 的变化复制下来时，客户端的 `ASC` 上已经不存在这个 `AttributeSet` 了，于是游戏崩溃。

<a name="concepts-as-design-itemattributes-asc"></a>
###### 4.4.2.3.3 在物品上使用 `ASC`
在每个物品上放一个完整的 `AbilitySystemComponent` 是一种极端的做法。我自己没这样做过，也没在实际项目中见过。要让它跑起来需要大量的工程工作。

> 能否让多个 AbilitySystemComponent 拥有相同的 Owner、但拥有不同的 Avatar（例如分别挂在 Pawn 和武器/物品/投射物上，Owner 都设为 PlayerState）？
> 
> 我看到的第一个问题是如何在拥有者 Actor 上实现 IGameplayTagAssetInterface 和 IAbilitySystemInterface。前者也许可行：把所有 ASC 的标签聚合起来即可（但要小心——HasAllMatchingGameplayTags 可能只有在跨 ASC 聚合后才能满足。仅仅把调用转发给每个 ASC，再把结果做 OR 运算是不够的）。但后者更加棘手：哪一个 ASC 才是权威的那个？如果有人想施加一个 GE，应该由哪一个来接收？也许你能把这些问题解决掉，但这一方面会是整个问题中最难的部分：一个 Owner 下面会挂着多个 ASC。
> 
> 不过，单就 Pawn 和武器上各有一个独立的 ASC 而言，这本身可以说得通。例如，用来区分描述武器的标签和描述拥有者 Pawn 的标签。也许让授予武器的标签同时“作用于”拥有者、而其他东西不共享，是合理的（例如属性和 GE 相互独立，但拥有者会像我上面说的那样聚合其拥有的标签）。我相信这是可以实现的。但让多个 ASC 拥有同一个 Owner，可能会变得很棘手。

*摘自 Epic 的 Dave Ratti 对[社区问题 #6](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)的回答*

优点：
1. 可以使用现有的 `GameplayAbility` 和 `GameplayEffect` 工作流（例如用 `Cost GEs` 扣除弹药）
1. 可以复用 `AttributeSet` 类（每把武器的 ASC 上各一个）

限制：
1. 工程成本未知
1. 这到底能不能实现？

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-as-attributes"></a>
#### 4.4.3 定义属性
**`Attributes` 只能在 C++ 中定义**，位置是 `AttributeSet` 的头文件。建议在每个 `AttributeSet` 头文件的顶部加入下面这组宏，它会为你的 `Attributes` 自动生成 getter 和 setter 函数。

```c++
// Uses macros from AttributeSet.h
// 使用 AttributeSet.h 中的宏
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```

一个被复制的生命值属性可以这样定义：

```c++
UPROPERTY(BlueprintReadOnly, Category = "Health", ReplicatedUsing = OnRep_Health)
FGameplayAttributeData Health;
ATTRIBUTE_ACCESSORS(UGDAttributeSetBase, Health)
```

同时在头文件中定义 `OnRep` 函数：
```c++
UFUNCTION()
virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);
```

`AttributeSet` 的 .cpp 文件应当用预测系统所使用的 `GAMEPLAYATTRIBUTE_REPNOTIFY` 宏来实现 `OnRep` 函数：
```c++
void UGDAttributeSetBase::OnRep_Health(const FGameplayAttributeData& OldHealth)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGDAttributeSetBase, Health, OldHealth);
}
```

最后，需要把该 `Attribute` 添加到 `GetLifetimeReplicatedProps` 中：
```c++
void UGDAttributeSetBase::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME_CONDITION_NOTIFY(UGDAttributeSetBase, Health, COND_None, REPNOTIFY_Always);
}
```

`REPNOTIFY_Always` 表示：即使本地值（由于预测）已经等于服务器复制下来的值，也要触发 `OnRep` 函数。默认情况下，如果本地值与服务器复制下来的值相同，就不会触发 `OnRep` 函数。

如果该 `Attribute` 不需要复制（比如 `Meta Attribute`），则可以跳过 `OnRep` 和 `GetLifetimeReplicatedProps` 这两步。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-as-init"></a>
#### 4.4.4 初始化属性
初始化 `Attributes`（把它们的 `BaseValue`，进而 `CurrentValue`，设置为某个初始值）的方法有多种。Epic 推荐使用瞬时 `GameplayEffect`，示例项目用的也是这种方法。

关于如何制作用于初始化 `Attributes` 的瞬时 `GameplayEffect`，请参阅示例项目中的 `GE_HeroAttributes` 蓝图。该 `GameplayEffect` 的施加是在 C++ 中进行的。

如果你在定义 `Attributes` 时使用了 `ATTRIBUTE_ACCESSORS` 宏，`AttributeSet` 上会为每个 `Attribute` 自动生成一个初始化函数，你可以在 C++ 中随时调用。

```c++
// InitHealth(float InitialValue) is an automatically generated function for an Attribute 'Health' defined with the `ATTRIBUTE_ACCESSORS` macro
// InitHealth(float InitialValue) 是为用 `ATTRIBUTE_ACCESSORS` 宏定义的属性 'Health' 自动生成的函数
AttributeSet->InitHealth(100.0f);
```

更多初始化 `Attributes` 的方式，请参阅 `AttributeSet.h`。

**注意：** 在 4.24 之前，`FAttributeSetInitterDiscreteLevels` 无法与 `FGameplayAttributeData` 一起使用。它是在 `Attributes` 还是原始浮点数的时代编写的，会报错说 `FGameplayAttributeData` 不是 `Plain Old Data`（`POD`）。该问题已在 4.24 中修复：<https://issues.unrealengine.com/issue/UE-76557>。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-as-preattributechange"></a>
#### 4.4.5 PreAttributeChange()
`PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)` 是 `AttributeSet` 中的主要函数之一，用于在 `Attribute` 的 `CurrentValue` 发生变化**之前**做出响应。它是通过引用参数 `NewValue` 对即将应用到 `CurrentValue` 的变化进行钳制的理想位置。

例如，示例项目是这样钳制移动速度修改器的：
```c++
if (Attribute == GetMoveSpeedAttribute())
{
	// Cannot slow less than 150 units/s and cannot boost more than 1000 units/s
	// 减速后不能低于 150 单位/秒，加速后不能高于 1000 单位/秒
	NewValue = FMath::Clamp<float>(NewValue, 150, 1000);
}
```
`GetMoveSpeedAttribute()` 函数是由我们添加到 `AttributeSet.h` 中的那组宏生成的（[定义属性](#concepts-as-attributes)）。

任何对 `Attributes` 的修改都会触发它，无论是使用 `Attribute` 的 setter（由 `AttributeSet.h` 中的那组宏定义（[定义属性](#concepts-as-attributes)））还是使用 [`GameplayEffects`](#concepts-ge)。

**注意：** 在这里进行的任何钳制都不会永久改变 `ASC` 上的修改器，只会改变查询修改器时返回的值。这意味着，任何根据所有修改器重新计算 `CurrentValue` 的地方，比如 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 和 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc)，都需要再实现一次钳制。

**注意：** Epic 在 `PreAttributeChange()` 的注释中说明，不要把它用于游戏玩法事件，而应主要用于钳制。在 `Attribute` 变化时触发游戏玩法事件，推荐的位置是 `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`（[响应属性变化](#concepts-a-changes)）。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-as-postgameplayeffectexecute"></a>
#### 4.4.6 PostGameplayEffectExecute()
`PostGameplayEffectExecute(const FGameplayEffectModCallbackData & Data)` 只会在瞬时 [`GameplayEffect`](#concepts-ge) 修改了 `Attribute` 的 `BaseValue` 之后触发。当 `Attributes` 因 `GameplayEffect` 而变化时，这里是对它们做进一步处理的合适位置。

例如，在示例项目中，我们在这里把最终的伤害 `Meta Attribute` 从生命值 `Attribute` 中扣除。如果有护盾 `Attribute`，我们会先从护盾中扣除伤害，再把剩余部分从生命值中扣除。示例项目还在这里播放受击反应动画、显示漂浮的伤害数字，并把经验和金币赏金分配给击杀者。按照设计，伤害 `Meta Attribute` 总是通过瞬时 `GameplayEffect` 传入，而不会通过 `Attribute` 的 setter。

其他只会被瞬时 `GameplayEffects` 修改 `BaseValue` 的 `Attributes`，比如法力值和耐力值，也可以在这里钳制到与之对应的最大值 `Attributes`。

**注意：** 调用 `PostGameplayEffectExecute()` 时，对 `Attribute` 的修改已经发生，但还没有复制回客户端，所以在这里钳制数值不会导致客户端收到两次网络更新。客户端只会收到钳制之后的更新。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-as-onattributeaggregatorcreated"></a>
#### 4.4.7 OnAttributeAggregatorCreated()
`OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator)` 会在为本属性集中的某个 `Attribute` 创建 `Aggregator`（聚合器）时触发。它允许对 [`FAggregatorEvaluateMetaData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FAggregatorEvaluateMetaData/index.html) 进行自定义设置。`Aggregator` 在根据施加到 `Attribute` 上的所有[`修改器（Modifiers）`](#concepts-ge-mods)计算其 `CurrentValue` 时，会使用 `AggregatorEvaluateMetaData`。默认情况下，`Aggregator` 只用 `AggregatorEvaluateMetaData` 来判断哪些 `Modifiers` 符合条件，例如 `MostNegativeMod_AllPositiveMods`：它允许所有正向的 `Modifiers` 生效，但负向的 `Modifiers` 只允许最负的那一个生效。Paragon 就用它来实现：无论玩家身上同时有多少个减速效果，只有减速最多的那一个生效，而所有正向的移动速度增益都会生效。不符合条件的 `Modifiers` 仍然存在于 `ASC` 上，只是不会被聚合到最终的 `CurrentValue` 中。条件变化后，它们之后可能会变得符合条件，例如最负的那个 `Modifier` 到期后，次负的 `Modifier`（如果存在）就会符合条件。

以“只允许最负的 `Modifier` 和所有正向 `Modifiers`”为例，使用 AggregatorEvaluateMetaData 的方法如下：

```c++
virtual void OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const override;
```

```c++
void UGSAttributeSetBase::OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const
{
	Super::OnAttributeAggregatorCreated(Attribute, NewAggregator);

	if (!NewAggregator)
	{
		return;
	}

	if (Attribute == GetMoveSpeedAttribute())
	{
		NewAggregator->EvaluationMetaData = &FAggregatorEvaluateMetaDataLibrary::MostNegativeMod_AllPositiveMods;
	}
}
```

你自定义的用于筛选条件的 `AggregatorEvaluateMetaData`，应当作为静态变量添加到 `FAggregatorEvaluateMetaDataLibrary` 中。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge"></a>
### 4.5 GameplayEffect

<a name="concepts-ge-definition"></a>
#### 4.5.1 GameplayEffect 的定义
[`GameplayEffects`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html)（`GE`）是能力用来修改自身及他人的 [`Attributes`](#concepts-a) 和 [`GameplayTags`](#concepts-gt) 的载体。它们可以造成即时的 `Attribute` 变化（如伤害或治疗），也可以施加长期的状态增益/减益（如移动速度提升或眩晕）。`UGameplayEffect` 类被设计为一个**纯数据**类，用来定义单个游戏效果，不应向 `GameplayEffects` 中添加任何额外逻辑。通常，策划会创建大量 `UGameplayEffect` 的蓝图子类。

`GameplayEffects` 通过[`修改器（Modifiers）`](#concepts-ge-mods)和[`执行计算（Executions，即 GameplayEffectExecutionCalculation）`](#concepts-ge-ec)来修改 `Attributes`。

`GameplayEffects` 有三种持续时间类型：`Instant`（瞬时）、`Duration`（持续）和 `Infinite`（无限）。

此外，`GameplayEffects` 还可以添加/执行 [`GameplayCues`](#concepts-gc)。`Instant` `GameplayEffect` 会对 `GameplayCue` `GameplayTags` 调用 `Execute`，而 `Duration` 或 `Infinite` `GameplayEffect` 会对 `GameplayCue` `GameplayTags` 调用 `Add` 和 `Remove`。

| 持续时间类型  | GameplayCue 事件  | 适用场景                                                                                                                                                                       |
| ------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Instant`     | Execute           | 用于对 `Attribute` 的 `BaseValue` 做即时的永久修改。不会添加 `GameplayTags`，连一帧都不会。                                                                                   |
| `Duration`    | Add & Remove      | 用于临时修改 `Attribute` 的 `CurrentValue`，以及添加在 `GameplayEffect` 到期或被手动移除时随之移除的 `GameplayTags`。持续时间在 `UGameplayEffect` 类/蓝图中指定。          |
| `Infinite`    | Add & Remove      | 用于临时修改 `Attribute` 的 `CurrentValue`，以及添加在 `GameplayEffect` 被移除时随之移除的 `GameplayTags`。它们永远不会自行到期，必须由能力或 `ASC` 手动移除。            |

`Duration` 和 `Infinite` `GameplayEffects` 可以选择施加周期效果（`Periodic Effects`），按照其 `Period`（周期）的定义，每隔 `X` 秒应用一次它的 `Modifiers` 和 `Executions`。在修改 `Attribute` 的 `BaseValue` 和执行（`Executing`）`GameplayCues` 方面，`Periodic Effects` 被当作 `Instant` `GameplayEffects` 处理。它们非常适合持续伤害（DOT）类效果。**注意：** `Periodic Effects` 无法被[预测](#concepts-p)。

`Duration` 和 `Infinite` `GameplayEffects` 在施加后，如果其 `Ongoing Tag Requirements`（持续标签要求）不满足/满足（参见 [GameplayEffect 的标签](#concepts-ge-tags)），可以被临时关闭/开启。关闭一个 `GameplayEffect` 会移除其 `Modifiers` 的效果和它添加的 `GameplayTags`，但不会移除该 `GameplayEffect` 本身。重新开启 `GameplayEffect` 会重新应用它的 `Modifiers` 和 `GameplayTags`。

如果你需要手动重新计算 `Duration` 或 `Infinite` `GameplayEffect` 的 `Modifiers`（比如你有一个 `MMC` 使用了并非来自 `Attributes` 的数据），可以调用 `UAbilitySystemComponent::ActiveGameplayEffects.SetActiveGameplayEffectLevel(FActiveGameplayEffectHandle ActiveHandle, int32 NewLevel)`，并传入它当前的等级（通过 `UAbilitySystemComponent::ActiveGameplayEffects.GetActiveGameplayEffect(ActiveHandle).Spec.GetLevel()` 获取）。基于后备（backing）`Attributes` 的 `Modifiers` 会在这些后备 `Attributes` 更新时自动更新。`SetActiveGameplayEffectLevel()` 中用于更新 `Modifiers` 的关键函数是：

```C++
MarkItemDirty(Effect);
Effect.Spec.CalculateModifierMagnitudes();
// Private function otherwise we'd call these three functions without needing to set the level to what it already is
// 这是私有函数，否则我们直接调用这三个函数就行，不必把等级重新设置成它当前的值
UpdateAllAggregatorModMagnitudes(Effect);
```

`GameplayEffects` 通常不会被实例化。当能力或 `ASC` 想要施加一个 `GameplayEffect` 时，会从该 `GameplayEffect` 的 `ClassDefaultObject`（CDO）创建一个 [`GameplayEffectSpec`](#concepts-ge-spec)。成功施加的 `GameplayEffectSpecs` 随后会被放进一个名为 `FActiveGameplayEffect` 的新结构体中，`ASC` 用一个名为 `ActiveGameplayEffects` 的特殊容器结构体来跟踪它们。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-applying"></a>
#### 4.5.2 施加 GameplayEffect
`GameplayEffects` 可以通过 [`GameplayAbilities`](#concepts-ga) 上的函数和 `ASC` 上的函数以多种方式施加，这些函数通常形如 `ApplyGameplayEffectTo`。这些不同的函数本质上都是便捷函数，最终都会在 `Target`（目标）上调用 `UAbilitySystemComponent::ApplyGameplayEffectSpecToSelf()`。

要在 `GameplayAbility` 之外（例如从投射物中）施加 `GameplayEffects`，需要获取 `Target` 的 `ASC`，并使用它的某个 `ApplyGameplayEffectToSelf` 函数。

你可以绑定 `ASC` 的委托，监听任何 `Duration` 或 `Infinite` `GameplayEffects` 被施加到该 `ASC` 上的时机：
```c++
AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(this, &APACharacterBase::OnActiveGameplayEffectAddedCallback);
```
回调函数：
```c++
virtual void OnActiveGameplayEffectAddedCallback(UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);
```

无论复制模式如何，服务器总会调用这个函数。自主代理（autonomous proxy）只会在 `Full` 和 `Mixed` 复制模式下，针对被复制的 `GameplayEffects` 调用它。模拟代理（simulated proxy）只会在 `Full` [复制模式](#concepts-asc-rm)下调用它。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-removing"></a>
#### 4.5.3 移除 GameplayEffect
`GameplayEffects` 可以通过 [`GameplayAbilities`](#concepts-ga) 上的函数和 `ASC` 上的函数以多种方式移除，这些函数通常形如 `RemoveActiveGameplayEffect`。这些不同的函数本质上都是便捷函数，最终都会在 `Target` 上调用 `FActiveGameplayEffectsContainer::RemoveActiveEffects()`。

要在 `GameplayAbility` 之外移除 `GameplayEffects`，需要获取 `Target` 的 `ASC`，并使用它的某个 `RemoveActiveGameplayEffect` 函数。

你可以绑定 `ASC` 的委托，监听任何 `Duration` 或 `Infinite` `GameplayEffects` 从该 `ASC` 上被移除的时机：
```c++
AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate().AddUObject(this, &APACharacterBase::OnRemoveGameplayEffectCallback);
```
回调函数：
```c++
virtual void OnRemoveGameplayEffectCallback(const FActiveGameplayEffect& EffectRemoved);
```

无论复制模式如何，服务器总会调用这个函数。自主代理只会在 `Full` 和 `Mixed` 复制模式下，针对被复制的 `GameplayEffects` 调用它。模拟代理只会在 `Full` [复制模式](#concepts-asc-rm)下调用它。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-mods"></a>
#### 4.5.4 GameplayEffect 修改器（Modifier）
`Modifiers`（修改器）用来修改 `Attribute`，并且是[预测性地](#concepts-p)修改 `Attribute` 的唯一方式。一个 `GameplayEffect` 可以有零个或多个 `Modifiers`。每个 `Modifier` 只负责通过一种指定的运算修改一个 `Attribute`。

| 运算       | 说明                                                                                     |
| ---------- | ---------------------------------------------------------------------------------------- |
| `Add`      | 把结果加到 `Modifier` 指定的 `Attribute` 上。使用负值即可实现减法。                      |
| `Multiply` | 用结果乘以 `Modifier` 指定的 `Attribute`。                                               |
| `Divide`   | 用 `Modifier` 指定的 `Attribute` 除以结果。                                              |
| `Override` | 用结果覆盖 `Modifier` 指定的 `Attribute`。                                               |

`Attribute` 的 `CurrentValue` 是它所有 `Modifiers` 叠加到 `BaseValue` 上的聚合结果。`Modifiers` 的聚合公式在 `GameplayEffectAggregator.cpp` 的 `FAggregatorModChannel::EvaluateWithBase` 中定义如下：
```c++
((InlineBaseValue + Additive) * Multiplicitive) / Division
```

所有 `Override` `Modifiers` 都会覆盖最终值，以最后施加的 `Modifier` 为准。

**注意：** 对于基于百分比的修改，务必使用 `Multiply` 运算，以确保它在加法之后进行。

**注意：** [预测](#concepts-p)在处理百分比修改时会有问题。

`Modifiers` 有四种类型：Scalable Float（可缩放浮点数）、Attribute Based（基于属性）、Custom Calculation Class（自定义计算类）和 Set By Caller（由调用者设置）。它们都会生成一个浮点数值，然后根据 `Modifier` 的运算类型，用这个值去修改 `Modifier` 指定的 `Attribute`。

| `Modifier` 类型            | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Scalable Float`           | `FScalableFloats` 是一种结构体，可以指向一张以变量为行、以等级为列的数据表（Data Table）。Scalable Float 会自动读取指定表格行在能力当前等级下的值（如果在 [`GameplayEffectSpec`](#concepts-ge-spec) 上重写了等级，则读取那个等级的值）。这个值还可以再乘以一个系数。如果没有指定数据表/行，它会把值当作 1，这样就可以用系数为所有等级写死一个固定值。 ![ScalableFloat](https://github.com/tranek/GASDocumentation/raw/master/Images/scalablefloats.png)                                                                                  |
| `Attribute Based`          | `Attribute Based` `Modifiers` 读取 `Source`（创建 `GameplayEffectSpec` 的一方）或 `Target`（接收 `GameplayEffectSpec` 的一方）身上某个后备 `Attribute` 的 `CurrentValue` 或 `BaseValue`，再用一个系数以及系数前、系数后的加数做进一步修正。`Snapshotting`（快照）表示后备 `Attribute` 在创建 `GameplayEffectSpec` 时被捕获；不使用快照则表示 `Attribute` 在施加 `GameplayEffectSpec` 时才被捕获。                                                                                                                                    |
| `Custom Calculation Class` | `Custom Calculation Class` 为复杂的 `Modifiers` 提供了最大的灵活性。这种 `Modifier` 接受一个 [`ModifierMagnitudeCalculation`](#concepts-ge-mmc) 类，并且可以用一个系数以及系数前、系数后的加数对计算出的浮点数值做进一步修正。                                                                                                                                                                                                                                                                                                 |
| `Set By Caller`            | `SetByCaller` `Modifiers` 的值是在运行时、在 `GameplayEffect` 之外，由能力或创建 `GameplayEffectSpec` 的一方设置到 `GameplayEffectSpec` 上的。例如，如果你想根据玩家按住按键为能力蓄力的时长来决定伤害，就可以使用 `SetByCaller`。`SetByCallers` 本质上就是存放在 `GameplayEffectSpec` 上的 `TMap<FGameplayTag, float>`。`Modifier` 只是告诉 `Aggregator`（聚合器）去查找与指定 `GameplayTag` 关联的 `SetByCaller` 值。`Modifiers` 使用的 `SetByCallers` 只能用 `GameplayTag` 版本，`FName` 版本在这里是禁用的。如果 `Modifier` 被设置为 `SetByCaller`，但 `GameplayEffectSpec` 上不存在带有对应 `GameplayTag` 的 `SetByCaller`，游戏会抛出运行时错误并返回 0。对于 `Divide` 运算，这可能会引发问题。关于 `SetByCallers` 的更多用法，参见 [`SetByCallers`](#concepts-ge-spec-setbycaller)。 |

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-mods-multiplydivide"></a>
##### 4.5.4.1 乘法与除法修改器
默认情况下，所有 `Multiply` 和 `Divide` `Modifiers` 会先被加在一起，然后再与 `Attribute` 的 `BaseValue` 相乘或相除。

```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Additive = SumMods(Mods[EGameplayModOp::Additive], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Additive), Parameters);
	float Multiplicitive = SumMods(Mods[EGameplayModOp::Multiplicitive], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Multiplicitive), Parameters);
	float Division = SumMods(Mods[EGameplayModOp::Division], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Division), Parameters);
	...
	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
	...
}
```

```c++
float FAggregatorModChannel::SumMods(const TArray<FAggregatorMod>& InMods, float Bias, const FAggregatorEvaluateParameters& Parameters)
{
	float Sum = Bias;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Sum += (Mod.EvaluatedMagnitude - Bias);
		}
	}

	return Sum;
}
```
*摘自 `GameplayEffectAggregator.cpp`*

在这个公式中，`Multiply` 和 `Divide` `Modifiers` 的 `Bias`（偏置）值都是 `1`（`Addition` 的 `Bias` 是 `0`）。所以计算过程大致如下：

```
1 + (Mod1.Magnitude - 1) + (Mod2.Magnitude - 1) + ...
```

这个公式会导致一些出人意料的结果。首先，它在与 `BaseValue` 相乘或相除之前，先把所有修改器加在一起。大多数人会以为它是把这些修改器相乘或相除。例如，如果有两个值为 `1.5` 的 `Multiply` 修改器，大多数人会以为 `BaseValue` 会乘以 `1.5 x 1.5 = 2.25`。但实际上，它会把两个 `1.5` 相加，让 `BaseValue` 乘以 `2`（`提升 50% + 再提升 50% = 提升 100%`）。这是为了符合 `GameplayPrediction.h` 中的例子：基础速度 `500`，加一个 `10%` 的速度增益后是 `550`；再加一个 `10%` 的速度增益，就是 `600`。

其次，这个公式在设计时以 Paragon 为考量，因此对可以使用哪些值有一些没有写进文档的规则。

`Multiply` 和 `Divide` 的“乘数相加”公式的规则：
* `(小于 1 的值不超过一个) AND (任意数量位于 [1, 2) 区间的值)`
* `OR (一个 >= 2 的值)`

公式中的 `Bias` 基本上就是减去 `[1, 2)` 区间内数值的整数位。第一个 `Modifier` 的 `Bias` 会从初始的 `Sum` 值（循环开始前被设为 `Bias`）中减去，这就是为什么任何单个值都能正确工作，以及为什么一个 `< 1` 的值可以与 `[1, 2)` 区间内的数值配合使用。

一些 `Multiply` 的例子：  
乘数：`0.5`  
`1 + (0.5 - 1) = 0.5`，正确

乘数：`0.5, 0.5`  
`1 + (0.5 - 1) + (0.5 - 1) = 0`，错误，期望值应为 `1`？对于乘数相加来说，多个小于 `1` 的值没有意义。Paragon 在设计上只会对 `Multiply` `Modifiers` 使用[负向最大的那个值](#cae-nonstackingge)，因此最多只会有一个小于 `1` 的值乘到 `BaseValue` 上。

乘数：`1.1, 0.5`  
`1 + (0.5 - 1) + (1.1 - 1) = 0.6`，正确

乘数：`5, 5`  
`1 + (5 - 1) + (5 - 1) = 9`，错误，期望值应为 `10`。结果总是 `各 Modifier 之和 - Modifier 的数量 + 1`。

很多游戏会希望 `Multiply` 和 `Divide` `Modifiers` 先相互乘除，再应用到 `BaseValue` 上。要实现这一点，你需要**修改引擎代码**中的 `FAggregatorModChannel::EvaluateWithBase()`。

```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Multiplicitive = MultiplyMods(Mods[EGameplayModOp::Multiplicitive], Parameters);
	float Division = MultiplyMods(Mods[EGameplayModOp::Division], Parameters);
	...

	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
}
```

```c++
float FAggregatorModChannel::MultiplyMods(const TArray<FAggregatorMod>& InMods, const FAggregatorEvaluateParameters& Parameters)
{
	float Multiplier = 1.0f;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Multiplier *= Mod.EvaluatedMagnitude;
		}
	}

	return Multiplier;
}
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-mods-gameplaytags"></a>
##### 4.5.4.2 修改器上的 GameplayTag

每个[修改器](#concepts-ge-mods)都可以设置 `SourceTags` 和 `TargetTags`。它们的作用与 `GameplayEffect` 的 [`Application Tag requirements`](#concepts-ge-tags)（施加标签要求）相同，因此这些标签只在效果被施加时才会被考虑。也就是说，对于一个周期性的无限效果，它们只在效果首次施加时被考虑，而*不会*在每次周期执行时被考虑。

`Attribute Based` 修改器还可以设置 `SourceTagFilter` 和 `TargetTagFilter`。在确定作为 `Attribute Based` 修改器来源的那个属性的幅度时，这些过滤器用来排除作用于该属性的某些修改器。其来源或目标没有包含过滤器中全部标签的修改器会被排除。

具体来说：源 ASC 和目标 ASC 的标签会被 `GameplayEffects` 捕获。源 ASC 的标签在创建 `GameplayEffectSpec` 时捕获，目标 ASC 的标签在执行效果时捕获。在判断无限或持续效果的某个修改器是否“符合条件”（qualifies）而被应用时（即它的聚合器是否符合条件），如果设置了这些过滤器，就会把捕获的标签与过滤器进行比较。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-stacking"></a>
#### 4.5.5 GameplayEffect 的堆叠
默认情况下，每次施加 `GameplayEffects` 都会创建新的 `GameplayEffectSpec` 实例，这些新实例既不知道也不关心之前已经存在的 `GameplayEffectSpec` 实例。`GameplayEffects` 可以设置为可堆叠（stack）：此时不会添加新的 `GameplayEffectSpec` 实例，而是修改当前已存在的 `GameplayEffectSpec` 的堆叠层数（stack count）。堆叠只适用于 `Duration` 和 `Infinite` `GameplayEffects`。

堆叠有两种类型：按来源聚合（Aggregate by Source）和按目标聚合（Aggregate by Target）。

| 堆叠类型            | 说明                                                                                     |
| ------------------- | ---------------------------------------------------------------------------------------- |
| Aggregate by Source | 目标身上每个来源 `ASC` 各有一份独立的堆叠实例。每个来源都可以施加 X 层。                 |
| Aggregate by Target | 无论来源是谁，目标身上只有一份堆叠实例。每个来源都可以施加堆叠，直至达到共享的堆叠上限。 |

堆叠还有关于到期、持续时间刷新和周期重置的策略。在 `GameplayEffect` 蓝图中，把鼠标悬停在这些选项上会显示很有帮助的提示信息。

示例项目包含一个自定义蓝图节点，用来监听 `GameplayEffect` 堆叠层数的变化。HUD UMG 控件用它来更新玩家拥有的被动护甲层数。这个 `AsyncTask` 会一直存活，直到手动调用 `EndTask()`，我们在 UMG 控件的 `Destruct` 事件中调用它。请参阅 `AsyncTaskEffectStackChanged.h/cpp`。

![监听 GameplayEffect 堆叠层数变化的蓝图节点](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-ga"></a>
#### 4.5.6 授予的能力
`GameplayEffects` 可以向 `ASCs` 授予新的 [`GameplayAbilities`](#concepts-ga)。只有 `Duration` 和 `Infinite` `GameplayEffects` 才能授予能力。

一个常见的用例是强制另一个玩家做某件事，比如通过击退或拉拽来移动他们。你可以对他们施加一个 `GameplayEffect`，授予他们一个会自动激活的能力（关于如何在授予时自动激活能力，参见[被动能力](#concepts-ga-activating-passive)），由这个能力对他们执行所需的动作。

策划可以选择 `GameplayEffect` 授予哪些能力、以什么等级授予、将它们绑定到哪个[输入](#concepts-ga-input)，以及被授予能力的移除策略。

| 移除策略                   | 说明                                                                                                                       |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Cancel Ability Immediately | 当授予该能力的 `GameplayEffect` 从目标身上移除时，立即取消并移除被授予的能力。                                             |
| Remove Ability on End      | 允许被授予的能力执行完毕，然后再从目标身上移除。                                                                           |
| Do Nothing                 | 授予能力的 `GameplayEffect` 从目标身上移除，不会影响被授予的能力。目标会永久拥有该能力，直到之后被手动移除。               |

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-tags"></a>
#### 4.5.7 GameplayEffect 的标签
`GameplayEffects` 带有多个 [`GameplayTagContainers`](#concepts-gt)。策划会为每个类别编辑 `Added`（添加）和 `Removed`（移除）这两个 `GameplayTagContainers`，编译后结果会显示在 `Combined`（合并）`GameplayTagContainer` 中。`Added` 标签是该 `GameplayEffect` 新增的、其父类原本没有的标签；`Removed` 标签是父类拥有、但该子类没有的标签。

| 类别                              | 说明                                                                                                                                                                                                                           |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Gameplay Effect Asset Tags        | `GameplayEffect` 自身拥有的标签。它们本身不起任何作用，只用于描述该 `GameplayEffect`。                                                                                                                                       |
| Granted Tags                      | 存在于 `GameplayEffect` 上、同时也会授予被施加该 `GameplayEffect` 的 `ASC` 的标签。`GameplayEffect` 被移除时，这些标签也会从 `ASC` 上移除。仅适用于 `Duration` 和 `Infinite` `GameplayEffects`。                           |
| Ongoing Tag Requirements          | 施加之后，这些标签决定 `GameplayEffect` 处于开启还是关闭状态。`GameplayEffect` 可以处于关闭状态但仍然是已施加的。如果 `GameplayEffect` 因不满足持续标签要求而被关闭，之后又满足了要求，`GameplayEffect` 会重新开启并重新应用它的修改器。仅适用于 `Duration` 和 `Infinite` `GameplayEffects`。 |
| Application Tag Requirements      | 目标身上的标签，决定 `GameplayEffect` 能否施加到该目标上。如果不满足这些要求，`GameplayEffect` 就不会被施加。                                                                                                                |
| Remove Gameplay Effects with Tags | 当本 `GameplayEffect` 成功施加时，目标身上那些在其 `Asset Tags` 或 `Granted Tags` 中包含任一上述标签的 `GameplayEffects` 会被移除。                                                                                         |

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-immunity"></a>
#### 4.5.8 免疫
`GameplayEffects` 可以基于 [`GameplayTags`](#concepts-gt) 授予免疫，从而阻止其他 `GameplayEffects` 的施加。虽然通过 `Application Tag Requirements` 等其他方式也能有效地实现免疫，但使用这套系统可以获得一个在 `GameplayEffects` 因免疫被阻止时触发的委托：`UAbilitySystemComponent::OnImmunityBlockGameplayEffectDelegate`。

`GrantedApplicationImmunityTags` 会检查来源 `ASC`（如果有来源能力，也包括该能力 `AbilityTags` 中的标签）是否拥有任一指定的标签。借助它，可以根据某些角色或来源的标签，让目标对它们发出的所有 `GameplayEffects` 免疫。

`Granted Application Immunity Query` 会检查传入的 `GameplayEffectSpec` 是否匹配任一查询，以决定阻止还是允许其施加。

在 `GameplayEffect` 蓝图中，把鼠标悬停在这些查询上会显示很有帮助的提示信息。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-spec"></a>
#### 4.5.9 GameplayEffectSpec
[`GameplayEffectSpec`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectSpec/index.html)（`GESpec`）可以看作 `GameplayEffects` 的实例化对象。它持有对所代表的 `GameplayEffect` 类的引用、创建时的等级以及创建者。`GameplayEffects` 应当由策划在运行前创建，而 `GameplayEffectSpec` 则可以在施加前于运行时自由地创建和修改。施加 `GameplayEffect` 时，会先从该 `GameplayEffect` 创建一个 `GameplayEffectSpec`，真正施加到目标上的其实是这个 `GameplayEffectSpec`。

`GameplayEffectSpecs` 通过 `UAbilitySystemComponent::MakeOutgoingSpec()`（`BlueprintCallable`）从 `GameplayEffects` 创建。`GameplayEffectSpecs` 不必立即施加。常见的做法是把一个 `GameplayEffectSpec` 传给能力生成的投射物，让投射物稍后在命中目标时施加它。`GameplayEffectSpecs` 成功施加后，会返回一个名为 `FActiveGameplayEffect` 的新结构体。

`GameplayEffectSpec` 中值得关注的内容：
* 创建该 `GameplayEffect` 所用的 `GameplayEffect` 类。
* 该 `GameplayEffectSpec` 的等级。通常与创建该 `GameplayEffectSpec` 的能力的等级相同，但也可以不同。
* 该 `GameplayEffectSpec` 的持续时间。默认为 `GameplayEffect` 的持续时间，但也可以不同。
* 对于周期效果，该 `GameplayEffectSpec` 的周期。默认为 `GameplayEffect` 的周期，但也可以不同。
* 该 `GameplayEffectSpec` 当前的堆叠层数。堆叠上限定义在 `GameplayEffect` 上。
* [`GameplayEffectContextHandle`](#concepts-ge-context)，告诉我们是谁创建了这个 `GameplayEffectSpec`。
* 由于快照而在创建 `GameplayEffectSpec` 时捕获的 `Attributes`。
* `DynamicGrantedTags`：除了 `GameplayEffect` 授予的 `GameplayTags` 之外，该 `GameplayEffectSpec` 额外授予目标的标签。
* `DynamicAssetTags`：除了 `GameplayEffect` 拥有的 `AssetTags` 之外，该 `GameplayEffectSpec` 额外拥有的标签。
* `SetByCaller` `TMaps`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-spec-setbycaller"></a>
##### 4.5.9.1 SetByCaller
`SetByCallers` 允许 `GameplayEffectSpec` 携带与某个 `GameplayTag` 或 `FName` 关联的浮点数值。它们分别存放在 `GameplayEffectSpec` 上对应的 `TMaps` 中：`TMap<FGameplayTag, float>` 和 `TMap<FName, float>`。它们既可以用作 `GameplayEffect` 上的 `Modifiers`，也可以作为传递浮点数的通用手段。常见的做法是通过 `SetByCallers`，把能力内部生成的数值数据传给 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 或 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc)。

| `SetByCaller` 用途 | 说明                                                                                                                                                                                                                                       |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Modifiers`        | 必须预先在 `GameplayEffect` 类中定义，只能使用 `GameplayTag` 版本。如果 `GameplayEffect` 类中定义了某个 `SetByCaller`，但 `GameplayEffectSpec` 上没有对应的标签与浮点值对，施加 `GameplayEffectSpec` 时游戏会产生运行时错误并返回 0。这对于 `Divide` 运算来说是一个潜在问题。参见 [`Modifiers`](#concepts-ge-mods)。 |
| 其他场合           | 无需在任何地方预先定义。读取 `GameplayEffectSpec` 上不存在的 `SetByCaller` 时，可以返回一个由开发者定义的默认值，并可选择是否输出警告。                                                                                                  |

在蓝图中为 `SetByCaller` 赋值时，使用你需要的那个版本（`GameplayTag` 或 `FName`）对应的蓝图节点：

![为 SetByCaller 赋值](https://github.com/tranek/GASDocumentation/raw/master/Images/setbycaller.png)

要在蓝图中读取 `SetByCaller` 的值，你需要在自己的蓝图函数库中编写自定义节点。

在 C++ 中为 `SetByCaller` 赋值时，使用你需要的那个版本（`GameplayTag` 或 `FName`）的函数：

```c++
void FGameplayEffectSpec::SetSetByCallerMagnitude(FName DataName, float Magnitude);
```
```c++
void FGameplayEffectSpec::SetSetByCallerMagnitude(FGameplayTag DataTag, float Magnitude);
```

在 C++ 中读取 `SetByCaller` 的值时，使用你需要的那个版本（`GameplayTag` 或 `FName`）的函数：

```c++
float GetSetByCallerMagnitude(FName DataName, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```
```c++
float GetSetByCallerMagnitude(FGameplayTag DataTag, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```

我推荐使用 `GameplayTag` 版本而不是 `FName` 版本，这样可以避免在蓝图中出现拼写错误。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-context"></a>
#### 4.5.10 GameplayEffectContext
[`GameplayEffectContext`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectContext/index.html) 结构体保存着 `GameplayEffectSpec` 的发起者（instigator）和 [`TargetData`](#concepts-targeting-data) 等信息。它也很适合被子类化，用来在 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc) / [`GameplayEffectExecutionCalculations`](#concepts-ge-ec)、[`AttributeSets`](#concepts-as) 和 [`GameplayCues`](#concepts-gc) 等位置之间传递任意数据。

子类化 `GameplayEffectContext` 的步骤：

1. 子类化 `FGameplayEffectContext`
1. 重写 `FGameplayEffectContext::GetScriptStruct()`
1. 重写 `FGameplayEffectContext::Duplicate()`
1. 如果新数据需要复制，重写 `FGameplayEffectContext::NetSerialize()`
1. 像父结构体 `FGameplayEffectContext` 那样，为你的子类实现 `TStructOpsTypeTraits`
1. 在你的 [`AbilitySystemGlobals`](#concepts-asg) 类中重写 `AllocGameplayEffectContext()`，使其返回你的子类的新对象

[GASShooter](https://github.com/tranek/GASShooter) 使用了一个子类化的 `GameplayEffectContext` 来添加 `TargetData`，以便在 `GameplayCues` 中访问，这主要是为霰弹枪准备的，因为它可以同时击中多个敌人。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-mmc"></a>
#### 4.5.11 修改器幅度计算（MMC）
[`ModifierMagnitudeCalculations`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayModMagnitudeCalculation/index.html)（`ModMagCalc` 或 `MMC`）是一类功能强大的类，在 `GameplayEffects` 中用作[`修改器（Modifiers）`](#concepts-ge-mods)。它们的作用与 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 类似，但能力稍弱；最重要的是，它们可以被[预测](#concepts-p)。它们唯一的用途就是从 `CalculateBaseMagnitude_Implementation()` 返回一个浮点数值。你可以在蓝图和 C++ 中子类化并重写这个函数。

`MMCs` 可以用于任何持续时间类型的 `GameplayEffects`：`Instant`、`Duration`、`Infinite` 或 `Periodic`。

`MMCs` 的强大之处在于：它可以捕获 `GameplayEffect` 的 `Source` 或 `Target` 身上任意数量 `Attributes` 的值，并且可以完整访问 `GameplayEffectSpec` 来读取 `GameplayTags` 和 `SetByCallers`。`Attributes` 可以选择快照或不快照。快照的 `Attributes` 在创建 `GameplayEffectSpec` 时被捕获；不快照的 `Attributes` 在施加 `GameplayEffectSpec` 时被捕获，并且对于 `Infinite` 和 `Duration` `GameplayEffects`，会在 `Attribute` 变化时自动更新。捕获 `Attributes` 时，会根据 `ASC` 上现有的修改器重新计算它们的 `CurrentValue`。这次重新计算**不会**执行 `AbilitySet` 中的 [`PreAttributeChange()`](#concepts-as-preattributechange)，所以任何钳制都必须在这里再做一次。

| 快照 | 来源或目标 | 在 `GameplayEffectSpec` 的何时捕获 | 对 `Infinite` 或 `Duration` `GE`，`Attribute` 变化时是否自动更新 |
| ---- | ---------- | ---------------------------------- | ---------------------------------------------------------------- |
| 是   | Source     | 创建时                             | 否                                                               |
| 是   | Target     | 施加时                             | 否                                                               |
| 否   | Source     | 施加时                             | 是                                                               |
| 否   | Target     | 施加时                             | 是                                                               |

`MMC` 算出的浮点数结果，还可以在 `GameplayEffect` 的 `Modifier` 中用一个系数以及系数前、系数后的加数做进一步修正。

下面是一个 `MMC` 示例：它捕获 `Target` 的法力 `Attribute`，通过一个中毒效果减少法力，减少的量取决于 `Target` 当前有多少法力，以及 `Target` 是否拥有某个标签：
```c++
UPAMMC_PoisonMana::UPAMMC_PoisonMana()
{

	//ManaDef defined in header FGameplayEffectAttributeCaptureDefinition ManaDef;
	// ManaDef 在头文件中定义：FGameplayEffectAttributeCaptureDefinition ManaDef;
	ManaDef.AttributeToCapture = UPAAttributeSetBase::GetManaAttribute();
	ManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	ManaDef.bSnapshot = false;

	//MaxManaDef defined in header FGameplayEffectAttributeCaptureDefinition MaxManaDef;
	// MaxManaDef 在头文件中定义：FGameplayEffectAttributeCaptureDefinition MaxManaDef;
	MaxManaDef.AttributeToCapture = UPAAttributeSetBase::GetMaxManaAttribute();
	MaxManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	MaxManaDef.bSnapshot = false;

	RelevantAttributesToCapture.Add(ManaDef);
	RelevantAttributesToCapture.Add(MaxManaDef);
}

float UPAMMC_PoisonMana::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	// Gather the tags from the source and target as that can affect which buffs should be used
	// 收集来源和目标的标签，因为它们会影响应当使用哪些增益
	const FGameplayTagContainer* SourceTags = Spec.CapturedSourceTags.GetAggregatedTags();
	const FGameplayTagContainer* TargetTags = Spec.CapturedTargetTags.GetAggregatedTags();

	FAggregatorEvaluateParameters EvaluationParameters;
	EvaluationParameters.SourceTags = SourceTags;
	EvaluationParameters.TargetTags = TargetTags;

	float Mana = 0.f;
	GetCapturedAttributeMagnitude(ManaDef, Spec, EvaluationParameters, Mana);
	Mana = FMath::Max<float>(Mana, 0.0f);

	float MaxMana = 0.f;
	GetCapturedAttributeMagnitude(MaxManaDef, Spec, EvaluationParameters, MaxMana);
	MaxMana = FMath::Max<float>(MaxMana, 1.0f); // Avoid divide by zero
	// 避免除以零

	float Reduction = -20.0f;
	if (Mana / MaxMana > 0.5f)
	{
		// Double the effect if the target has more than half their mana
		// 如果目标的法力超过一半，效果翻倍
		Reduction *= 2;
	}
	
	if (TargetTags->HasTagExact(FGameplayTag::RequestGameplayTag(FName("Status.WeakToPoisonMana"))))
	{
		// Double the effect if the target is weak to PoisonMana
		// 如果目标对 PoisonMana 有弱点，效果翻倍
		Reduction *= 2;
	}
	
	return Reduction;
}
```

如果你没有在 `MMC` 的构造函数中把 `FGameplayEffectAttributeCaptureDefinition` 添加到 `RelevantAttributesToCapture`，却试图捕获 `Attributes`，就会在捕获时收到缺少 Spec 的错误。如果不需要捕获 `Attributes`，就不必向 `RelevantAttributesToCapture` 添加任何内容。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-ec"></a>
#### 4.5.12 GameplayEffect 执行计算（ExecCalc）
[`GameplayEffectExecutionCalculations`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffectExecutionCalculat-/index.html)（`ExecutionCalculation`、`Execution`（你会在插件源码中经常看到这个词）或 `ExecCalc`）是 `GameplayEffects` 修改 `ASC` 最强大的方式。与 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc) 一样，它们可以捕获 `Attributes`，并可选择是否快照。与 `MMCs` 不同的是，它们可以修改多个 `Attribute`，基本上可以做程序员想做的任何事。这种强大和灵活的代价是：它们无法被[预测](#concepts-p)，而且必须用 C++ 实现。

`ExecutionCalculations` 只能用于 `Instant` 和 `Periodic` `GameplayEffects`。名称中带有“Execute”一词的东西，通常指的就是这两类 `GameplayEffects`。

快照会在创建 `GameplayEffectSpec` 时捕获 `Attribute`，不快照则在施加 `GameplayEffectSpec` 时捕获 `Attribute`。捕获 `Attributes` 时，会根据 `ASC` 上现有的修改器重新计算它们的 `CurrentValue`。这次重新计算**不会**执行 `AbilitySet` 中的 [`PreAttributeChange()`](#concepts-as-preattributechange)，所以任何钳制都必须在这里再做一次。

| 快照 | 来源或目标 | 在 `GameplayEffectSpec` 的何时捕获 |
| ---- | ---------- | ---------------------------------- |
| 是   | Source     | 创建时                             |
| 是   | Target     | 施加时                             |
| 否   | Source     | 施加时                             |
| 否   | Target     | 施加时                             |

为了设置 `Attribute` 捕获，我们沿用 Epic 的 ActionRPG 示例项目中的模式：定义一个结构体，用来持有并定义如何捕获 `Attributes`，并在该结构体的构造函数中创建它的一份实例。每个 `ExecCalc` 都会有一个这样的结构体。**注意：** 每个结构体都需要一个唯一的名称，因为它们共享同一个命名空间。如果结构体使用相同的名称，会导致捕获 `Attributes` 时出现错误行为（通常是捕获到错误 `Attributes` 的值）。

对于 `Local Predicted`、`Server Only` 和 `Server Initiated` 的 [`GameplayAbilities`](#concepts-ga)，`ExecCalc` 只会在服务器上调用。

根据读取 `Source` 和 `Target` 身上众多属性的复杂公式来计算所受伤害，是 `ExecCalc` 最常见的例子。随附的示例项目有一个计算伤害的简单 `ExecCalc`：它从 `GameplayEffectSpec` 的 [`SetByCaller`](#concepts-ge-spec-setbycaller) 中读取伤害值，然后根据从 `Target` 身上捕获的护甲 `Attribute` 对该值进行减免。请参阅 `GDDamageExecCalculation.cpp/.h`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-ec-senddata"></a>
##### 4.5.12.1 向执行计算传递数据
除了捕获 `Attributes`，还有几种方式可以向 `ExecutionCalculation` 传递数据。

<a name="concepts-ge-ec-senddata-setbycaller"></a>
###### 4.5.12.1.1 SetByCaller
任何[设置在 `GameplayEffectSpec` 上的 `SetByCallers`](#concepts-ge-spec-setbycaller)，都可以在 `ExecutionCalculation` 中直接读取。

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
float Damage = FMath::Max<float>(Spec.GetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName("Data.Damage")), false, -1.0f), 0.0f);
```

<a name="concepts-ge-ec-senddata-backingdataattribute"></a>
###### 4.5.12.1.2 后备数据为属性的计算修改器
如果你想向 `GameplayEffect` 传入写死的数值，可以使用一个 `CalculationModifier`（计算修改器），并把某个被捕获的 `Attributes` 作为它的后备数据（backing data）。

在这张截图的例子中，我们给被捕获的 Damage `Attribute` 加上 50。你也可以把它设为 `Override`，只取这个写死的值。

![后备数据为属性的计算修改器](https://github.com/tranek/GASDocumentation/raw/master/Images/calculationmodifierbackingdataattribute.png)

`ExecutionCalculation` 在捕获该 `Attribute` 时会读入这个值。

```c++
float Damage = 0.0f;
// Capture optional damage value set on the damage GE as a CalculationModifier under the ExecutionCalculation
// 捕获在伤害 GE 上以 ExecutionCalculation 下的 CalculationModifier 形式设置的可选伤害值
ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(DamageStatics().DamageDef, EvaluationParameters, Damage);
```

<a name="concepts-ge-ec-senddata-backingdatatempvariable"></a>
###### 4.5.12.1.3 后备数据为临时变量的计算修改器
如果你想向 `GameplayEffect` 传入写死的数值，可以使用一个以 `Temporary Variable`（临时变量）为后备数据的 `CalculationModifier`，它在 C++ 中被称为 `Transient Aggregator`（瞬态聚合器）。`Temporary Variable` 与一个 `GameplayTag` 相关联。

在这张截图的例子中，我们用 `Data.Damage` `GameplayTag` 给一个 `Temporary Variable` 加上 50。

![后备数据为临时变量的计算修改器](https://github.com/tranek/GASDocumentation/raw/master/Images/calculationmodifierbackingdatatempvariable.png)

在你的 `ExecutionCalculation` 的构造函数中添加作为后备的 `Temporary Variables`：

```c++
ValidTransientAggregatorIdentifiers.AddTag(FGameplayTag::RequestGameplayTag("Data.Damage"));
```

`ExecutionCalculation` 使用与 `Attribute` 捕获函数类似的专用捕获函数来读入这个值。

```c++
float Damage = 0.0f;
ExecutionParams.AttemptCalculateTransientAggregatorMagnitude(FGameplayTag::RequestGameplayTag("Data.Damage"), EvaluationParameters, Damage);
```

<a name="concepts-ge-ec-senddata-effectcontext"></a>
###### 4.5.12.1.4 GameplayEffectContext
你可以通过 [`GameplayEffectSpec` 上的自定义 `GameplayEffectContext`](#concepts-ge-context) 向 `ExecutionCalculation` 传递数据。

在 `ExecutionCalculation` 中，可以从 `FGameplayEffectCustomExecutionParameters` 访问 `EffectContext`。

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(Spec.GetContext().Get());
```

如果需要修改 `GameplayEffectSpec` 或 `EffectContext` 上的某些内容：

```c++
FGameplayEffectSpec* MutableSpec = ExecutionParams.GetOwningSpecForPreExecuteMod();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(MutableSpec->GetContext().Get());
```

在 `ExecutionCalculation` 中修改 `GameplayEffectSpec` 时要格外小心。请参阅 `GetOwningSpecForPreExecuteMod()` 的注释。

```c++
/** Non const access. Be careful with this, especially when modifying a spec after attribute capture. */
/** 非 const 访问。使用时要小心，尤其是在属性捕获之后修改 Spec 时。 */
FGameplayEffectSpec* GetOwningSpecForPreExecuteMod() const;
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-car"></a>
#### 4.5.13 自定义施加条件（Custom Application Requirement）
与 `GameplayEffect` 上简单的 `GameplayTag` 检查相比，[`CustomApplicationRequirement`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffectCustomApplication-/index.html)（`CAR`）类让策划可以更精细地控制 `GameplayEffect` 能否被施加。在蓝图中可以通过重写 `CanApplyGameplayEffect()` 来实现，在 C++ 中则重写 `CanApplyGameplayEffect_Implementation()`。

适合使用 `CARs` 的例子：
* `Target` 需要拥有一定数量的某个 `Attribute`
* `Target` 身上某个 `GameplayEffect` 需要达到一定的堆叠层数

`CARs` 还可以做更高级的事情，例如检查 `Target` 身上是否已经有该 `GameplayEffect` 的实例，如果有，就[修改现有实例的持续时间](#concepts-ge-duration)，而不是施加一个新实例（让 `CanApplyGameplayEffect()` 返回 false）。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-cost"></a>
#### 4.5.14 消耗 GameplayEffect
[`GameplayAbilities`](#concepts-ga) 可以选配一个专门用作能力消耗的 `GameplayEffect`。消耗（Cost）是指 `ASC` 需要拥有多少某种 `Attribute` 才能激活该 `GameplayAbility`。如果 `GA` 负担不起 `Cost GE`，就无法激活。这个 `Cost GE` 应当是一个 `Instant` `GameplayEffect`，带有一个或多个从 `Attributes` 中扣减数值的 `Modifiers`。默认情况下，`Cost GEs` 设计为可被预测，建议保留这一能力，也就是说不要使用 `ExecutionCalculations`。对于复杂的消耗计算，完全可以并且鼓励使用 `MMCs`。

刚开始时，你很可能会为每个有消耗的 `GA` 都配一个独立的 `Cost GE`。更高级的做法是让多个 `GAs` 复用同一个 `Cost GE`，只需用 `GA` 特有的数据（消耗数值定义在 `GA` 上）修改从 `Cost GE` 创建出来的 `GameplayEffectSpec` 即可。**这只适用于 `Instanced`（实例化的）能力。**

复用 `Cost GE` 的两种技巧：

1. **使用 `MMC`。** 这是最简单的方法。创建一个 [`MMC`](#concepts-ge-mmc)，从 `GameplayAbility` 实例中读取消耗数值，而该实例可以从 `GameplayEffectSpec` 中获取。

```c++
float UPGMMC_HeroAbilityCost::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->Cost.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

在这个例子中，消耗数值是我在 `GameplayAbility` 子类上添加的一个 `FScalableFloat`。
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cost")
FScalableFloat Cost;
```

![使用 MMC 的 Cost GE](https://github.com/tranek/GASDocumentation/raw/master/Images/costmmc.png)

2. **重写 `UGameplayAbility::GetCostGameplayEffect()`。** 重写这个函数，并[在运行时创建一个 `GameplayEffect`](#concepts-ge-dynamic)，让它读取 `GameplayAbility` 上的消耗数值。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-cooldown"></a>
#### 4.5.15 冷却 GameplayEffect
[`GameplayAbilities`](#concepts-ga) 可以选配一个专门用作能力冷却的 `GameplayEffect`。冷却（Cooldown）决定了能力在激活后要过多久才能再次激活。如果 `GA` 仍在冷却中，就无法激活。这个 `Cooldown GE` 应当是一个不带任何 `Modifiers` 的 `Duration` `GameplayEffect`，并在其 `GrantedTags` 中为每个 `GameplayAbility`（或每个能力槽位，如果你的游戏中可互换的能力被分配到共享冷却的槽位上）配置一个唯一的 `GameplayTag`（即“`Cooldown Tag`”，冷却标签）。`GA` 实际检查的是 `Cooldown Tag` 是否存在，而不是 `Cooldown GE` 是否存在。默认情况下，`Cooldown GEs` 设计为可被预测，建议保留这一能力，也就是说不要使用 `ExecutionCalculations`。对于复杂的冷却计算，完全可以并且鼓励使用 `MMCs`。

刚开始时，你很可能会为每个有冷却的 `GA` 都配一个独立的 `Cooldown GE`。更高级的做法是让多个 `GAs` 复用同一个 `Cooldown GE`，只需用 `GA` 特有的数据（冷却时长和 `Cooldown Tag` 定义在 `GA` 上）修改从 `Cooldown GE` 创建出来的 `GameplayEffectSpec` 即可。**这只适用于 `Instanced`（实例化的）能力。**

复用 `Cooldown GE` 的两种技巧：

1. **使用 [`SetByCaller`](#concepts-ge-spec-setbycaller)。** 这是最简单的方法。把共享 `Cooldown GE` 的持续时间设置为带某个 `GameplayTag` 的 `SetByCaller`。在你的 `GameplayAbility` 子类上，定义一个表示持续时间的 float / `FScalableFloat`，一个存放唯一 `Cooldown Tag` 的 `FGameplayTagContainer`，以及一个临时的 `FGameplayTagContainer`，我们会把它作为 `Cooldown Tag` 与 `Cooldown GE` 自身标签之并集的返回指针。
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// Temp container that we will return the pointer to in GetCooldownTags().
// 临时容器，我们会在 GetCooldownTags() 中返回指向它的指针。
// This will be a union of our CooldownTags and the Cooldown GE's cooldown tags.
// 它是我们的 CooldownTags 与 Cooldown GE 自身冷却标签的并集。
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

然后重写 `UGameplayAbility::GetCooldownTags()`，返回我们的 `Cooldown Tags` 与现有 `Cooldown GE` 标签的并集。
```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset(); // MutableTags writes to the TempCooldownTags on the CDO so clear it in case the ability cooldown tags change (moved to a different slot)
	// MutableTags 写入的是 CDO 上的 TempCooldownTags，所以要先清空，以防能力的冷却标签发生变化（例如被移到了别的槽位）
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

最后，重写 `UGameplayAbility::ApplyCooldown()`，把我们的 `Cooldown Tags` 注入冷却 `GameplayEffectSpec`，并向其添加 `SetByCaller`。
```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		SpecHandle.Data.Get()->SetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName(  OurSetByCallerTag  )), CooldownDuration.GetValueAtLevel(GetAbilityLevel()));
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

在这张图中，冷却的持续时间 `Modifier` 被设置为 `SetByCaller`，其 `Data Tag` 为 `Data.Cooldown`。`Data.Cooldown` 就是上面代码中的 `OurSetByCallerTag`。

![使用 SetByCaller 的 Cooldown GE](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownsbc.png)

2. **使用 [`MMC`](#concepts-ge-mmc)。** 配置与上面相同，区别只在于不再在 `Cooldown GE` 上和 `ApplyCooldown` 中把持续时间设为 `SetByCaller`，而是把持续时间设为 `Custom Calculation Class`，并指向我们即将编写的新 `MMC`。
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// Temp container that we will return the pointer to in GetCooldownTags().
// 临时容器，我们会在 GetCooldownTags() 中返回指向它的指针。
// This will be a union of our CooldownTags and the Cooldown GE's cooldown tags.
// 它是我们的 CooldownTags 与 Cooldown GE 自身冷却标签的并集。
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

然后重写 `UGameplayAbility::GetCooldownTags()`，返回我们的 `Cooldown Tags` 与现有 `Cooldown GE` 标签的并集。
```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset(); // MutableTags writes to the TempCooldownTags on the CDO so clear it in case the ability cooldown tags change (moved to a different slot)
	// MutableTags 写入的是 CDO 上的 TempCooldownTags，所以要先清空，以防能力的冷却标签发生变化（例如被移到了别的槽位）
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

最后，重写 `UGameplayAbility::ApplyCooldown()`，把我们的 `Cooldown Tags` 注入冷却 `GameplayEffectSpec`。
```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

```c++
float UPGMMC_HeroAbilityCooldown::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->CooldownDuration.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

![使用 MMC 的 Cooldown GE](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownmmc.png)

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-cooldown-tr"></a>
##### 4.5.15.1 获取冷却 GameplayEffect 的剩余时间
```c++
bool APGPlayerState::GetCooldownRemainingForTag(FGameplayTagContainer CooldownTags, float & TimeRemaining, float & CooldownDuration)
{
	if (AbilitySystemComponent && CooldownTags.Num() > 0)
	{
		TimeRemaining = 0.f;
		CooldownDuration = 0.f;

		FGameplayEffectQuery const Query = FGameplayEffectQuery::MakeQuery_MatchAnyOwningTags(CooldownTags);
		TArray< TPair<float, float> > DurationAndTimeRemaining = AbilitySystemComponent->GetActiveEffectsTimeRemainingAndDuration(Query);
		if (DurationAndTimeRemaining.Num() > 0)
		{
			int32 BestIdx = 0;
			float LongestTime = DurationAndTimeRemaining[0].Key;
			for (int32 Idx = 1; Idx < DurationAndTimeRemaining.Num(); ++Idx)
			{
				if (DurationAndTimeRemaining[Idx].Key > LongestTime)
				{
					LongestTime = DurationAndTimeRemaining[Idx].Key;
					BestIdx = Idx;
				}
			}

			TimeRemaining = DurationAndTimeRemaining[BestIdx].Key;
			CooldownDuration = DurationAndTimeRemaining[BestIdx].Value;

			return true;
		}
	}

	return false;
}
```

**注意：** 在客户端上查询冷却剩余时间，要求客户端能够接收被复制的 `GameplayEffects`。这取决于其 `ASC` 的[复制模式](#concepts-asc-rm)。

<a name="concepts-ge-cooldown-listen"></a>
##### 4.5.15.2 监听冷却的开始与结束
要监听冷却何时开始，可以绑定 `AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf`，在 `Cooldown GE` 被施加时做出响应；也可以绑定 `AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)`，在 `Cooldown Tag` 被添加时做出响应。我推荐监听 `Cooldown GE` 的添加，因为这样你还能访问到施加它的 `GameplayEffectSpec`，并据此判断这个 `Cooldown GE` 是本地预测的那个，还是服务器用来校正的那个。

要监听冷却何时结束，可以绑定 `AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate()`，在 `Cooldown GE` 被移除时做出响应；也可以绑定 `AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)`，在 `Cooldown Tag` 被移除时做出响应。我推荐监听 `Cooldown Tag` 的移除，因为当服务器校正后的 `Cooldown GE` 到达时，它会移除我们本地预测的那个，从而触发 `OnAnyGameplayEffectRemovedDelegate()`，尽管此时我们其实仍在冷却中。而在移除预测的 `Cooldown GE`、施加服务器校正的 `Cooldown GE` 的整个过程中，`Cooldown Tag` 都不会发生变化。

**注意：** 在客户端上监听 `GameplayEffect` 的添加或移除，要求客户端能够接收被复制的 `GameplayEffects`。这取决于其 `ASC` 的[复制模式](#concepts-asc-rm)。

示例项目包含一个自定义蓝图节点，用来监听冷却的开始与结束。HUD UMG 控件用它来更新陨石能力的冷却剩余时间。这个 `AsyncTask` 会一直存活，直到手动调用 `EndTask()`，我们在 UMG 控件的 `Destruct` 事件中调用它。请参阅 [`AsyncTaskCooldownChanged.h/cpp`](Source/GASDocumentation/Private/Characters/Abilities/AsyncTaskCooldownChanged.cpp)。

![监听冷却变化的蓝图节点](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

<a name="concepts-ge-cooldown-prediction"></a>
##### 4.5.15.3 预测冷却
目前冷却其实无法真正被预测。我们可以在本地预测的 `Cooldown GE` 被施加时启动 UI 上的冷却计时器，但 `GameplayAbility` 的实际冷却取决于服务器上冷却的剩余时间。视玩家的延迟而定，本地预测的冷却可能已经结束，但服务器上的 `GameplayAbility` 仍在冷却中，这会阻止 `GameplayAbility` 立即再次激活，直到服务器上的冷却结束。

示例项目的处理方式是：本地预测的冷却开始时，把陨石能力的 UI 图标置灰；等服务器校正后的 `Cooldown GE` 到达后，再启动冷却计时器。

这在玩法上带来的后果是：对于冷却很短的能力，高延迟玩家的射速比低延迟玩家低，处于劣势。Fortnite 通过让武器使用自定义的计数逻辑、而不使用冷却 `GameplayEffects` 来避免这个问题。

实现真正可预测的冷却（本地冷却结束时玩家就能激活 `GameplayAbility`，即使服务器上仍在冷却中），是 Epic 希望在 [GAS 的未来迭代](#concepts-p-future)中某一天实现的功能。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-duration"></a>
#### 4.5.16 修改激活中 GameplayEffect 的持续时间
要修改 `Cooldown GE` 或任何 `Duration` `GameplayEffect` 的剩余时间，需要修改 `GameplayEffectSpec` 的 `Duration`，更新它的 `StartServerWorldTime`、`CachedStartServerWorldTime` 和 `StartWorldTime`，然后用 `CheckDuration()` 重新检查持续时间。在服务器上这样做，并把该 `FActiveGameplayEffect` 标记为脏（dirty），就会把这些修改复制到客户端。
**注意：** 这里用到了 `const_cast`，可能并不是 Epic 预想的修改持续时间的方式，但目前看来效果不错。

```c++
bool UPAAbilitySystemComponent::SetGameplayEffectDurationHandle(FActiveGameplayEffectHandle Handle, float NewDuration)
{
	if (!Handle.IsValid())
	{
		return false;
	}

	const FActiveGameplayEffect* ActiveGameplayEffect = GetActiveGameplayEffect(Handle);
	if (!ActiveGameplayEffect)
	{
		return false;
	}

	FActiveGameplayEffect* AGE = const_cast<FActiveGameplayEffect*>(ActiveGameplayEffect);
	if (NewDuration > 0)
	{
		AGE->Spec.Duration = NewDuration;
	}
	else
	{
		AGE->Spec.Duration = 0.01f;
	}

	AGE->StartServerWorldTime = ActiveGameplayEffects.GetServerWorldTime();
	AGE->CachedStartServerWorldTime = AGE->StartServerWorldTime;
	AGE->StartWorldTime = ActiveGameplayEffects.GetWorldTime();
	ActiveGameplayEffects.MarkItemDirty(*AGE);
	ActiveGameplayEffects.CheckDuration(Handle);

	AGE->EventSet.OnTimeChanged.Broadcast(AGE->Handle, AGE->StartWorldTime, AGE->GetDuration());
	OnGameplayEffectDurationChange(*AGE);

	return true;
}
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-dynamic"></a>
#### 4.5.17 在运行时创建动态 GameplayEffect
在运行时创建动态 `GameplayEffects` 是一个进阶主题，你应该不会经常需要这样做。

只有 `Instant` `GameplayEffects` 可以在运行时用 C++ 从零创建。`Duration` 和 `Infinite` `GameplayEffects` 无法在运行时动态创建，因为它们在复制时会去查找一个并不存在的 `GameplayEffect` 类定义。要实现类似的功能，你应该像平常一样在编辑器中制作一个作为原型（archetype）的 `GameplayEffect` 类，然后在运行时按需定制 `GameplayEffectSpec` 实例。

在运行时创建的 `Instant` `GameplayEffects` 也可以在[本地预测](#concepts-p)的 `GameplayAbility` 中调用。不过，目前还不清楚这种动态创建是否会有副作用。

##### 示例

示例项目会在角色于其 `AttributeSet` 中受到致命一击时，创建一个这样的 GE，把金币和经验值发给击杀者。

```c++
// Create a dynamic instant Gameplay Effect to give the bounties
// 创建一个动态的瞬时 GameplayEffect 来发放赏金
UGameplayEffect* GEBounty = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Bounty")));
GEBounty->DurationPolicy = EGameplayEffectDurationType::Instant;

int32 Idx = GEBounty->Modifiers.Num();
GEBounty->Modifiers.SetNum(Idx + 2);

FGameplayModifierInfo& InfoXP = GEBounty->Modifiers[Idx];
InfoXP.ModifierMagnitude = FScalableFloat(GetXPBounty());
InfoXP.ModifierOp = EGameplayModOp::Additive;
InfoXP.Attribute = UGDAttributeSetBase::GetXPAttribute();

FGameplayModifierInfo& InfoGold = GEBounty->Modifiers[Idx + 1];
InfoGold.ModifierMagnitude = FScalableFloat(GetGoldBounty());
InfoGold.ModifierOp = EGameplayModOp::Additive;
InfoGold.Attribute = UGDAttributeSetBase::GetGoldAttribute();

Source->ApplyGameplayEffectToSelf(GEBounty, 1.0f, Source->MakeEffectContext());
```

第二个例子展示了在本地预测的 `GameplayAbility` 中创建运行时 `GameplayEffect`。使用风险自负（参见代码中的注释）！

```c++
UGameplayAbilityRuntimeGE::UGameplayAbilityRuntimeGE()
{
	NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;
}

void UGameplayAbilityRuntimeGE::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
	if (HasAuthorityOrPredictionKey(ActorInfo, &ActivationInfo))
	{
		if (!CommitAbility(Handle, ActorInfo, ActivationInfo))
		{
			EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
		}

		// Create the GE at runtime.
		// 在运行时创建 GE。
		UGameplayEffect* GameplayEffect = NewObject<UGameplayEffect>(GetTransientPackage(), TEXT("RuntimeInstantGE"));
		GameplayEffect->DurationPolicy = EGameplayEffectDurationType::Instant; // Only instant works with runtime GE.
		// 运行时创建的 GE 只能是瞬时的。

		// Add a simple scalable float modifier, which overrides MyAttribute with 42.
		// 添加一个简单的可缩放浮点数修改器，用 42 覆盖 MyAttribute。
		// In real world applications, consume information passed via TriggerEventData.
		// 在实际应用中，应使用通过 TriggerEventData 传入的信息。
		const int32 Idx = GameplayEffect->Modifiers.Num();
		GameplayEffect->Modifiers.SetNum(Idx + 1);
		FGameplayModifierInfo& ModifierInfo = GameplayEffect->Modifiers[Idx];
		ModifierInfo.Attribute.SetUProperty(UMyAttributeSet::GetMyModifiedAttribute());
		ModifierInfo.ModifierMagnitude = FScalableFloat(42.f);
		ModifierInfo.ModifierOp = EGameplayModOp::Override;

		// Apply the GE.
		// 施加该 GE。

		// Create the GESpec here to avoid the behavior of ASC to create GESpecs from the GE class default object.
		// 在这里创建 GESpec，以避开 ASC 从 GE 类默认对象（CDO）创建 GESpec 的行为。
		// Since we have a dynamic GE here, this would create a GESpec with the base GameplayEffect class, so we
		// 由于这里是动态 GE，那样会用 GameplayEffect 基类创建 GESpec，
		// would lose our modifiers. Attention: It is unknown, if this "hack" done here can have drawbacks!
		// 我们的修改器就会丢失。注意：目前还不清楚这里的“hack”做法是否会带来问题！
		// The spec prevents the GE object being collected by the GarbageCollector, since the GE is a UPROPERTY on the spec.
		// 由于 GE 是 Spec 上的一个 UPROPERTY，Spec 会防止 GE 对象被垃圾回收器回收。
		FGameplayEffectSpec* GESpec = new FGameplayEffectSpec(GameplayEffect, {}, 0.f); // "new", since lifetime is managed by a shared ptr within the handle
		// 这里用 "new"，因为其生命周期由 Handle 内部的共享指针管理
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, FGameplayEffectSpecHandle(GESpec));
	}
	EndAbility(Handle, ActorInfo, ActivationInfo, false, false);
}
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ge-containers"></a>
#### 4.5.18 GameplayEffect 容器
Epic 的 [Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)实现了一个名为 `FGameplayEffectContainer` 的结构体。它不属于原生 GAS，但用来容纳 `GameplayEffects` 和 [`TargetData`](#concepts-targeting-data) 极其方便。它把一些工作自动化了，比如从 `GameplayEffects` 创建 `GameplayEffectSpecs`，以及在其 `GameplayEffectContext` 中设置默认值。在 `GameplayAbility` 中创建一个 `GameplayEffectContainer` 并把它传给生成的投射物，非常简单直接。我选择不在随附的示例项目中实现 `GameplayEffectContainers`，是为了展示在原生 GAS 中没有它们时该如何工作，但我强烈建议你研究一下，并考虑把它们加入你的项目。

要访问 `GameplayEffectContainers` 内部的 `GESpecs`（例如为其添加 `SetByCallers`），可以拆分（break）`FGameplayEffectContainer`，然后按索引从 `GESpecs` 数组中取出 `GESpec` 引用。这要求你事先知道想访问的 `GESpec` 的索引。

![在 GameplayEffectContainer 中使用 SetByCaller](https://github.com/tranek/GASDocumentation/raw/master/Images/gecontainersetbycaller.png)

`GameplayEffectContainers` 还提供了一种可选的、高效的[目标选择](#concepts-targeting-containers)方式。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga"></a>
### 4.6 GameplayAbility

<a name="concepts-ga-definition"></a>
#### 4.6.1 GameplayAbility 的定义
[`GameplayAbilities`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/UGameplayAbility/index.html)（`GA`）是 `Actor` 在游戏中可以执行的任何动作或技能。同一时间可以有多个 `GameplayAbility` 处于激活状态，例如一边冲刺一边开枪。它们可以用蓝图或 C++ 编写。

`GameplayAbilities` 的例子：
* 跳跃
* 冲刺
* 开枪
* 每隔 X 秒被动格挡一次攻击
* 使用药水
* 开门
* 采集资源
* 建造建筑

不应该用 `GameplayAbilities` 实现的东西：
* 基础的移动输入
* 部分与 UI 的交互——不要用 `GameplayAbility` 从商店购买物品。

这些不是硬性规定，只是我的建议。你的设计和实现可以有所不同。

`GameplayAbilities` 默认带有等级功能，可以用来调整对属性的修改量，或者改变 `GameplayAbility` 的功能。

根据 [`Net Execution Policy`](#concepts-ga-net)（网络执行策略）的不同，`GameplayAbilities` 会在拥有者客户端和/或服务器上运行，但不会在模拟代理上运行。`Net Execution Policy` 决定了 `GameplayAbility` 是否会在本地被[预测](#concepts-p)。它们内置了对[可选的消耗与冷却 `GameplayEffects`](#concepts-ga-commit) 的默认处理。`GameplayAbilities` 使用 [`AbilityTasks`](#concepts-at) 来处理随时间推进的行为，例如等待某个事件、等待属性变化、等待玩家选择目标，或者用 `Root Motion Source`（根运动源）移动 `Character`。**模拟客户端不会运行 `GameplayAbilities`**。取而代之的是，当服务器运行能力时，任何需要在模拟代理上播放的视觉内容（比如动画蒙太奇）会通过 `AbilityTasks` 进行复制或 RPC，而声音、粒子之类的表现性内容则通过 [`GameplayCues`](#concepts-gc) 处理。

所有 `GameplayAbilities` 都会重写 `ActivateAbility()` 函数来编写你的游戏逻辑。还可以在 `EndAbility()` 中添加额外逻辑，它会在 `GameplayAbility` 完成或被取消时执行。

一个简单 `GameplayAbility` 的流程图：
![简单 GameplayAbility 的流程图](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartsimple.png)


一个更复杂 `GameplayAbility` 的流程图：
![复杂 GameplayAbility 的流程图](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartcomplex.png)

复杂的能力可以用多个相互交互（激活、取消等）的 `GameplayAbilities` 来实现。

<a name="concepts-ga-definition-reppolicy"></a>
##### 4.6.1.1 复制策略（Replication Policy）
不要使用这个选项。它的名字有误导性，而且你并不需要它。[`GameplayAbilitySpecs`](#concepts-ga-spec) 默认就会从服务器复制到拥有者客户端。如上所述，**`GameplayAbilities` 不会在模拟代理上运行**。它们使用 `AbilityTasks` 和 `GameplayCues` 把视觉上的变化复制或 RPC 到模拟代理。Epic 的 Dave Ratti 曾表示希望[在将来移除这个选项](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)。

<a name="concepts-ga-definition-remotecancel"></a>
##### 4.6.1.2 服务器响应远程能力取消（Server Respects Remote Ability Cancellation）
这个选项往往弊大于利。它的意思是：如果客户端的 `GameplayAbility` 结束了，无论是因为被取消还是自然完成，都会强制服务器上的版本结束，不管服务器上的版本是否已经完成。后一种情况才是关键问题，尤其是对于高延迟玩家使用的本地预测 `GameplayAbilities`。通常你会希望禁用这个选项。

<a name="concepts-ga-definition-repinputdirectly"></a>
##### 4.6.1.3 直接复制输入（Replicate Input Directly）
启用这个选项后，输入的按下和松开事件总会被复制到服务器。Epic 建议不要使用它；如果你已经[把输入绑定到了 `ASC`](#concepts-ga-input)，就应该依赖现有输入相关 [`AbilityTasks`](#concepts-at) 中内置的 `Generic Replicated Events`（通用复制事件）。

Epic 的注释：
```c++
/** Direct Input state replication. These will be called if bReplicateInputDirectly is true on the ability and is generally not a good thing to use. (Instead, prefer to use Generic Replicated Events). */
/** 直接复制输入状态。如果能力上的 bReplicateInputDirectly 为 true，就会调用这些函数，但通常不建议使用。（更推荐使用 Generic Replicated Events。） */
UAbilitySystemComponent::ServerSetInputPressed()
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-input"></a>
#### 4.6.2 将输入绑定到 ASC
`ASC` 允许你直接把输入动作（input action）绑定到它上面，并在授予 `GameplayAbilities` 时把这些输入分配给它们。分配给 `GameplayAbilities` 的输入动作在被按下时，如果满足 `GameplayTag` 要求，就会自动激活对应的 `GameplayAbilities`。要使用内置的、响应输入的 `AbilityTasks`，就必须分配输入动作。

除了分配用于激活 `GameplayAbilities` 的输入动作，`ASC` 还接受通用的 `Confirm`（确认）和 `Cancel`（取消）输入。`AbilityTasks` 用这两个特殊输入来确认或取消诸如 [`Target Actors`](#concepts-targeting-actors) 之类的东西。

要把输入绑定到 `ASC`，首先必须创建一个枚举，把输入动作名称转换为一个字节值。枚举项的名称必须与项目设置中输入动作的名称完全一致，`DisplayName` 则无关紧要。

来自示例项目：
```c++
UENUM(BlueprintType)
enum class EGDAbilityInputID : uint8
{
	// 0 None
	// 0 无
	None			UMETA(DisplayName = "None"),
	// 1 Confirm
	// 1 确认
	Confirm			UMETA(DisplayName = "Confirm"),
	// 2 Cancel
	// 2 取消
	Cancel			UMETA(DisplayName = "Cancel"),
	// 3 LMB
	// 3 鼠标左键
	Ability1		UMETA(DisplayName = "Ability1"),
	// 4 RMB
	// 4 鼠标右键
	Ability2		UMETA(DisplayName = "Ability2"),
	// 5 Q
	Ability3		UMETA(DisplayName = "Ability3"),
	// 6 E
	Ability4		UMETA(DisplayName = "Ability4"),
	// 7 R
	Ability5		UMETA(DisplayName = "Ability5"),
	// 8 Sprint
	// 8 冲刺
	Sprint			UMETA(DisplayName = "Sprint"),
	// 9 Jump
	// 9 跳跃
	Jump			UMETA(DisplayName = "Jump")
};
```

如果你的 `ASC` 位于 `Character` 上，就在 `SetupPlayerInputComponent()` 中加入绑定到 `ASC` 的函数调用：
```c++
// Bind to AbilitySystemComponent
// 绑定到 AbilitySystemComponent
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));
AbilitySystemComponent->BindAbilityActivationToInputComponent(PlayerInputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), AbilityEnumAssetPath, static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

如果你的 `ASC` 位于 `PlayerState` 上，那么在 `SetupPlayerInputComponent()` 内部可能存在竞态条件（race condition）：此时 `PlayerState` 可能还没有复制到客户端。因此，我建议在 `SetupPlayerInputComponent()` 和 `OnRep_PlayerState()` 中都尝试绑定输入。仅靠 `OnRep_PlayerState()` 是不够的，因为可能出现这样一种情况：`PlayerState` 已经复制过来，而 `PlayerController` 还没有通知客户端调用负责创建 `InputComponent` 的 `ClientRestart()`，此时 `Actor` 的 `InputComponent` 可能为空。示例项目演示了在这两个位置都尝试绑定，并用一个布尔值把关，确保实际只绑定一次输入。

**注意：** 在示例项目中，枚举里的 `Confirm` 和 `Cancel` 与项目设置中的输入动作名称（`ConfirmTarget` 和 `CancelTarget`）并不一致，但我们在 `BindAbilityActivationToInputComponent()` 中提供了它们之间的映射。这两个输入比较特殊，因为我们提供了映射，所以它们不必一致，当然一致也可以。枚举中的其他所有输入都必须与项目设置中的输入动作名称一致。

对于只会由一个输入激活的 `GameplayAbilities`（它们总是位于同一个“槽位”，就像 MOBA 那样），我更喜欢在自己的 `UGameplayAbility` 子类上添加一个变量来定义它们的输入，然后在授予能力时从 `ClassDefaultObject` 中读取它。

<a name="concepts-ga-input-noactivate"></a>
##### 4.6.2.1 绑定输入但不激活能力
如果你不希望 `GameplayAbilities` 在按下输入时自动激活，但仍想把它们绑定到输入上以配合 `AbilityTasks` 使用，可以在 `UGameplayAbility` 子类上添加一个新的布尔变量 `bActivateOnInput`，默认值为 `true`，然后重写 `UAbilitySystemComponent::AbilityLocalInputPressed()`。

```c++
void UGSAbilitySystemComponent::AbilityLocalInputPressed(int32 InputID)
{
	// Consume the input if this InputID is overloaded with GenericConfirm/Cancel and the GenericConfim/Cancel callback is bound
	// 如果该 InputID 同时被用作 GenericConfirm/Cancel，并且绑定了 GenericConfirm/Cancel 回调，就消费掉这次输入
	if (IsGenericConfirmInputBound(InputID))
	{
		LocalInputConfirm();
		return;
	}

	if (IsGenericCancelInputBound(InputID))
	{
		LocalInputCancel();
		return;
	}

	// ---------------------------------------------------------

	ABILITYLIST_SCOPE_LOCK();
	for (FGameplayAbilitySpec& Spec : ActivatableAbilities.Items)
	{
		if (Spec.InputID == InputID)
		{
			if (Spec.Ability)
			{
				Spec.InputPressed = true;
				if (Spec.IsActive())
				{
					if (Spec.Ability->bReplicateInputDirectly && IsOwnerActorAuthoritative() == false)
					{
						ServerSetInputPressed(Spec.Handle);
					}

					AbilitySpecInputPressed(Spec);

					// Invoke the InputPressed event. This is not replicated here. If someone is listening, they may replicate the InputPressed event to the server.
					// 触发 InputPressed 事件。这里不会复制它。如果有人在监听，他们可能会把 InputPressed 事件复制到服务器。
					InvokeReplicatedEvent(EAbilityGenericReplicatedEvent::InputPressed, Spec.Handle, Spec.ActivationInfo.GetActivationPredictionKey());
				}
				else
				{
					UGSGameplayAbility* GA = Cast<UGSGameplayAbility>(Spec.Ability);
					if (GA && GA->bActivateOnInput)
					{
						// Ability is not active, so try to activate it
						// 能力尚未激活，所以尝试激活它
						TryActivateAbility(Spec.Handle);
					}
				}
			}
		}
	}
}
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-granting"></a>
#### 4.6.3 授予能力
向 `ASC` 授予一个 `GameplayAbility`，会把它添加到 `ASC` 的 `ActivatableAbilities` 列表中，这样只要满足 [`GameplayTag` 要求](#concepts-ga-tags)，`ASC` 就可以随时激活该 `GameplayAbility`。

我们在服务器上授予 `GameplayAbilities`，服务器随后会自动把 [`GameplayAbilitySpec`](#concepts-ga-spec) 复制到拥有者客户端。其他客户端/模拟代理不会收到 `GameplayAbilitySpec`。

示例项目在 `Character` 类上存放了一个 `TArray<TSubclassOf<UGDGameplayAbility>>`，在游戏开始时读取它并授予其中的能力：
```c++
void AGDCharacterBase::AddCharacterAbilities()
{
	// Grant abilities, but only on the server	
	// 授予能力，但只在服务器上进行
	if (Role != ROLE_Authority || !AbilitySystemComponent.IsValid() || AbilitySystemComponent->bCharacterAbilitiesGiven)
	{
		return;
	}

	for (TSubclassOf<UGDGameplayAbility>& StartupAbility : CharacterAbilities)
	{
		AbilitySystemComponent->GiveAbility(
			FGameplayAbilitySpec(StartupAbility, GetAbilityLevel(StartupAbility.GetDefaultObject()->AbilityID), static_cast<int32>(StartupAbility.GetDefaultObject()->AbilityInputID), this));
	}

	AbilitySystemComponent->bCharacterAbilitiesGiven = true;
}
```

授予这些 `GameplayAbilities` 时，我们用 `UGameplayAbility` 类、能力等级、绑定的输入，以及 `SourceObject`（即是谁把这个 `GameplayAbility` 授予了这个 `ASC`）来创建 `GameplayAbilitySpecs`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-activating"></a>
#### 4.6.4 激活能力
如果给 `GameplayAbility` 分配了输入动作，那么在按下该输入且满足其 `GameplayTag` 要求时，它会被自动激活。但这并不总是激活 `GameplayAbility` 的理想方式。`ASC` 还提供了另外四种激活 `GameplayAbilities` 的方法：通过 `GameplayTag`、通过 `GameplayAbility` 类、通过 `GameplayAbilitySpec` 句柄（handle），以及通过事件。通过事件激活 `GameplayAbility`，可以让你[随事件一起传入数据载荷（payload）](#concepts-ga-data)。

```c++
UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilitiesByTag(const FGameplayTagContainer& GameplayTagContainer, bool bAllowRemoteActivation = true);

UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilityByClass(TSubclassOf<UGameplayAbility> InAbilityToActivate, bool bAllowRemoteActivation = true);

bool TryActivateAbility(FGameplayAbilitySpecHandle AbilityToActivate, bool bAllowRemoteActivation = true);

bool TriggerAbilityFromGameplayEvent(FGameplayAbilitySpecHandle AbilityToTrigger, FGameplayAbilityActorInfo* ActorInfo, FGameplayTag Tag, const FGameplayEventData* Payload, UAbilitySystemComponent& Component);

FGameplayAbilitySpecHandle GiveAbilityAndActivateOnce(const FGameplayAbilitySpec& AbilitySpec, const FGameplayEventData* GameplayEventData);
```
要通过事件激活 `GameplayAbility`，必须在该 `GameplayAbility` 中设置好它的 `Triggers`（触发器）：指定一个 `GameplayTag`，并为 `GameplayEvent` 选择一个选项。要发送事件，使用函数 `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)`。通过事件激活 `GameplayAbility` 可以传入带数据的载荷。

`GameplayAbility` 的 `Triggers` 还允许你在某个 `GameplayTag` 被添加或移除时激活该 `GameplayAbility`。

**注意：** 在蓝图中通过事件激活 `GameplayAbility` 时，必须使用 `ActivateAbilityFromEvent` 节点。

**注意：** 当 `GameplayAbility` 应当终止时，别忘了调用 `EndAbility()`，除非你的 `GameplayAbility` 是像被动能力那样始终运行的。

**本地预测（locally predicted）**的 `GameplayAbilities` 的激活流程：
1. **拥有者客户端**调用 `TryActivateAbility()`
1. 调用 `InternalTryActivateAbility()`
1. 调用 `CanActivateAbility()`，返回以下条件是否满足：`GameplayTag` 要求是否满足、`ASC` 是否负担得起消耗、`GameplayAbility` 是否不在冷却中，以及当前是否没有其他实例处于激活状态
1. 调用 `CallServerTryActivateAbility()`，并把它生成的 `Prediction Key`（预测键）传过去
1. 调用 `CallActivateAbility()`
1. 调用 `PreActivate()`，Epic 称之为“样板式的初始化工作（boilerplate init stuff）”
1. 调用 `ActivateAbility()`，最终激活该能力

**服务器**收到 `CallServerTryActivateAbility()`
1. 调用 `ServerTryActivateAbility()`
1. 调用 `InternalServerTryActivateAbility()` 
1. 调用 `InternalTryActivateAbility()`
1. 调用 `CanActivateAbility()`，返回以下条件是否满足：`GameplayTag` 要求是否满足、`ASC` 是否负担得起消耗、`GameplayAbility` 是否不在冷却中，以及当前是否没有其他实例处于激活状态
1. 如果成功，调用 `ClientActivateAbilitySucceed()`，通知客户端更新其 `ActivationInfo`（表明激活已被服务器确认），并广播 `OnConfirmDelegate` 委托。这与输入确认（input confirmation）不是一回事。
1. 调用 `CallActivateAbility()`
1. 调用 `PreActivate()`，Epic 称之为“样板式的初始化工作（boilerplate init stuff）”
1. 调用 `ActivateAbility()`，最终激活该能力

如果服务器在任何一步激活失败，它会调用 `ClientActivateAbilityFailed()`，立即终止客户端的 `GameplayAbility`，并撤销所有预测的修改。

<a name="concepts-ga-activating-passive"></a>
##### 4.6.4.1 被动能力
要实现自动激活并持续运行的被动 `GameplayAbilities`，可以重写 `UGameplayAbility::OnAvatarSet()`（该函数会在 `GameplayAbility` 被授予且 `AvatarActor` 设置好之后自动调用），并在其中调用 `TryActivateAbility()`。

我建议在你自定义的 `UGameplayAbility` 类中添加一个 `bool`，用来指定该 `GameplayAbility` 是否应在被授予时激活。示例项目的被动护甲叠层能力就是这么做的。

被动 `GameplayAbilities` 的 [`Net Execution Policy`](#concepts-ga-net) 通常为 `Server Only`。

```c++
void UGDGameplayAbility::OnAvatarSet(const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilitySpec & Spec)
{
	Super::OnAvatarSet(ActorInfo, Spec);

	if (bActivateAbilityOnGranted)
	{
		ActorInfo->AbilitySystemComponent->TryActivateAbility(Spec.Handle, false);
	}
}
```

Epic 认为这个函数是启动被动能力、执行类似 `BeginPlay` 工作的正确位置。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-activating-failedtags"></a>
##### 4.6.4.2 激活失败标签（Activation Failed Tags）

能力带有默认逻辑，可以告诉你能力激活失败的原因。要启用它，必须配置好与默认失败情形相对应的 GameplayTag。

把这些标签（或者按你自己的命名规范命名的标签）添加到项目中：
```
+GameplayTagList=(Tag="Activation.Fail.BlockedByTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.CantAffordCost",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.IsDead",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.MissingTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.Networking",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.OnCooldown",DevComment="")
```

然后把它们添加到 [`GASDocumentation\Config\DefaultGame.ini`](https://github.com/tranek/GASDocumentation/blob/master/Config/DefaultGame.ini#L8-L13) 中：
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
ActivateFailIsDeadName=Activation.Fail.IsDead
ActivateFailCooldownName=Activation.Fail.OnCooldown
ActivateFailCostName=Activation.Fail.CantAffordCost
ActivateFailTagsBlockedName=Activation.Fail.BlockedByTags
ActivateFailTagsMissingName=Activation.Fail.MissingTags
ActivateFailNetworkingName=Activation.Fail.Networking
```

这样，每当能力激活失败时，对应的 GameplayTag 就会出现在输出日志信息中，或者显示在 `showdebug AbilitySystem` 的 HUD 上。
```
LogAbilitySystem: Display: InternalServerTryActivateAbility. Rejecting ClientActivation of Default__GA_FireGun_C. InternalTryActivateAbility failed: Activation.Fail.BlockedByTags
LogAbilitySystem: Display: ClientActivateAbilityFailed_Implementation. PredictionKey :109 Ability: Default__GA_FireGun_C
```

![showdebug AbilitySystem 中显示的激活失败标签](https://github.com/tranek/GASDocumentation/raw/master/Images/activationfailedtags.png)

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-cancelabilities"></a>
#### 4.6.5 取消能力
要在 `GameplayAbility` 内部取消它，调用 `CancelAbility()` 即可。它会调用 `EndAbility()`，并把 `WasCancelled` 参数设为 true。

要从外部取消 `GameplayAbility`，`ASC` 提供了以下几个函数：

```c++
/** Cancels the specified ability CDO. */
/** 取消指定的能力 CDO。 */
void CancelAbility(UGameplayAbility* Ability);	

/** Cancels the ability indicated by passed in spec handle. If handle is not found among reactivated abilities nothing happens. */
/** 取消传入的 Spec 句柄所指向的能力。如果在已激活的能力中找不到该句柄，则什么也不做。 */
void CancelAbilityHandle(const FGameplayAbilitySpecHandle& AbilityHandle);

/** Cancel all abilities with the specified tags. Will not cancel the Ignore instance */
/** 取消所有带有指定标签的能力。不会取消 Ignore 实例 */
void CancelAbilities(const FGameplayTagContainer* WithTags=nullptr, const FGameplayTagContainer* WithoutTags=nullptr, UGameplayAbility* Ignore=nullptr);

/** Cancels all abilities regardless of tags. Will not cancel the ignore instance */
/** 取消所有能力，不论其标签。不会取消 Ignore 实例 */
void CancelAllAbilities(UGameplayAbility* Ignore=nullptr);

/** Cancels all abilities and kills any remaining instanced abilities */
/** 取消所有能力，并销毁所有残留的能力实例 */
virtual void DestroyActiveState();
```

**注意：** 我发现，如果存在 `Non-Instanced`（非实例化）的 `GameplayAbilities`，`CancelAllAbilities` 似乎无法正常工作。它似乎在遇到 `Non-Instanced` `GameplayAbility` 时就放弃了。`CancelAbilities` 能更好地处理 `Non-Instanced` `GameplayAbilities`，示例项目用的就是它（跳跃是一个非实例化的 `GameplayAbility`）。具体效果可能因项目而异。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-definition-activeability"></a>
#### 4.6.6 获取激活中的能力
初学者经常问：“怎样获取当前激活的能力？”——也许是想给它设置变量，或者取消它。由于同一时间可以有多个 `GameplayAbility` 处于激活状态，所以并不存在唯一的“激活中的能力”。你需要在 `ASC` 的 `ActivatableAbilities` 列表（`ASC` 所拥有的、已被授予的 `GameplayAbilities`）中搜索，找到与你要找的 [`Asset` 或 `Granted` `GameplayTag`](#concepts-ga-tags) 相匹配的那个。

`UAbilitySystemComponent::GetActivatableAbilities()` 会返回一个 `TArray<FGameplayAbilitySpec>` 供你遍历。

`ASC` 还有另一个辅助函数，它接受一个 `GameplayTagContainer` 参数来辅助搜索，免得你手动遍历 `GameplayAbilitySpecs` 列表。`bOnlyAbilitiesThatSatisfyTagRequirements` 参数会让它只返回那些满足自身 `GameplayTag` 要求、此刻能够被激活的 `GameplayAbilitySpecs`。例如，你可能有两个普通攻击 `GameplayAbilities`，一个用武器，一个用拳头，根据是否装备了武器（它决定了 `GameplayTag` 要求）来激活正确的那个。更多信息请参阅 Epic 在该函数上的注释。
```c++
UAbilitySystemComponent::GetActivatableGameplayAbilitySpecsByAllMatchingTags(const FGameplayTagContainer& GameplayTagContainer, TArray < struct FGameplayAbilitySpec* >& MatchingGameplayAbilities, bool bOnlyAbilitiesThatSatisfyTagRequirements = true)
```

获取到想要的 `FGameplayAbilitySpec` 后，就可以对它调用 `IsActive()`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-instancing"></a>
#### 4.6.7 实例化策略（Instancing Policy）
`GameplayAbility` 的 `Instancing Policy`（实例化策略）决定了 `GameplayAbility` 在激活时是否以及如何被实例化。

| `Instancing Policy`     | 说明                                                                           | 适用场景示例                                                                                                                                                                                                                                         |
| ----------------------- | ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Instanced Per Actor     | 每个 `ASC` 只有一个该 `GameplayAbility` 的实例，在多次激活之间复用。           | 这很可能是你用得最多的 `Instancing Policy`。它适用于任何能力，并能在多次激活之间保留状态。需要重置的变量，由策划负责在两次激活之间手动重置。                                                                                                     |
| Instanced Per Execution | 每次激活 `GameplayAbility` 时，都会创建一个新的 `GameplayAbility` 实例。       | 这类 `GameplayAbilities` 的好处是每次激活时变量都会被重置。由于每次激活都会生成新的 `GameplayAbilities`，其性能比 `Instanced Per Actor` 差。示例项目中没有使用这种策略。                                                                         |
| Non-Instanced           | `GameplayAbility` 直接在它的 `ClassDefaultObject` 上运行，不创建任何实例。     | 三者中性能最好，但能做的事情限制也最多。`Non-Instanced` `GameplayAbilities` 无法存储状态，也就是说不能有动态变量，也不能绑定 `AbilityTask` 委托。最适合用于频繁使用的简单能力，比如 MOBA 或 RTS 中小兵的普通攻击。示例项目中的跳跃 `GameplayAbility` 就是 `Non-Instanced` 的。 |

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-net"></a>
#### 4.6.8 网络执行策略（Net Execution Policy）
`GameplayAbility` 的 `Net Execution Policy`（网络执行策略）决定了由谁运行该 `GameplayAbility`，以及按什么顺序运行。

| `Net Execution Policy` | 说明                                                                                                                                                         |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Local Only`           | `GameplayAbility` 只在拥有者客户端上运行。对于只做本地表现性修改的能力可能有用。单人游戏应当使用 `Server Only`。                                            |
| `Local Predicted`      | `Local Predicted` `GameplayAbilities` 先在拥有者客户端上激活，然后在服务器上激活。服务器上的版本会纠正客户端预测错误的部分。参见[预测](#concepts-p)。       |
| `Server Only`          | `GameplayAbility` 只在服务器上运行。被动 `GameplayAbilities` 通常是 `Server Only`。单人游戏应当使用这个。                                                    |
| `Server Initiated`     | `Server Initiated` `GameplayAbilities` 先在服务器上激活，然后在拥有者客户端上激活。我个人很少（如果有的话）使用它。                                          | 

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-tags"></a>
#### 4.6.9 能力标签
`GameplayAbilities` 自带若干内置了逻辑的 `GameplayTagContainers`。这些 `GameplayTags` 都不会被复制。

| `GameplayTag Container`     | 说明                                                                                                                                                           |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Ability Tags`              | `GameplayAbility` 自身拥有的 `GameplayTags`。它们只是用来描述该 `GameplayAbility` 的 `GameplayTags`。                                                         |
| `Cancel Abilities with Tag` | 当本 `GameplayAbility` 激活时，其他在 `Ability Tags` 中带有这些 `GameplayTags` 的 `GameplayAbilities` 会被取消。                                              |
| `Block Abilities with Tag`  | 当本 `GameplayAbility` 处于激活状态时，其他在 `Ability Tags` 中带有这些 `GameplayTags` 的 `GameplayAbilities` 会被阻止激活。                                  |
| `Activation Owned Tags`     | 当本 `GameplayAbility` 处于激活状态时，这些 `GameplayTags` 会被赋予该 `GameplayAbility` 的拥有者。记住，这些标签不会被复制。                                   |
| `Activation Required Tags`  | 只有当拥有者拥有**全部**这些 `GameplayTags` 时，本 `GameplayAbility` 才能被激活。                                                                              |
| `Activation Blocked Tags`   | 如果拥有者拥有这些 `GameplayTags` 中的**任意一个**，本 `GameplayAbility` 就无法被激活。                                                                        |
| `Source Required Tags`      | 只有当 `Source` 拥有**全部**这些 `GameplayTags` 时，本 `GameplayAbility` 才能被激活。只有当 `GameplayAbility` 由事件触发时，才会设置 `Source` 的 `GameplayTags`。 |
| `Source Blocked Tags`       | 如果 `Source` 拥有这些 `GameplayTags` 中的**任意一个**，本 `GameplayAbility` 就无法被激活。只有当 `GameplayAbility` 由事件触发时，才会设置 `Source` 的 `GameplayTags`。 |
| `Target Required Tags`      | 只有当 `Target` 拥有**全部**这些 `GameplayTags` 时，本 `GameplayAbility` 才能被激活。只有当 `GameplayAbility` 由事件触发时，才会设置 `Target` 的 `GameplayTags`。 |
| `Target Blocked Tags`       | 如果 `Target` 拥有这些 `GameplayTags` 中的**任意一个**，本 `GameplayAbility` 就无法被激活。只有当 `GameplayAbility` 由事件触发时，才会设置 `Target` 的 `GameplayTags`。 |

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-spec"></a>
#### 4.6.10 GameplayAbilitySpec
`GameplayAbility` 被授予后，`ASC` 上就会存在一个 `GameplayAbilitySpec`，它定义了这个可激活的 `GameplayAbility`：`GameplayAbility` 类、等级、输入绑定，以及必须与 `GameplayAbility` 类分开保存的运行时状态。

在服务器上授予 `GameplayAbility` 时，服务器会把 `GameplayAbilitySpec` 复制到拥有者客户端，以便客户端激活它。

激活一个 `GameplayAbilitySpec` 时，会根据其 `Instancing Policy` 创建一个 `GameplayAbility` 实例（`Non-Instanced` `GameplayAbilities` 则不创建）。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-data"></a>
#### 4.6.11 向能力传递数据
`GameplayAbilities` 的一般范式是 `激活 -> 生成数据 -> 应用 -> 结束`。但有时你需要基于已有的数据进行操作。GAS 提供了几种把外部数据传入 `GameplayAbilities` 的方式：

| 方法                                            | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 通过事件激活 `GameplayAbility`                  | 用一个带数据载荷的事件来激活 `GameplayAbility`。对于本地预测的 `GameplayAbilities`，事件的载荷会从客户端复制到服务器。对于现有变量都装不下的任意数据，可以使用两个 `Optional Object` 变量或 [`TargetData`](#concepts-targeting-data) 变量。这种方式的缺点是无法再通过输入绑定来激活该能力。要通过事件激活 `GameplayAbility`，必须在该 `GameplayAbility` 中设置好它的 `Triggers`：指定一个 `GameplayTag`，并为 `GameplayEvent` 选择一个选项。要发送事件，使用函数 `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)`。 |
| 使用 `WaitGameplayEvent` `AbilityTask`          | 使用 `WaitGameplayEvent` `AbilityTask`，让 `GameplayAbility` 在激活后监听一个带载荷数据的事件。事件载荷及其发送流程，与通过事件激活 `GameplayAbilities` 相同。这种方式的缺点是 `AbilityTask` 不会复制事件，因此只应用于 `Local Only` 和 `Server Only` 的 `GameplayAbilities`。你也可以自己编写一个会复制事件载荷的 `AbilityTask`。                                                                                                                                                      |
| 使用 `TargetData`                               | 自定义的 `TargetData` 结构体是在客户端与服务器之间传递任意数据的好方法。                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 把数据存放在 `OwnerActor` 或 `AvatarActor` 上    | 使用存放在 `OwnerActor`、`AvatarActor` 或任何你能拿到引用的对象上的被复制变量。这种方式最灵活，也适用于通过输入绑定激活的 `GameplayAbilities`。但它不能保证在使用时数据已经通过复制同步完成，你必须事先确保这一点——也就是说，如果你设置了一个被复制的变量，然后立即激活一个 `GameplayAbility`，由于可能存在丢包，接收方上两者发生的先后顺序是无法保证的。                                                                                                                                |

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-commit"></a>
#### 4.6.12 能力的消耗与冷却
`GameplayAbilities` 内置了可选的消耗和冷却功能。消耗是指 `ASC` 为了激活 `GameplayAbility` 而必须拥有的预定数量的 `Attributes`，用一个 `Instant` `GameplayEffect`（[`Cost GE`](#concepts-ge-cost)）实现。冷却是一个计时器，在它到期之前阻止 `GameplayAbility` 再次激活，用一个 `Duration` `GameplayEffect`（[`Cooldown GE`](#concepts-ge-cooldown)）实现。

在 `GameplayAbility` 调用 `UGameplayAbility::Activate()` 之前，会先调用 `UGameplayAbility::CanActivateAbility()`。该函数会检查拥有者 `ASC` 是否负担得起消耗（`UGameplayAbility::CheckCost()`），并确保 `GameplayAbility` 不在冷却中（`UGameplayAbility::CheckCooldown()`）。

`GameplayAbility` 调用 `Activate()` 之后，可以在任意时刻选择用 `UGameplayAbility::CommitAbility()` 提交（commit）消耗和冷却，该函数会调用 `UGameplayAbility::CommitCost()` 和 `UGameplayAbility::CommitCooldown()`。如果消耗和冷却不应同时提交，策划可以选择分别调用 `CommitCost()` 或 `CommitCooldown()`。提交消耗和冷却时会再调用一次 `CheckCost()` 和 `CheckCooldown()`，这是 `GameplayAbility` 因它们而失败的最后机会。在 `GameplayAbility` 激活之后，拥有者 `ASC` 的 `Attributes` 可能会发生变化，导致提交时无法满足消耗。如果提交时[预测键](#concepts-p-key)有效，提交消耗和冷却就可以被[本地预测](#concepts-p)。

实现细节请参阅 [`CostGE`](#concepts-ge-cost) 和 [`CooldownGE`](#concepts-ge-cooldown)。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-leveling"></a>
#### 4.6.13 能力升级
升级能力有两种常见的方法：

| 升级方法                                   | 说明                                                                                                                                       |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| 撤销授予后以新等级重新授予                 | 在服务器上从 `ASC` 撤销（移除）该 `GameplayAbility`，再以下一个等级重新授予。如果此时该 `GameplayAbility` 处于激活状态，它会被终止。          |
| 提升 `GameplayAbilitySpec` 的等级          | 在服务器上找到对应的 `GameplayAbilitySpec`，提升它的等级，并把它标记为脏，以便复制到拥有者客户端。如果此时 `GameplayAbility` 处于激活状态，这种方法不会终止它。 |

两种方法的主要区别在于：升级时是否希望取消处于激活状态的 `GameplayAbilities`。根据 `GameplayAbilities` 的不同，你很可能两种方法都会用到。我建议在 `UGameplayAbility` 子类中添加一个 `bool`，用来指定使用哪种方法。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-sets"></a>
#### 4.6.14 能力集（Ability Set）
`GameplayAbilitySets` 是一种便捷的 `UDataAsset` 类，用来为角色保存输入绑定和初始 `GameplayAbilities` 列表，并带有授予这些 `GameplayAbilities` 的逻辑。子类还可以加入额外的逻辑或属性。Paragon 为每个英雄配了一个 `GameplayAbilitySet`，其中包含该英雄被授予的所有 `GameplayAbilities`。

至少就我目前所见，我认为这个类没什么必要。示例项目在 `GDCharacterBase` 及其子类中实现了 `GameplayAbilitySets` 的全部功能。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-batching"></a>
#### 4.6.15 能力批处理（Ability Batching）
传统的 `Gameplay Ability` 生命周期中，客户端至少要向服务器发送两到三次 RPC：

1. `CallServerTryActivateAbility()`
1. `ServerSetReplicatedTargetData()`（可选）
1. `ServerEndAbility()`

如果一个 `GameplayAbility` 在一帧之内以一个原子分组完成上述所有动作，我们就可以优化这个流程，把这两到三次 RPC 批处理（合并）成一次 RPC。`GAS` 把这种 RPC 优化称为 `Ability Batching`（能力批处理）。使用 `Ability Batching` 的常见例子是即时命中（hitscan）枪械。即时命中枪械会在一帧之内以一个原子分组完成激活、射线检测（line trace）、把 [`TargetData`](#concepts-targeting-data) 发送到服务器以及结束能力。[GASShooter](https://github.com/tranek/GASShooter) 示例项目在它的即时命中枪械上演示了这项技术。

半自动枪械是最理想的情况，可以把 `CallServerTryActivateAbility()`、`ServerSetReplicatedTargetData()`（子弹的命中结果）和 `ServerEndAbility()` 批处理成一次 RPC，而不是三次。

全自动/点射枪械会把第一发子弹的 `CallServerTryActivateAbility()` 和 `ServerSetReplicatedTargetData()` 批处理成一次 RPC，而不是两次。之后的每一发子弹都各自发送一次 `ServerSetReplicatedTargetData()` RPC。最后，在枪停止射击时，`ServerEndAbility()` 作为一次单独的 RPC 发送。这是最差的情况，我们只在第一发子弹上省下了一次 RPC，而不是两次。这种场景也可以通过 [`Gameplay Event`](#concepts-ga-data) 激活能力来实现，让客户端把子弹的 `TargetData` 放在 `EventPayload` 中发送给服务器。后一种方法的缺点是 `TargetData` 必须在能力外部生成，而批处理方法是在能力内部生成 `TargetData`。

[`ASC`](#concepts-asc) 上的 `Ability Batching` 默认是禁用的。要启用 `Ability Batching`，重写 `ShouldDoServerAbilityRPCBatch()` 并让它返回 true：

```c++
virtual bool ShouldDoServerAbilityRPCBatch() const override { return true; }
```

启用 `Ability Batching` 后，在激活需要批处理的能力之前，必须先创建一个 `FScopedServerAbilityRPCBatcher` 结构体。这个特殊的结构体会尝试对其作用域内随后激活的所有能力进行批处理。一旦 `FScopedServerAbilityRPCBatcher` 离开作用域，之后激活的能力就不会再尝试批处理。`FScopedServerAbilityRPCBatcher` 的工作原理是：每个可被批处理的函数中都有一段特殊代码，会拦截发送 RPC 的调用，转而把消息打包进一个批处理结构体中。当 `FScopedServerAbilityRPCBatcher` 离开作用域时，它会在 `UAbilitySystemComponent::EndServerAbilityRPCBatch()` 中自动把这个批处理结构体通过 RPC 发送给服务器。服务器在 `UAbilitySystemComponent::ServerAbilityRPCBatch_Internal(FServerAbilityRPCBatch& BatchInfo)` 中接收批处理 RPC。`BatchInfo` 参数包含若干标志，表示能力是否应当结束、激活时输入是否处于按下状态，以及 `TargetData`（如果包含了的话）。要确认批处理是否正常工作，在这个函数上打断点是个好办法。或者，也可以使用控制台变量 `AbilitySystem.ServerRPCBatching.Log 1` 开启专门的能力批处理日志。

这套机制只能在 C++ 中使用，并且只能通过 `FGameplayAbilitySpecHandle` 激活能力。

```c++
bool UGSAbilitySystemComponent::BatchRPCTryActivateAbility(FGameplayAbilitySpecHandle InAbilityHandle, bool EndAbilityImmediately)
{
	bool AbilityActivated = false;
	if (InAbilityHandle.IsValid())
	{
		FScopedServerAbilityRPCBatcher GSAbilityRPCBatcher(this, InAbilityHandle);
		AbilityActivated = TryActivateAbility(InAbilityHandle, true);

		if (EndAbilityImmediately)
		{
			FGameplayAbilitySpec* AbilitySpec = FindAbilitySpecFromHandle(InAbilityHandle);
			if (AbilitySpec)
			{
				UGSGameplayAbility* GSAbility = Cast<UGSGameplayAbility>(AbilitySpec->GetPrimaryInstance());
				GSAbility->ExternalEndAbility();
			}
		}

		return AbilityActivated;
	}

	return AbilityActivated;
}
```

GASShooter 对半自动和全自动枪械复用了同一个批处理 `GameplayAbility`，这个能力从不直接调用 `EndAbility()`（结束能力的处理在能力外部，由一个仅本地运行的能力负责，它管理玩家输入，并根据当前射击模式调用批处理能力）。由于所有 RPC 都必须发生在 `FScopedServerAbilityRPCBatcher` 的作用域内，我提供了 `EndAbilityImmediately` 参数，让负责控制/管理的那个仅本地能力可以指定：这个能力是否应当把 `EndAbility()` 调用一并批处理（半自动），还是不批处理 `EndAbility()` 调用（全自动），让 `EndAbility()` 调用稍后以单独的 RPC 发生。

GASShooter 暴露了一个允许批处理能力的蓝图节点，前面提到的仅本地能力就是用它来触发批处理能力的。

![激活批处理能力](https://github.com/tranek/GASDocumentation/raw/master/Images/batchabilityactivate.png)

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-ga-netsecuritypolicy"></a>
#### 4.6.16 网络安全策略（Net Security Policy）
`GameplayAbility` 的 `NetSecurityPolicy`（网络安全策略）决定了能力应当在网络中的哪一端执行。它可以防止客户端试图执行受限的能力。

| `NetSecurityPolicy`     | 说明                                                                                         |
| ----------------------- | -------------------------------------------------------------------------------------------- |
| `ClientOrServer`        | 没有安全要求。客户端或服务器都可以自由地触发该能力的执行和终止。                             |
| `ServerOnlyExecution`   | 客户端请求执行该能力时，会被服务器忽略。客户端仍然可以请求服务器取消或结束该能力。           |
| `ServerOnlyTermination` | 客户端请求取消或结束该能力时，会被服务器忽略。客户端仍然可以请求执行该能力。                 |
| `ServerOnly`            | 由服务器同时控制该能力的执行和终止。客户端发出的任何请求都会被忽略。                         |

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-at"></a>
### 4.7 AbilityTask

<a name="concepts-at-definition"></a>
### 4.7.1 AbilityTask 的定义
`GameplayAbilities` 只在一帧内执行，单靠它本身并没有多少灵活性。要执行随时间推进的动作，或者需要响应在之后某个时刻触发的委托，我们会使用一种名为 `AbilityTasks` 的潜伏动作（latent action）。

GAS 自带了许多开箱即用的 `AbilityTasks`：
* 用 `RootMotionSource` 移动角色的任务
* 播放动画蒙太奇的任务
* 响应 `Attribute` 变化的任务
* 响应 `GameplayEffect` 变化的任务
* 响应玩家输入的任务
* 以及更多

`UAbilityTask` 的构造函数中写死了一个全局上限：整个游戏同时运行的 `AbilityTasks` 最多为 1000 个。如果你的游戏世界中可能同时存在数百个角色（比如 RTS 游戏），在设计 `GameplayAbilities` 时要记住这一点。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-at-definition"></a>
### 4.7.2 自定义 AbilityTask
你经常需要（用 C++）创建自己的自定义 `AbilityTasks`。示例项目自带两个自定义 `AbilityTasks`：
1. `PlayMontageAndWaitForEvent` 是默认的 `PlayMontageAndWait` 与 `WaitGameplayEvent` 两个 `AbilityTasks` 的组合。它允许动画蒙太奇通过 `AnimNotifies` 把游戏事件发回启动它们的 `GameplayAbility`。可以用它在动画蒙太奇的特定时刻触发动作。
1. `WaitReceiveDamage` 监听 `OwnerActor` 受到伤害。被动护甲叠层 `GameplayAbility` 会在英雄每受到一次伤害时移除一层护甲。

`AbilityTasks` 由以下部分组成：
* 一个用于创建 `AbilityTask` 新实例的静态函数
* 在 `AbilityTask` 完成其任务时广播的委托
* 一个 `Activate()` 函数，用来开始它的主要工作、绑定外部委托等
* 一个 `OnDestroy()` 函数，用于清理工作，包括解绑它绑定过的外部委托
* 它所绑定的外部委托的回调函数
* 成员变量以及各种内部辅助函数

**注意：** `AbilityTasks` 只能声明一种类型的输出委托。所有输出委托都必须是这种类型，不管它们是否用到这些参数。对于未使用的委托参数，传入默认值即可。

`AbilityTasks` 只在运行其所属 `GameplayAbility` 的客户端或服务器上运行；不过，可以通过以下方式让 `AbilityTasks` 在模拟客户端上运行：在 `AbilityTask` 构造函数中设置 `bSimulatedTask = true;`，重写 `virtual void InitSimulatedTask(UGameplayTasksComponent& InGameplayTasksComponent);`，并把需要的成员变量设置为可复制。这只在少数情况下有用，比如移动类 `AbilityTasks`：你不想复制每一次移动变化，而是希望模拟整个移动 `AbilityTask`。所有 `RootMotionSource` `AbilityTasks` 都是这么做的。可以参考 `AbilityTask_MoveToLocation.h/.cpp`。

如果在 `AbilityTask` 构造函数中设置 `bTickingTask = true;` 并重写 `virtual void TickTask(float DeltaTime);`，`AbilityTasks` 就可以 `Tick`。当你需要在多帧之间平滑地插值（lerp）数值时，这很有用。可以参考 `AbilityTask_MoveToLocation.h/.cpp`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-at-using"></a>
### 4.7.3 使用 AbilityTask
在 C++ 中创建并激活一个 `AbilityTask`（摘自 `GDGA_FireGun.cpp`）：
```c++
UGDAT_PlayMontageAndWaitForEvent* Task = UGDAT_PlayMontageAndWaitForEvent::PlayMontageAndWaitForEvent(this, NAME_None, MontageToPlay, FGameplayTagContainer(), 1.0f, NAME_None, false, 1.0f);
Task->OnBlendOut.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnCompleted.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnInterrupted.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->OnCancelled.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->EventReceived.AddDynamic(this, &UGDGA_FireGun::EventReceived);
Task->ReadyForActivation();
```

在蓝图中，我们直接使用为该 `AbilityTask` 创建的蓝图节点即可，不需要调用 `ReadyForActivation()`，它会由 `Engine/Source/Editor/GameplayTasksEditor/Private/K2Node_LatentGameplayTaskCall.cpp` 自动调用。如果你的 `AbilityTask` 类中存在 `BeginSpawningActor()` 和 `FinishSpawningActor()`，`K2Node_LatentGameplayTaskCall` 也会自动调用它们（参见 `AbilityTask_WaitTargetData`）。再强调一次，`K2Node_LatentGameplayTaskCall` 的这些“自动魔法”只对蓝图有效。在 C++ 中，我们必须手动调用 `ReadyForActivation()`、`BeginSpawningActor()` 和 `FinishSpawningActor()`。

![蓝图中的 WaitTargetData AbilityTask](https://github.com/tranek/GASDocumentation/raw/master/Images/abilitytask.png)

要手动取消一个 `AbilityTask`，只需在蓝图（其中称为 `Async Task Proxy`）或 C++ 中对 `AbilityTask` 对象调用 `EndTask()`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-at-rms"></a>
### 4.7.4 根运动源（Root Motion Source）AbilityTask
GAS 自带一些 `AbilityTasks`，使用挂接到 `CharacterMovementComponent` 上的 `Root Motion Sources`（根运动源），随时间推移移动 `Characters`，用于击退、复杂跳跃、拉拽和冲刺等。

**注意：** 预测 `RootMotionSource` `AbilityTasks` 在 4.19 及之前、以及 4.25 及之后的引擎版本中可以正常工作。在 4.20-4.24 版本中预测存在 bug；不过这些 `AbilityTasks` 在多人游戏中仍能发挥作用，只是会有轻微的网络校正，在单人游戏中则完全正常。你可以把 4.25 中的[预测修复](https://github.com/EpicGames/UnrealEngine/commit/94107438dd9f490e7b743f8e13da46927051bf33#diff-65f6196f9f28f560f95bd578e07e290c) cherry-pick 到自定义的 4.20-4.24 引擎中。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gc"></a>
### 4.8 GameplayCue

<a name="concepts-gc-definition"></a>
#### 4.8.1 GameplayCue 的定义
`GameplayCues`（`GC`）负责执行与游戏玩法无关的内容，比如音效、粒子效果、镜头抖动等。`GameplayCues` 通常会被复制（除非显式地在本地 `Executed`、`Added` 或 `Removed`）并被预测。

触发 `GameplayCues` 的方式是：通过 `ASC` 向 `GameplayCueManager` 发送一个对应的 `GameplayTag`（**必须以 `GameplayCue.` 作为父级名称**）以及一个事件类型（`Execute`、`Add` 或 `Remove`）。`GameplayCueNotify` 对象和其他实现了 `IGameplayCueInterface` 的 `Actors` 可以根据 `GameplayCue` 的 `GameplayTag`（即 `GameplayCueTag`）订阅这些事件。

**注意：** 再强调一次，`GameplayCue` 的 `GameplayTags` 必须以父级 `GameplayTag` `GameplayCue` 开头。例如，一个有效的 `GameplayCue` `GameplayTag` 可以是 `GameplayCue.A.B.C`。

`GameplayCueNotifies` 有两类：`Static` 和 `Actor`。它们响应不同的事件，可以由不同类型的 `GameplayEffects` 触发。重写对应的事件来编写你的逻辑。

| `GameplayCue` 类                                                                                                                     | 事件              | `GameplayEffect` 类型    | 说明                                                                                                                                                                                                                                                         |
| ------------------------------------------------------------------------------------------------------------------------------------ | ----------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`GameplayCueNotify_Static`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayCueNotify_Static/index.html) | `Execute`         | `Instant` 或 `Periodic`  | Static `GameplayCueNotifies` 在 `ClassDefaultObject` 上运行（即不创建实例），非常适合命中效果这类一次性效果。                                                                                                                                                |
| [`GameplayCueNotify_Actor`](https://docs.unrealengine.com/en-US/BlueprintAPI/GameplayCueNotify/index.html)                           | `Add` 或 `Remove` | `Duration` 或 `Infinite` | Actor `GameplayCueNotifies` 在被 `Added` 时会生成一个新实例。由于它们是实例化的，可以在被 `Removed` 之前持续执行动作。它们适合用于循环音效和粒子效果，这些效果会在其背后的 `Duration` 或 `Infinite` `GameplayEffect` 被移除时，或者通过手动调用移除时被移除。它们还带有一些选项，可以控制同一时间允许 `Added` 多少个，这样同一效果多次施加时，音效或粒子只会启动一次。 |

从技术上讲，`GameplayCueNotifies` 可以响应任何事件，但上面是我们通常的用法。

**注意：** 使用 `GameplayCueNotify_Actor` 时，要勾选 `Auto Destroy on Remove`（移除时自动销毁），否则之后再对该 `GameplayCueTag` 调用 `Add` 将不起作用。

当 `ASC` 使用 `Full` 以外的[复制模式](#concepts-asc-rm)时，`Add` 和 `Remove` `GC` 事件会在服务器端玩家（Listen Server）上触发两次——一次来自施加 `GE`，另一次来自发往客户端的“Minimal” `NetMultiCast`。不过，`WhileActive` 事件仍然只会触发一次。在客户端上，所有事件都只会触发一次。

示例项目包含用于眩晕和冲刺效果的 `GameplayCueNotify_Actor`，还有一个用于 FireGun 投射物命中效果的 `GameplayCueNotify_Static`。这些 `GCs` 还可以进一步优化：[在本地触发它们](#concepts-gc-local)，而不是通过 `GE` 复制。我在示例项目中选择展示面向初学者的用法。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gc-trigger"></a>
#### 4.8.2 触发 GameplayCue

在 `GameplayEffect` 内部，填入在其成功施加（未被标签或免疫阻止）时应当触发的所有 `GameplayCues` 的 `GameplayTags`。

![由 GameplayEffect 触发的 GameplayCue](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromge.png)

`UGameplayAbility` 提供了用于 `Execute`、`Add` 或 `Remove` `GameplayCues` 的蓝图节点。

![由 GameplayAbility 触发的 GameplayCue](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromga.png)

在 C++ 中，可以直接调用 `ASC` 上的函数（或者在你的 `ASC` 子类中把它们暴露给蓝图）：

```c++
/** GameplayCues can also come on their own. These take an optional effect context to pass through hit result, etc */
/** GameplayCue 也可以单独触发。这些函数接受一个可选的 EffectContext，用于传递命中结果等信息 */
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** Add a persistent gameplay cue */
/** 添加一个持续存在的 GameplayCue */
void AddGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void AddGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** Remove a persistent gameplay cue */
/** 移除一个持续存在的 GameplayCue */
void RemoveGameplayCue(const FGameplayTag GameplayCueTag);
	
/** Removes any GameplayCue added on its own, i.e. not as part of a GameplayEffect. */
/** 移除所有单独添加的 GameplayCue，即不是作为 GameplayEffect 一部分添加的那些。 */
void RemoveAllGameplayCues();
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gc-local"></a>
#### 4.8.3 本地 GameplayCue
从 `GameplayAbilities` 和 `ASC` 上暴露出来的、用于触发 `GameplayCues` 的函数默认都是会被复制的。每个 `GameplayCue` 事件都是一次多播（multicast）RPC，这可能导致大量 RPC。GAS 还强制规定，每次网络更新中同一个 `GameplayCue` 的 RPC 最多只能有两次。我们在可能的情况下使用本地 `GameplayCues` 来避免这些问题。本地 `GameplayCues` 只在单个客户端上 `Execute`、`Add` 或 `Remove`。

可以使用本地 `GameplayCues` 的场景：
* 投射物命中
* 近战碰撞命中
* 由动画蒙太奇触发的 `GameplayCues`

应当添加到你的 `ASC` 子类中的本地 `GameplayCue` 函数：

```c++
UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);
```

```c++
void UPAAbilitySystemComponent::ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Executed, GameplayCueParameters);
}

void UPAAbilitySystemComponent::AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::OnActive, GameplayCueParameters);
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::WhileActive, GameplayCueParameters);
}

void UPAAbilitySystemComponent::RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Removed, GameplayCueParameters);
}
```

如果一个 `GameplayCue` 是在本地 `Added` 的，就应当在本地 `Removed`；如果它是通过复制 `Added` 的，就应当通过复制 `Removed`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gc-parameters"></a>
#### 4.8.4 GameplayCue 参数
`GameplayCues` 会接收一个 `FGameplayCueParameters` 结构体作为参数，其中包含该 `GameplayCue` 的额外信息。如果你是通过 `GameplayAbility` 或 `ASC` 上的函数手动触发 `GameplayCue`，就必须手动填写传给 `GameplayCue` 的 `GameplayCueParameters` 结构体。如果 `GameplayCue` 是由 `GameplayEffect` 触发的，那么 `GameplayCueParameters` 结构体中的以下变量会被自动填写：

* AggregatedSourceTags
* AggregatedTargetTags
* GameplayEffectLevel
* AbilityLevel
* [EffectContext](#concepts-ge-context)
* Magnitude（如果 `GameplayEffect` 在 `GameplayCue` 标签容器上方的下拉框中选择了一个用作幅度的 `Attribute`，并且有一个影响该 `Attribute` 的对应 `Modifier`）

手动触发 `GameplayCue` 时，`GameplayCueParameters` 结构体中的 `SourceObject` 变量或许是向 `GameplayCue` 传递任意数据的好地方。

**注意：** 参数结构体中的某些变量（比如 `Instigator`）可能已经存在于 `EffectContext` 中。`EffectContext` 还可以包含一个 `FHitResult`，用来指定在世界中的哪个位置生成 `GameplayCue`。子类化 `EffectContext` 或许是向 `GameplayCues`（尤其是由 `GameplayEffect` 触发的那些）传递更多数据的好方法。

更多信息，请参阅 [`UAbilitySystemGlobals`](#concepts-asg) 中用于填充 `GameplayCueParameters` 结构体的 3 个函数。它们是虚函数，所以你可以重写它们来自动填充更多信息。

```c++
/** Initialize GameplayCue Parameters */
/** 初始化 GameplayCue 参数 */
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectSpecForRPC &Spec);
virtual void InitGameplayCueParameters_GESpec(FGameplayCueParameters& CueParameters, const FGameplayEffectSpec &Spec);
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectContextHandle& EffectContext);
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gc-manager"></a>
#### 4.8.5 GameplayCue 管理器（GameplayCueManager）
默认情况下，`GameplayCueManager` 会扫描整个游戏目录来查找 `GameplayCueNotifies`，并在开始游戏时把它们加载到内存中。我们可以在 `DefaultGame.ini` 中设置 `GameplayCueManager` 扫描的路径。

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
GameplayCueNotifyPaths="/Game/GASDocumentation/Characters"
```

我们确实希望 `GameplayCueManager` 扫描并找到所有 `GameplayCueNotifies`，但并不希望它在开始游戏时异步加载其中的每一个。那样会把每个 `GameplayCueNotify` 及其引用的所有音效和粒子都加载到内存中，不管它们在关卡中是否会用到。在 Paragon 这样的大型游戏中，这可能意味着内存中有数百 MB 不需要的资产，并在启动时造成卡顿甚至游戏冻结。

除了在启动时异步加载所有 `GameplayCue`，另一种选择是只在游戏中触发 `GameplayCues` 时才异步加载它们。这样可以避免异步加载所有 `GameplayCue` 带来的不必要的内存占用和潜在的游戏卡死，代价是某个 `GameplayCue` 在游戏中首次触发时，其效果可能会有延迟。在 SSD 上这种延迟基本不存在，我没有在 HDD 上测试过。如果在 UE 编辑器中使用这个选项，首次加载 GameplayCue 时，如果编辑器需要编译粒子系统，可能会有轻微的卡顿或冻结。在打包版本中这不是问题，因为粒子系统已经编译好了。

首先，我们必须子类化 `UGameplayCueManager`，并在 `DefaultGame.ini` 中告诉 `AbilitySystemGlobals` 类使用我们的 `UGameplayCueManager` 子类。

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
GlobalGameplayCueManagerClass="/Script/ParagonAssets.PBGameplayCueManager"
```

在我们的 `UGameplayCueManager` 子类中，重写 `ShouldAsyncLoadRuntimeObjectLibraries()`。

```c++
virtual bool ShouldAsyncLoadRuntimeObjectLibraries() const override
{
	return false;
}
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gc-prevention"></a>
#### 4.8.6 阻止 GameplayCue 触发
有时我们不希望 `GameplayCues` 触发。例如，当我们格挡了一次攻击时，可能不想播放伤害 `GameplayEffect` 上附带的命中效果，或者想改为播放一个自定义的效果。我们可以在 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 中调用 `OutExecutionOutput.MarkGameplayCuesHandledManually()`，然后手动把我们的 `GameplayCue` 事件发送给 `Target` 或 `Source` 的 `ASC`。

如果你永远不希望某个 `ASC` 上触发任何 `GameplayCues`，可以设置 `AbilitySystemComponent->bSuppressGameplayCues = true;`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gc-batching"></a>
#### 4.8.7 GameplayCue 批处理
每触发一个 `GameplayCue`，就是一次不可靠的（unreliable）NetMulticast RPC。在同时触发多个 `GCs` 的情况下，有几种优化方法可以把它们压缩成一次 RPC，或者通过发送更少的数据来节省带宽。

<a name="concepts-gc-batching-manualrpc"></a>
##### 4.8.7.1 手动 RPC
假设你有一把一次发射八颗弹丸的霰弹枪，那就是八个射线和命中 `GameplayCues`。[GASShooter](https://github.com/tranek/GASShooter) 采用了偷懒的方法：把所有射线信息作为 [`TargetData`](#concepts-targeting-data) 塞进 [`EffectContext`](#concepts-ge-ec) 中，从而把它们合并成一次 RPC。虽然这把 RPC 从八次减少到了一次，但这一次 RPC 仍然要在网络上发送大量数据（约 500 字节）。更优化的方法是发送一个带自定义结构体的 RPC，在其中高效地编码命中位置，或者给它一个随机种子，让接收端重建/近似出命中位置。客户端随后解包这个自定义结构体，再把它还原为[在本地执行的 `GameplayCues`](#concepts-gc-local)。

其工作原理如下：
1. 声明一个 `FScopedGameplayCueSendContext`。它会在离开作用域之前抑制 `UGameplayCueManager::FlushPendingCues()`，也就是说，所有 `GameplayCues` 都会排队，直到 `FScopedGameplayCueSendContext` 离开作用域。
1. 重写 `UGameplayCueManager::FlushPendingCues()`，根据某个自定义的 `GameplayTag`，把可以一起批处理的 `GameplayCues` 合并到你的自定义结构体中，并通过 RPC 发送给客户端。
1. 客户端接收自定义结构体，并把它解包为在本地执行的 `GameplayCues`。

如果你的 `GameplayCues` 需要一些 `GameplayCueParameters` 无法提供的特定参数，而你又不想把它们加到 `EffectContext` 中（例如伤害数字、暴击标识、护盾破碎标识、致命一击标识等），也可以使用这种方法。

https://forums.unrealengine.com/development-discussion/c-gameplay-programming/1711546-fscopedgameplaycuesendcontext-gameplaycuemanager

<a name="concepts-gc-batching-gcsonge"></a>
##### 4.8.7.2 一个 GE 上挂多个 GC
一个 `GameplayEffect` 上的所有 `GameplayCues` 本来就是在一次 RPC 中发送的。默认情况下，无论 `ASC` 的 `Replication Mode` 是什么，`UGameplayCueManager::InvokeGameplayCueAddedAndWhileActive_FromSpec()` 都会在不可靠的 NetMulticast 中发送整个 `GameplayEffectSpec`（但会转换为 `FGameplayEffectSpecForRPC`）。视 `GameplayEffectSpec` 中的内容而定，这可能会占用大量带宽。我们可以通过设置控制台变量 `AbilitySystem.AlwaysConvertGESpecToGCParams 1` 来进行优化。它会把 `GameplayEffectSpecs` 转换为 `FGameplayCueParameter` 结构体，然后通过 RPC 发送这些结构体，而不是整个 `FGameplayEffectSpecForRPC`。这可能会节省带宽，但携带的信息也会更少，具体取决于 `GESpec` 如何被转换为 `GameplayCueParameters`，以及你的 `GCs` 需要知道哪些信息。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gc-events"></a>
#### 4.8.8 GameplayCue 事件
`GameplayCues` 响应特定的 `EGameplayCueEvents`：

| `EGameplayCueEvent` | 说明                                                                                                                                                                                                                                         |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OnActive`          | 在 `GameplayCue` 被激活（添加）时调用。                                                                                                                                                                                                      |
| `WhileActive`       | 在 `GameplayCue` 处于激活状态时调用，即使它并不是刚刚被施加的（例如中途加入游戏等情况）。这不是 `Tick`！和 `OnActive` 一样，它会在 `GameplayCueNotify_Actor` 被添加或变得相关（relevant）时调用一次。如果你需要 `Tick()`，直接用 `GameplayCueNotify_Actor` 的 `Tick()` 即可，毕竟它就是个 `AActor`。 |
| `Removed`           | 在 `GameplayCue` 被移除时调用。响应该事件的蓝图 `GameplayCue` 函数是 `OnRemove`。                                                                                                                                                            |
| `Executed`          | 在 `GameplayCue` 被执行时调用：瞬时效果或周期性的 `Tick()`。响应该事件的蓝图 `GameplayCue` 函数是 `OnExecute`。                                                                                                                             |

对于 `GameplayCue` 中那些在其开始时发生、且中途加入的玩家错过也无妨的内容，使用 `OnActive`。对于 `GameplayCue` 中那些希望中途加入的玩家也能看到的持续效果，使用 `WhileActive`。例如，MOBA 中有一个防御塔爆炸的 `GameplayCue`，你会把最初的爆炸粒子系统和爆炸音效放在 `OnActive` 中，而把残留的持续燃烧粒子或音效放在 `WhileActive` 中。在这种情况下，让中途加入的玩家重新播放 `OnActive` 里的初始爆炸是没有意义的，但你会希望他们能看到爆炸发生后地面上那些来自 `WhileActive` 的持续循环火焰效果。`OnRemove` 应当清理 `OnActive` 和 `WhileActive` 中添加的所有内容。每当有 Actor 进入 `GameplayCueNotify_Actor` 的相关性（relevancy）范围时，都会调用 `WhileActive`；每当有 Actor 离开 `GameplayCueNotify_Actor` 的相关性范围时，都会调用 `OnRemove`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-gc-reliability"></a>
#### 4.8.9 GameplayCue 的可靠性

总的来说，`GameplayCues` 应当被视为不可靠的，因此不适合用于任何直接影响游戏玩法的内容。

**被执行（Executed）的 `GameplayCues`：** 这类 `GameplayCues` 通过不可靠的多播施加，始终是不可靠的。

**由 `GameplayEffects` 施加的 `GameplayCues`：**
* 自主代理可以可靠地收到 `OnActive`、`WhileActive` 和 `OnRemove`  
`FActiveGameplayEffectsContainer::NetDeltaSerialize()` 会调用 `UAbilitySystemComponent::HandleDeferredGameplayCues()` 来调用 `OnActive` 和 `WhileActive`。`FActiveGameplayEffectsContainer::RemoveActiveGameplayEffectGrantedTagsAndModifiers()` 负责调用 `OnRemoved`。
* 模拟代理可以可靠地收到 `WhileActive` 和 `OnRemove`  
`UAbilitySystemComponent::MinimalReplicationGameplayCues` 的复制会调用 `WhileActive` 和 `OnRemove`。`OnActive` 事件由一次不可靠的多播调用。

**不通过 `GameplayEffect` 施加的 `GameplayCues`：**
* 自主代理可以可靠地收到 `OnRemove`  
`OnActive` 和 `WhileActive` 事件由一次不可靠的多播调用。
* 模拟代理可以可靠地收到 `WhileActive` 和 `OnRemove`  
`UAbilitySystemComponent::MinimalReplicationGameplayCues` 的复制会调用 `WhileActive` 和 `OnRemove`。`OnActive` 事件由一次不可靠的多播调用。

如果你需要 `GameplayCue` 中的某些内容是“可靠的”，就通过 `GameplayEffect` 来施加它，并在 `WhileActive` 中添加特效、在 `OnRemove` 中移除特效。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-asg"></a>
### 4.9 能力系统全局对象（AbilitySystemGlobals）
[`AbilitySystemGlobals`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemGlobals/index.html) 类保存着 GAS 的全局信息。其中大多数变量都可以在 `DefaultGame.ini` 中设置。一般来说你不需要与这个类打交道，但应该知道它的存在。如果你需要子类化 [`GameplayCueManager`](#concepts-gc-manager) 或 [`GameplayEffectContext`](#concepts-ge-context) 之类的东西，就必须通过 `AbilitySystemGlobals` 来完成。

要子类化 `AbilitySystemGlobals`，在 `DefaultGame.ini` 中设置类名：
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
AbilitySystemGlobalsClassName="/Script/ParagonAssets.PAAbilitySystemGlobals"
```

<a name="concepts-asg-initglobaldata"></a>
#### 4.9.1 InitGlobalData()
在 UE 4.24 到 5.2 之间，必须调用 `UAbilitySystemGlobals::Get().InitGlobalData()` 才能使用 [`TargetData`](#concepts-targeting-data)，否则会出现与 `ScriptStructCache` 相关的错误，客户端也会与服务器断开连接。这个函数在一个项目中只需调用一次。Fortnite 在 `UAssetManager::StartInitialLoading()` 中调用它，Paragon 则在 `UEngine::Init()` 中调用。我觉得像示例项目那样把它放在 `UAssetManager::StartInitialLoading()` 中是个不错的选择。我认为这是一段样板代码，你应该把它复制到自己的项目中，以免 `TargetData` 出问题。从 5.3 开始，它会被自动调用。

如果在使用 `AbilitySystemGlobals` 的 `GlobalAttributeSetDefaultsTableNames` 时遇到崩溃，可能需要像 Fortnite 那样，稍晚一些在 `AssetManager` 或 `GameInstance` 中调用 `UAbilitySystemGlobals::Get().InitGlobalData()`。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-p"></a>
### 4.10 预测（Prediction）
GAS 开箱即支持客户端预测，但它并不会预测所有东西。GAS 中的客户端预测是指：客户端无需等待服务器的许可，就可以激活 `GameplayAbility` 并施加 `GameplayEffects`。它可以“预测”服务器会允许它这样做，并预测它会把 `GameplayEffects` 施加到哪些目标上。之后，服务器会在客户端激活后、经过一个网络延迟的时间再运行该 `GameplayAbility`，并告诉客户端它的预测是否正确。如果客户端的任何预测是错误的，它会把这些“错误预测”造成的修改“回滚（roll back）”，以与服务器保持一致。

关于 GAS 预测的权威资料，是插件源码中的 `GameplayPrediction.h`。

Epic 的思路是只预测那些“可以蒙混过关”的东西。例如，Paragon 和 Fortnite 都不预测伤害。它们的伤害很可能使用的是 [`ExecutionCalculations`](#concepts-ge-ec)，而这本来就无法被预测。这并不是说你不能尝试预测伤害之类的东西。如果你这么做了，而且效果不错，那当然很好。

> ……我们也并不完全认同“自动、无缝地预测一切”的方案。我们仍然认为，玩家预测最好保持在最低限度（意思是：只预测那些你能蒙混过关的最少量的东西）。

*摘自 Epic 的 Dave Ratti 对新的[网络预测插件](#concepts-p-npp)的评论*

**会被预测的内容：**
> * 能力激活
> *	触发的事件
> *	GameplayEffect 的施加：
>    * 属性修改（例外：Execution 目前不会被预测，只有属性修改器会）
>    * GameplayTag 修改
> * GameplayCue 事件（包括来自预测性 GameplayEffect 内部的，以及单独触发的）
> * 蒙太奇
> * 移动（内置于 UE 的 UCharacterMovement 中）

**不会被预测的内容：**
> * GameplayEffect 的移除
> * GameplayEffect 的周期效果（DOT 跳伤害）

*摘自 `GameplayPrediction.h`*

虽然我们可以预测 `GameplayEffect` 的施加，但无法预测 `GameplayEffect` 的移除。绕过这一限制的一种方法是：当我们想移除某个 `GameplayEffect` 时，预测一个效果相反的 `GameplayEffect`。比如我们预测了一个 40% 的移动速度减速，就可以通过施加一个 40% 的移动速度增益来预测性地把它“移除”，然后再同时移除这两个 `GameplayEffects`。这种做法并不适用于所有场景，对预测 `GameplayEffect` 移除的支持仍然是需要的。Epic 的 Dave Ratti 曾表示希望在 [GAS 的未来迭代](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)中加入这项支持。

由于无法预测 `GameplayEffects` 的移除，我们也就无法完全预测 `GameplayAbility` 的冷却，而且对冷却来说也没有“相反 `GameplayEffect`”这种变通方法。服务器复制下来的 `Cooldown GE` 会存在于客户端上，任何试图绕过它的做法（例如使用 `Minimal` 复制模式）都会被服务器拒绝。这意味着高延迟的客户端需要更长时间才能通知服务器进入冷却，也需要更长时间才能收到服务器 `Cooldown GE` 的移除。结果就是，高延迟玩家的射速会低于低延迟玩家，在与低延迟玩家的对抗中处于劣势。Fortnite 通过使用自定义的计数逻辑而不是 `Cooldown GEs` 来避免这个问题。

关于预测伤害，尽管这是大多数人刚开始用 GAS 时首先会尝试的事情之一，但我个人并不推荐。我尤其不推荐尝试预测死亡。虽然可以预测伤害，但做起来很棘手。如果你错误地预测了伤害的施加，玩家会看到敌人的生命值又跳了回去。如果你尝试预测死亡，这种情况会格外尴尬和令人沮丧。比如你错误地预测了一个 `Character` 的死亡，它开始布娃娃（ragdoll）化，结果服务器纠正后，它又停止布娃娃化，继续朝你开枪。

**注意：** 修改 `Attributes` 的 `Instant` `GameplayEffects`（比如 `Cost GEs`）在自己身上可以被无缝预测，但预测其他角色身上的 `Instant` `Attribute` 修改时，他们的 `Attributes` 会出现短暂的异常或“闪跳（blip）”。被预测的 `Instant` `GameplayEffects` 实际上会被当作 `Infinite` `GameplayEffects` 处理，以便在预测错误时可以回滚。当服务器的 `GameplayEffect` 被施加时，可能会短暂地同时存在两个相同的 `GameplayEffect`，导致 `Modifier` 在一小段时间内被应用两次或完全没有被应用。它最终会自行纠正，但有时这种闪跳会被玩家察觉到。

GAS 的预测实现试图解决的问题：
> 1. “我能做这件事吗？”——预测的基本协议。
> 2. “撤销（Undo）”——预测失败时如何撤销副作用。
> 3. “重做（Redo）”——如何避免重复播放那些我们已在本地预测、但又从服务器复制过来的副作用。
> 4. “完整性（Completeness）”——如何确保我们/真的/预测了所有副作用。
> 5. “依赖（Dependencies）”——如何管理相互依赖的预测以及预测事件链。
> 6. “覆盖（Override）”——如何预测性地覆盖原本由服务器复制/拥有的状态。

*摘自 `GameplayPrediction.h`*

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-p-key"></a>
#### 4.10.1 预测键（Prediction Key）
GAS 的预测基于 `Prediction Key`（预测键）这一概念，它是客户端在激活 `GameplayAbility` 时生成的一个整数标识符。

* 客户端在激活 `GameplayAbility` 时生成一个预测键，这就是 `Activation Prediction Key`（激活预测键）。
* 客户端通过 `CallServerTryActivateAbility()` 把这个预测键发送给服务器。
* 在预测键有效期间，客户端会把这个预测键添加到它施加的所有 `GameplayEffects` 上。
* 客户端的预测键离开作用域。同一个 `GameplayAbility` 中后续的预测效果需要一个新的[作用域预测窗口（Scoped Prediction Window）](#concepts-p-windows)。


* 服务器从客户端收到预测键。
* 服务器把这个预测键添加到它施加的所有 `GameplayEffects` 上。
* 服务器把这个预测键复制回客户端。


* 客户端从服务器收到被复制的 `GameplayEffects`，以及施加它们时所用的预测键。如果某个被复制的 `GameplayEffects` 与客户端用同一个预测键施加的 `GameplayEffects` 相匹配，说明它们被正确预测了。在客户端移除它预测的那个之前，目标身上会暂时存在两份该 `GameplayEffect`。
* 客户端从服务器收回预测键，这就是 `Replicated Prediction Key`（被复制的预测键）。此时这个预测键被标记为过期（stale）。
* 客户端移除它用这个已过期的被复制预测键创建的**所有** `GameplayEffects`。由服务器复制过来的 `GameplayEffects` 会保留下来。客户端添加了、但没有从服务器收到对应复制版本的任何 `GameplayEffects`，都属于预测错误。

在 `GameplayAbilities` 中，预测键保证在一个由原子指令分组构成的“窗口”内有效，这个窗口从使用激活预测键的 `Activation` 开始。你可以把它理解为只在一帧之内有效。来自潜伏动作 `AbilityTasks` 的任何回调都不再拥有有效的预测键，除非该 `AbilityTask` 内置了同步点（Synch Point），会生成一个新的[作用域预测窗口](#concepts-p-windows)。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-p-windows"></a>
#### 4.10.2 在能力中创建新的预测窗口
要在 `AbilityTasks` 的回调中预测更多动作，我们需要用一个新的作用域预测键（Scoped Prediction Key）创建一个新的作用域预测窗口（Scoped Prediction Window）。这有时被称为客户端与服务器之间的同步点（Synch Point）。有些 `AbilityTasks`（比如所有与输入相关的任务）内置了创建新作用域预测窗口的功能，也就是说，这些 `AbilityTasks` 回调中的原子代码有一个可用的有效作用域预测键。其他任务（比如 `WaitDelay` 任务）则没有为其回调创建新作用域预测窗口的内置代码。如果你需要在一个没有内置创建作用域预测窗口代码的 `AbilityTask`（比如 `WaitDelay`）之后预测动作，就必须使用带 `OnlyServerWait` 选项的 `WaitNetSync` `AbilityTask` 手动创建。当客户端执行到带 `OnlyServerWait` 的 `WaitNetSync` 时，它会基于该 `GameplayAbility` 的激活预测键生成一个新的作用域预测键，通过 RPC 发送给服务器，并把它添加到之后施加的所有新 `GameplayEffects` 上。当服务器执行到带 `OnlyServerWait` 的 `WaitNetSync` 时，它会等待，直到从客户端收到新的作用域预测键后才继续执行。这个作用域预测键会经历与激活预测键相同的流程——被添加到 `GameplayEffects` 上，再被复制回客户端并标记为过期。作用域预测键在离开作用域之前一直有效，离开作用域意味着作用域预测窗口已关闭。所以再说一遍，只有原子操作（不能是潜伏的）才能使用新的作用域预测键。

你可以根据需要创建任意多个作用域预测窗口。

如果你想为自己的自定义 `AbilityTasks` 添加同步点功能，可以看看那些输入相关的任务，它们本质上就是把 `WaitNetSync` `AbilityTask` 的代码注入到了自身中。

**注意：** 使用 `WaitNetSync` 时，服务器上的 `GameplayAbility` 确实会被阻塞，直到收到客户端的消息才会继续执行。这可能会被恶意用户利用：他们修改游戏，故意延迟发送新的作用域预测键。虽然 Epic 很少使用 `WaitNetSync`，但如果你担心这一点，Epic 建议构建一个新版本的 `AbilityTask`，加入超时机制，在超时后不等客户端就自动继续执行。

示例项目在冲刺 `GameplayAbility` 中使用了 `WaitNetSync`，在每次施加耐力消耗时创建一个新的作用域预测窗口，以便预测这次消耗。理想情况下，我们希望在施加消耗和冷却时拥有一个有效的预测键。

如果有一个被预测的 `GameplayEffect` 在拥有者客户端上播放了两次，说明你的预测键已经过期，你遇到了“重做（redo）”问题。通常可以在施加 `GameplayEffect` 之前，放置一个带 `OnlyServerWait` 的 `WaitNetSync` `AbilityTask` 来创建一个新的作用域预测键，从而解决这个问题。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-p-spawn"></a>
#### 4.10.3 预测性地生成 Actor
在客户端上预测性地生成 `Actors` 是一个进阶主题。GAS 并没有提供开箱即用的功能来处理这种情况（`SpawnActor` `AbilityTask` 只会在服务器上生成 `Actor`）。关键思路是在客户端和服务器上都生成一个被复制的 `Actor`。

如果这个 `Actor` 只是表现性的，或者不承担任何游戏玩法上的作用，简单的解决方法是重写该 `Actor` 的 `IsNetRelevantFor()` 函数，限制服务器把它复制给拥有者客户端。这样，拥有者客户端拥有自己在本地生成的版本，而服务器和其他客户端拥有服务器复制的版本。
```c++
bool APAReplicatedActorExceptOwner::IsNetRelevantFor(const AActor * RealViewer, const AActor * ViewTarget, const FVector & SrcLocation) const
{
	return !IsOwnedBy(ViewTarget);
}
```

如果生成的 `Actor` 会影响游戏玩法，比如需要预测伤害的投射物，那么你就需要更高级的逻辑，这超出了本文档的范围。可以去 Epic Games 的 GitHub 上看看 UnrealTournament 是如何预测性地生成投射物的。他们会生成一个只存在于拥有者客户端上的虚拟（dummy）投射物，并让它与服务器复制的投射物进行同步。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-p-future"></a>
#### 4.10.4 GAS 中预测的未来
`GameplayPrediction.h` 中提到，将来他们可能会加入预测 `GameplayEffect` 移除和周期性 `GameplayEffects` 的功能。

Epic 的 Dave Ratti [表示有兴趣](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)修复预测冷却时的 `latency reconciliation`（延迟校正）问题，这个问题让高延迟玩家相对于低延迟玩家处于劣势。

Epic 的新 [`Network Prediction` 插件](#concepts-p-npp)预计会像之前的 `CharacterMovementComponent` *曾经*那样，与 GAS 完全互通。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-p-npp"></a>
#### 4.10.5 网络预测插件（Network Prediction Plugin）
Epic 最近启动了一项计划，要用新的 `Network Prediction` 插件取代 `CharacterMovementComponent`。这个插件仍处于非常早期的阶段，但已经可以在 Unreal Engine 的 GitHub 上以极早期预览的形式获取。目前还说不准它会在哪个未来版本的引擎中以实验性 Beta 的形式首次亮相。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-targeting"></a>
### 4.11 目标选择（Targeting）

<a name="concepts-targeting-data"></a>
#### 4.11.1 目标数据（Target Data）
[`FGameplayAbilityTargetData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetData/index.html) 是一个通用的目标数据结构体，用于在网络上传递。`TargetData` 通常保存 `AActor`/`UObject` 引用、`FHitResults` 以及其他通用的位置/方向/起点信息。不过你可以子类化它，往里面放几乎任何你想要的东西，以此作为[在 `GameplayAbilities` 中于客户端与服务器之间传递数据](#concepts-ga-data)的简单手段。基础结构体 `FGameplayAbilityTargetData` 不应直接使用，而应被子类化。`GAS` 自带了几个开箱即用的 `FGameplayAbilityTargetData` 子结构体，位于 `GameplayAbilityTargetTypes.h` 中。

`TargetData` 通常由 [`Target Actors`](#concepts-targeting-actors) 产生，或者**手动创建**，然后由 [`AbilityTasks`](#concepts-at) 以及通过 [`EffectContext`](#concepts-ge-context) 由 [`GameplayEffects`](#concepts-ge) 消费。由于它存在于 `EffectContext` 中，[`Executions`](#concepts-ge-ec)、[`MMCs`](#concepts-ge-mmc)、[`GameplayCues`](#concepts-gc) 以及 [`AttributeSet`](#concepts-as) 后端的函数都可以访问 `TargetData`。

我们通常不直接传递 `FGameplayAbilityTargetData`，而是使用 [`FGameplayAbilityTargetDataHandle`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetDataHandle/index.html)，它内部有一个指向 `FGameplayAbilityTargetData` 的指针 TArray。这个中间结构体为 `TargetData` 提供了多态支持。

继承 `FGameplayAbilityTargetData` 的示例：
```c++
USTRUCT(BlueprintType)
struct MYGAME_API FGameplayAbilityTargetData_CustomData : public FGameplayAbilityTargetData
{
    GENERATED_BODY()
public:

    FGameplayAbilityTargetData_CustomData()
    { }

    UPROPERTY()
    FName CoolName = NAME_None;

    UPROPERTY()
    FPredictionKey MyCoolPredictionKey;

    // This is required for all child structs of FGameplayAbilityTargetData
    // FGameplayAbilityTargetData 的所有子结构体都必须实现它
    virtual UScriptStruct* GetScriptStruct() const override
    {
    	return FGameplayAbilityTargetData_CustomData::StaticStruct();
    }

	// This is required for all child structs of FGameplayAbilityTargetData
	// FGameplayAbilityTargetData 的所有子结构体都必须实现它
    bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
    {
	    // The engine already defined NetSerialize for FName & FPredictionKey, thanks Epic!
	    // 引擎已经为 FName 和 FPredictionKey 定义了 NetSerialize，感谢 Epic！
        CoolName.NetSerialize(Ar, Map, bOutSuccess);
        MyCoolPredictionKey.NetSerialize(Ar, Map, bOutSuccess);
        bOutSuccess = true;
        return true;
    }
}

template<>
struct TStructOpsTypeTraits<FGameplayAbilityTargetData_CustomData> : public TStructOpsTypeTraitsBase2<FGameplayAbilityTargetData_CustomData>
{
	enum
	{
        WithNetSerializer = true // This is REQUIRED for FGameplayAbilityTargetDataHandle net serialization to work
        // 要让 FGameplayAbilityTargetDataHandle 的网络序列化正常工作，这一项是必需的
	};
};
```
把 TargetData 添加到 Handle 中：
```c++
UFUNCTION(BlueprintPure)
FGameplayAbilityTargetDataHandle MakeTargetDataFromCustomName(const FName CustomName)
{
	// Create our target data type, 
	// 创建我们的 TargetData 类型，
	// Handle's automatically cleanup and delete this data when the handle is destructed, 
	// Handle 在析构时会自动清理并删除这份数据，
	// if you don't add this to a handle then be careful because this deals with memory management and memory leaks so its safe to just always add it to a handle at some point in the frame!
	// 如果你不把它加入某个 Handle，就要小心了，因为这涉及内存管理和内存泄漏，所以最稳妥的做法是总在这一帧的某个时刻把它加入 Handle！
	FGameplayAbilityTargetData_CustomData* MyCustomData = new FGameplayAbilityTargetData_CustomData();
	// Setup the struct's information to use the inputted name and any other changes we may want to do
	// 设置结构体的信息：使用传入的名称，以及我们想做的其他修改
	MyCustomData->CoolName = CustomName;
	
	// Make our handle wrapper for Blueprint usage
	// 创建供蓝图使用的 Handle 包装
	FGameplayAbilityTargetDataHandle Handle;
	// Add the target data to our handle
	// 把 TargetData 添加到我们的 Handle 中
	Handle.Add(MyCustomData);
	// Output our handle to Blueprint
	// 把 Handle 输出给蓝图
	return Handle
}
```

获取值时需要做类型安全检查，因为从 Handle 的 TargetData 中取值的唯一方式是使用通用的 C/C++ 类型转换，而它*不是*类型安全的，可能导致对象切片（object slicing）和崩溃。类型检查有多种方法（说实话，怎么做都行），两种常见的方法是：
- GameplayTag：你可以使用一种子类层级结构，只要知道某种代码架构的功能总会出现，就可以先转换为基础父类型，取得它的 GameplayTag，再据此判断如何转换为派生类。
- Script Struct 与 Static Struct：你也可以直接比较类（这可能涉及大量 IF 语句，或者编写一些模板函数）。下面给出了一个示例，基本思路是：可以从任何 `FGameplayAbilityTargetData` 获取它的 Script Struct（这是它作为 `USTRUCT` 的一大好处，并且要求所有派生类在 `GetScriptStruct` 中指定结构体类型），然后比较它是否是你要找的类型。下面是使用这些函数进行类型检查的示例：
```c++
UFUNCTION(BlueprintPure)
FName GetCoolNameFromTargetData(const FGameplayAbilityTargetDataHandle& Handle, const int Index)
{   
    // NOTE, there is two versions of this '::Get(int32 Index)' function; 
    // 注意，'::Get(int32 Index)' 函数有两个版本：
    // 1) const version that returns 'const FGameplayAbilityTargetData*', good for reading target data values 
    // 1) const 版本，返回 'const FGameplayAbilityTargetData*'，适合读取 TargetData 的值
    // 2) non-const version that returns 'FGameplayAbilityTargetData*', good for modifying target data values
    // 2) 非 const 版本，返回 'FGameplayAbilityTargetData*'，适合修改 TargetData 的值
    FGameplayAbilityTargetData* Data = Handle.Get(Index); // This will valid check the index for you 
    // 它会替你检查索引是否有效
    
    // Valid check we have something to use, null data means nothing to cast for
    // 检查是否有可用的数据，数据为空就没有可转换的对象
    if(Data == nullptr)
    {
       	return NAME_None;
    }
    // This is basically the type checking pass, static_cast does not have type safety, this is why we do this check.
    // 这一步基本上就是类型检查。static_cast 不具备类型安全，所以我们才要做这个检查。
    // If we don't do this then it will object slice the struct and thus we have no way of making sure its that type.
    // 如果不做这个检查，结构体就会发生对象切片，我们也就无法确保它就是那个类型。
    if(Data->GetScriptStruct() == FGameplayAbilityTargetData_CustomData::StaticStruct())
    {
        // Here is when you would do the cast because we know its the correct type already
        // 此时才进行转换，因为我们已经确定它是正确的类型
        FGameplayAbilityTargetData_CustomData* CustomData = static_cast<FGameplayAbilityTargetData_CustomData*>(Data);    
        return CustomData->CoolName;
    }
    return NAME_None;
}
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-targeting-actors"></a>
#### 4.11.2 目标 Actor（Target Actor）
`GameplayAbilities` 通过 `WaitTargetData` `AbilityTask` 生成 [`TargetActors`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor/index.html)，用来可视化并从世界中捕获目标信息。`TargetActors` 可以选择使用 [`GameplayAbilityWorldReticles`](#concepts-targeting-reticles) 来显示当前目标。确认之后，目标信息会以 [`TargetData`](#concepts-targeting-data) 的形式返回，随后可以传入 `GameplayEffects`。
 
`TargetActors` 基于 `AActor`，所以它们可以拥有任何类型的可见组件来表示它们瞄准的是**哪里**以及**如何**瞄准，比如静态网格体或贴花（decal）。静态网格体可以用来可视化角色将要建造的物体的摆放位置；贴花可以用来在地面上显示效果范围。示例项目使用 [`AGameplayAbilityTargetActor_GroundTrace`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor_Grou-/index.html) 并在地面上配一个贴花，来表示陨石能力的伤害范围。它们也可以什么都不显示。例如，[GASShooter](https://github.com/tranek/GASShooter) 中那种瞬间向目标做射线检测的即时命中枪械，显示任何东西都没有意义。

它们使用基础的射线检测（trace）或碰撞重叠（overlap）来捕获目标信息，并根据 `TargetActor` 的具体实现，把结果以 `FHitResults` 或 `AActor` 数组的形式转换为 `TargetData`。`WaitTargetData` `AbilityTask` 通过其 `TEnumAsByte<EGameplayTargetingConfirmation::Type> ConfirmationType` 参数决定目标何时被确认。当**不**使用 `TEnumAsByte<EGameplayTargetingConfirmation::Type::Instant` 时，`TargetActor` 通常会在 `Tick()` 中执行射线检测/重叠检测，并根据其实现把自身位置更新到 `FHitResult` 处。虽然这会在 `Tick()` 中执行射线检测/重叠检测，但一般来说问题不大，因为它不会被复制，而且通常同一时间只有一个 `TargetActor` 在运行（尽管也可以有多个）。只需注意它用到了 `Tick()`，而一些复杂的 `TargetActors` 可能会在其中做很多事情，比如 GASShooter 中火箭发射器的副能力。虽然在 `Tick()` 中做射线检测对客户端来说响应非常灵敏，但如果性能开销太大，可以考虑降低 `TargetActor` 的 Tick 频率。在使用 `TEnumAsByte<EGameplayTargetingConfirmation::Type::Instant` 的情况下，`TargetActor` 会立即生成、产生 `TargetData`，然后销毁，永远不会调用 `Tick()`。

| `EGameplayTargetingConfirmation::Type` | 目标何时被确认                                                                                                                                                                                                                                         |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Instant`                              | 立即进行目标选择，不需要任何特殊逻辑或用户输入来决定何时“开火”。                                                                                                                                                                                       |
| `UserConfirmed`                        | 当用户确认目标选择时进行，即[能力被绑定到 `Confirm` 输入](#concepts-ga-input)时由该输入确认，或者调用 `UAbilitySystemComponent::TargetConfirm()`。`TargetActor` 也会响应绑定的 `Cancel` 输入或对 `UAbilitySystemComponent::TargetCancel()` 的调用，以取消目标选择。 |
| `Custom`                               | 由负责目标选择的 GameplayAbility 通过调用 `UGameplayAbility::ConfirmTaskByInstanceName()` 来决定目标数据何时就绪。`TargetActor` 也会响应 `UGameplayAbility::CancelTaskByInstanceName()` 以取消目标选择。                                                |
| `CustomMulti`                          | 由负责目标选择的 GameplayAbility 通过调用 `UGameplayAbility::ConfirmTaskByInstanceName()` 来决定目标数据何时就绪。`TargetActor` 也会响应 `UGameplayAbility::CancelTaskByInstanceName()` 以取消目标选择。产生数据后不应结束 `AbilityTask`。             |

并非每个 `TargetActor` 都支持所有 EGameplayTargetingConfirmation::Type。例如，`AGameplayAbilityTargetActor_GroundTrace` 不支持 `Instant` 确认。

`WaitTargetData` `AbilityTask` 接受一个 `AGameplayAbilityTargetActor` 类作为参数，每次激活该 `AbilityTask` 时都会生成一个实例，并在 `AbilityTask` 结束时销毁该 `TargetActor`。`WaitTargetDataUsingActor` `AbilityTask` 接受一个已经生成好的 `TargetActor`，但在 `AbilityTask` 结束时仍会销毁它。这两个 `AbilityTasks` 都比较低效，因为它们每次使用都要么生成、要么需要一个新生成的 `TargetActor`。它们非常适合用来做原型，但在正式产品中，如果你有持续产生 `TargetData` 的情况（比如自动步枪），可能需要考虑优化。GASShooter 有一个自定义的 [`AGameplayAbilityTargetActor`](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/GSGATA_Trace.h) 子类，以及一个从零编写的新 [`WaitTargetDataWithReusableActor`](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/AbilityTasks/GSAT_WaitTargetDataUsingActor.h) `AbilityTask`，允许你复用 `TargetActor` 而不销毁它。

`TargetActors` 默认不会被复制；不过，如果在你的游戏中向其他玩家展示本地玩家正在瞄准哪里是合理的，也可以让它们被复制。它们内置了通过 `WaitTargetData` `AbilityTask` 上的 RPC 与服务器通信的默认功能。如果 `TargetActor` 的 `ShouldProduceTargetDataOnServer` 属性设为 `false`，客户端会在确认时，在 `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()` 中通过 `CallServerSetReplicatedTargetData()` 把它的 `TargetData` 以 RPC 发送给服务器。如果 `ShouldProduceTargetDataOnServer` 为 `true`，客户端会在 `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()` 中向服务器发送一个通用确认事件 `EAbilityGenericReplicatedEvent::GenericConfirm` 的 RPC，服务器收到 RPC 后会自行执行射线检测或重叠检测，在服务器上产生数据。如果客户端取消了目标选择，它会在 `UAbilityTask_WaitTargetData::OnTargetDataCancelledCallback` 中向服务器发送一个通用取消事件 `EAbilityGenericReplicatedEvent::GenericCancel` 的 RPC。可以看到，`TargetActor` 和 `WaitTargetData` `AbilityTask` 上都有大量委托。`TargetActor` 响应输入，产生 `TargetData` 并广播就绪、确认或取消委托。`WaitTargetData` 监听 `TargetActor` 的 `TargetData` 就绪、确认和取消委托，并把这些信息转发回 `GameplayAbility` 和服务器。如果你把 `TargetData` 发送给服务器，可能需要在服务器上进行校验，确保 `TargetData` 看起来合理，以防作弊。直接在服务器上产生 `TargetData` 可以完全避免这个问题，但可能会导致拥有者客户端出现预测错误。

根据你使用的 `AGameplayAbilityTargetActor` 具体子类的不同，`WaitTargetData` `AbilityTask` 节点上会暴露不同的 `ExposeOnSpawn` 参数。一些常见的参数包括：

| 常见的 `TargetActor` 参数       | 定义                                                                                                                                                                                                       |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Debug                           | 如果为 `true`，在非 Shipping 构建中，每当 `TargetActor` 执行射线检测时，都会绘制调试用的射线检测/重叠信息。记住，非 `Instant` 的 `TargetActors` 会在 `Tick()` 中执行射线检测，所以这些调试绘制调用也会在 `Tick()` 中发生。 |
| Filter                          | [可选] 一个特殊的结构体，用于在射线检测/重叠检测发生时从目标中过滤掉（移除）某些 `Actors`。典型用例是过滤掉玩家自己的 `Pawn`，或要求目标必须是某个特定的类。更高级的用例请参阅[目标数据过滤器](#concepts-target-data-filters)。 |
| Reticle Class                   | [可选] `TargetActor` 将要生成的 `AGameplayAbilityWorldReticle` 子类。                                                                                                                                       |
| Reticle Parameters              | [可选] 配置你的准星。参见[准星](#concepts-targeting-reticles)。                                                                                                                                            |
| Start Location                  | 一个特殊的结构体，指定射线检测的起点。通常是玩家的视点、武器枪口或 `Pawn` 的位置。                                                                                                                          |

使用默认的 `TargetActor` 类时，只有当 `Actors` 直接处于射线检测/重叠范围内时，才是有效的目标。如果它们离开了射线检测/重叠范围（它们移动了，或者你把视线移开了），就不再有效。如果你希望 `TargetActor` 记住最后一个（或几个）有效目标，就需要在自定义的 `TargetActor` 类中加入这个功能。我把这些称为持久目标（persistent targets），因为它们会一直保留，直到 `TargetActor` 收到确认或取消、`TargetActor` 在其射线检测/重叠范围内找到了新的有效目标，或者目标不再有效（被销毁）。GASShooter 在火箭发射器副能力的追踪火箭目标选择中使用了持久目标。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-target-data-filters"></a>
#### 4.11.3 目标数据过滤器（Target Data Filter）
同时使用 `Make GameplayTargetDataFilter` 和 `Make Filter Handle` 节点，可以过滤掉玩家自己的 `Pawn`，或者只选择某个特定的类。如果需要更高级的过滤，可以子类化 `FGameplayTargetDataFilter` 并重写 `FilterPassesForActor` 函数。
```c++
USTRUCT(BlueprintType)
struct GASDOCUMENTATION_API FGDNameTargetDataFilter : public FGameplayTargetDataFilter
{
	GENERATED_BODY()

	/** Returns true if the actor passes the filter and will be targeted */
	/** 如果该 Actor 通过了过滤器、将被选为目标，则返回 true */
	virtual bool FilterPassesForActor(const AActor* ActorToBeFiltered) const override;
};
```

不过，这样还不能直接用于 `Wait Target Data` 节点，因为它需要一个 `FGameplayTargetDataFilterHandle`。必须新建一个自定义的 `Make Filter Handle` 来接受这个子类：
```c++
FGameplayTargetDataFilterHandle UGDTargetDataFilterBlueprintLibrary::MakeGDNameFilterHandle(FGDNameTargetDataFilter Filter, AActor* FilterActor)
{
	FGameplayTargetDataFilter* NewFilter = new FGDNameTargetDataFilter(Filter);
	NewFilter->InitializeFilterContext(FilterActor);

	FGameplayTargetDataFilterHandle FilterHandle;
	FilterHandle.Filter = TSharedPtr<FGameplayTargetDataFilter>(NewFilter);
	return FilterHandle;
}
```

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-targeting-reticles"></a>
#### 4.11.4 GameplayAbility 世界准星（World Reticle）
在使用非 `Instant` 确认方式的 [`TargetActors`](#concepts-targeting-actors) 进行目标选择时，[`AGameplayAbilityWorldReticles`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityWorldReticle/index.html)（`Reticles`，准星）用来可视化你正在瞄准**谁**。`TargetActors` 负责所有 `Reticles` 的生成与销毁。`Reticles` 是 `AActors`，所以它们可以使用任何类型的可视组件来表现。一种常见的实现方式（如 [GASShooter](https://github.com/tranek/GASShooter) 中所见）是使用 `WidgetComponent` 在屏幕空间中显示一个 UMG 控件（始终朝向玩家的相机）。`Reticles` 并不知道自己附着在哪个 `AActor` 上，但你可以在自定义的 `TargetActor` 中通过子类化加入这个功能。`TargetActors` 通常会在每次 `Tick()` 时把 `Reticle` 的位置更新到目标的位置。

GASShooter 用 `Reticles` 来显示火箭发射器副能力的追踪火箭所锁定的目标。敌人身上的红色指示器就是 `Reticle`，而类似的白色图像是火箭发射器的准心。
![GASShooter 中的准星](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplayabilityworldreticle.png)

`Reticles` 自带一些供策划使用的 `BlueprintImplementableEvents`（它们本来就是打算在蓝图中开发的）：

```c++
/** Called whenever bIsTargetValid changes value. */
/** 每当 bIsTargetValid 的值发生变化时调用。 */
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnValidTargetChanged(bool bNewValue);

/** Called whenever bIsTargetAnActor changes value. */
/** 每当 bIsTargetAnActor 的值发生变化时调用。 */
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnTargetingAnActor(bool bNewValue);

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnParametersInitialized();

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void SetReticleMaterialParamFloat(FName ParamName, float value);

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void SetReticleMaterialParamVector(FName ParamName, FVector value);
```

`Reticles` 可以选择使用 `TargetActor` 提供的 [`FWorldReticleParameters`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FWorldReticleParameters/index.html) 进行配置。默认的结构体只提供一个变量 `FVector AOEScale`。虽然从技术上讲你可以子类化这个结构体，但 `TargetActor` 只接受基础结构体。默认的 `TargetActors` 不允许使用它的子类，这似乎有点欠考虑。不过，如果你编写自己的自定义 `TargetActor`，就可以提供自定义的准星参数结构体，并在生成 `AGameplayAbilityWorldReticles` 子类时手动把它传进去。

`Reticles` 默认不会被复制，但如果在你的游戏中向其他玩家展示本地玩家正在瞄准谁是合理的，也可以让它们被复制。

使用默认的 `TargetActors` 时，`Reticles` 只会显示在当前的有效目标上。例如，如果你使用 `AGameplayAbilityTargetActor_SingleLineTrace` 通过射线检测来寻找目标，那么只有当敌人直接处于射线路径上时，`Reticle` 才会出现。如果你把视线移开，敌人就不再是有效目标，`Reticle` 也会消失。如果你希望 `Reticle` 停留在最后一个有效目标上，就需要定制你的 `TargetActor`，让它记住最后一个有效目标并把 `Reticle` 保持在其身上。我把这些称为持久目标，因为它们会一直保留，直到 `TargetActor` 收到确认或取消、`TargetActor` 在其射线检测/重叠范围内找到了新的有效目标，或者目标不再有效（被销毁）。GASShooter 在火箭发射器副能力的追踪火箭目标选择中使用了持久目标。

**[⬆ 返回目录](#table-of-contents)**

<a name="concepts-targeting-containers"></a>
#### 4.11.5 GameplayEffect 容器的目标选择
[`GameplayEffectContainers`](#concepts-ge-containers) 提供了一种可选的、高效的产生 [`TargetData`](#concepts-targeting-data) 的方式。当 `EffectContainer` 在客户端和服务器上被施加时，这种目标选择会立即进行。它比 [`TargetActors`](#concepts-targeting-actors) 更高效，因为它在目标选择对象的 CDO 上运行（不需要生成和销毁 `Actors`），但它缺少玩家输入，会立即发生而无需确认，无法被取消，也无法把数据从客户端发送到服务器（两端各自产生数据）。它非常适合即时射线检测和碰撞重叠检测。Epic 的 [Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)在其容器中包含了两种目标选择的示例类型——以能力拥有者为目标，以及从事件中提取 `TargetData`。它还用蓝图实现了一种：从玩家位置偏移一定距离（由蓝图子类设置）做即时球形检测（sphere trace）。你可以在 C++ 或蓝图中子类化 `URPGTargetType`，制作自己的目标选择类型。

**[⬆ 返回目录](#table-of-contents)**

<a name="cae"></a>
## 5. 常见能力与效果的实现

<a name="cae-stun"></a>
### 5.1 眩晕（Stun）
对于眩晕，我们通常希望在眩晕持续期间取消 `Character` 所有处于激活状态的 `GameplayAbilities`，阻止新的 `GameplayAbility` 激活，并禁止移动。示例项目中的陨石 `GameplayAbility` 会对命中的目标施加眩晕。

为了取消目标处于激活状态的 `GameplayAbilities`，我们在眩晕 [`GameplayTag` 被添加时](#concepts-gt-change)调用 `AbilitySystemComponent->CancelAbilities()`。

为了在眩晕期间阻止新的 `GameplayAbilities` 激活，会把眩晕 `GameplayTag` 放进这些 `GameplayAbilities` 的 [`Activation Blocked Tags` `GameplayTagContainer`](#concepts-ga-tags) 中。

为了在眩晕期间禁止移动，我们重写 `CharacterMovementComponent` 的 `GetMaxSpeed()` 函数，当拥有者拥有眩晕 `GameplayTag` 时返回 0。

**[⬆ 返回目录](#table-of-contents)**

<a name="cae-sprint"></a>
### 5.2 冲刺（Sprint）
示例项目提供了一个冲刺的例子——按住 `Left Shift`（左 Shift）时跑得更快。

更快的移动由 `CharacterMovementComponent` 通过向服务器发送一个网络标志位来进行预测性处理。详情请参阅 `GDCharacterMovementComponent.h/cpp`。

`GA` 负责响应 `Left Shift` 输入，通知 `CMC` 开始和停止冲刺，并在按住 `Left Shift` 期间预测性地扣除耐力。详情请参阅 `GA_Sprint_BP`。

**[⬆ 返回目录](#table-of-contents)**

<a name="cae-ads"></a>
### 5.3 开镜瞄准（Aim Down Sights）
示例项目对它的处理方式与冲刺完全相同，只不过是降低移动速度而不是提高。

关于如何预测性地降低移动速度，详情请参阅 `GDCharacterMovementComponent.h/cpp`。

关于如何处理输入，详情请参阅 `GA_AimDownSight_BP`。开镜瞄准没有耐力消耗。

**[⬆ 返回目录](#table-of-contents)**

<a name="cae-ls"></a>
### 5.4 吸血（Lifesteal）
我在伤害 [`ExecutionCalculation`](#concepts-ge-ec) 内部处理吸血。`GameplayEffect` 上会带有一个类似 `Effect.CanLifesteal` 的 `GameplayTag`。`ExecutionCalculation` 会检查 `GameplayEffectSpec` 是否拥有这个 `Effect.CanLifesteal` `GameplayTag`。如果有，`ExecutionCalculation` 就会[创建一个动态的 `Instant` `GameplayEffect`](#concepts-ge-dynamic)，以要回复的生命值作为修改器，并把它施加回 `Source` 的 `ASC`。

```c++
if (SpecAssetTags.HasTag(FGameplayTag::RequestGameplayTag(FName("Effect.Damage.CanLifesteal"))))
{
	float Lifesteal = Damage * LifestealPercent;

	UGameplayEffect* GELifesteal = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Lifesteal")));
	GELifesteal->DurationPolicy = EGameplayEffectDurationType::Instant;

	int32 Idx = GELifesteal->Modifiers.Num();
	GELifesteal->Modifiers.SetNum(Idx + 1);
	FGameplayModifierInfo& Info = GELifesteal->Modifiers[Idx];
	Info.ModifierMagnitude = FScalableFloat(Lifesteal);
	Info.ModifierOp = EGameplayModOp::Additive;
	Info.Attribute = UPAAttributeSetBase::GetHealthAttribute();

	SourceAbilitySystemComponent->ApplyGameplayEffectToSelf(GELifesteal, 1.0f, SourceAbilitySystemComponent->MakeEffectContext());
}
```

**[⬆ 返回目录](#table-of-contents)**

<a name="cae-random"></a>
### 5.5 在客户端和服务器上生成随机数
有时你需要在 `GameplayAbility` 内部生成“随机”数，用于后坐力或子弹散布之类的东西。客户端和服务器都希望生成相同的随机数。为此，我们必须在 `GameplayAbility` 激活时把 `random seed`（随机种子）设置成相同的值。你应该在每次激活 `GameplayAbility` 时都设置 `random seed`，以防客户端错误地预测了激活，导致它的随机数序列与服务器的不同步。

| 种子设置方法                                                                 | 说明                                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 使用激活预测键                                                               | `GameplayAbility` 的激活预测键是一个 int16，保证在 `Activation()` 中于客户端和服务器两端同步且可用。你可以在客户端和服务器上都把它设为 `random seed`。这种方法的缺点是：每次游戏启动时预测键总是从零开始，并且每生成一个键都会以固定的方式递增。这意味着每局比赛都会有完全相同的随机数序列。对你的需求来说，这可能够随机，也可能不够。 |
| 激活 `GameplayAbility` 时通过事件载荷发送种子                                | 通过事件激活 `GameplayAbility`，并通过被复制的事件载荷把随机生成的种子从客户端发送到服务器。这样随机性更高，但客户端可以轻易地修改游戏，让它每次都只发送同一个种子值。此外，通过事件激活 `GameplayAbilities` 会导致它们无法通过输入绑定激活。                                                  |

如果你的随机偏差很小，大多数玩家不会注意到每局的序列都是一样的，此时使用激活预测键作为 `random seed` 应该就够用了。如果你要做的事情更复杂、需要防作弊，那么使用 `Server Initiated` 的 `GameplayAbility` 可能更合适，由服务器创建预测键，或者生成 `random seed` 并通过事件载荷发送。

**[⬆ 返回目录](#table-of-contents)**

<a name="cae-crit"></a>
### 5.6 暴击（Critical Hit）
我在伤害 [`ExecutionCalculation`](#concepts-ge-ec) 内部处理暴击。`GameplayEffect` 上会带有一个类似 `Effect.CanCrit` 的 `GameplayTag`。`ExecutionCalculation` 会检查 `GameplayEffectSpec` 是否拥有这个 `Effect.CanCrit` `GameplayTag`。如果有，`ExecutionCalculation` 会根据暴击几率（从 `Source` 捕获的 `Attribute`）生成一个随机数，如果判定成功，就加上暴击伤害（同样是从 `Source` 捕获的 `Attribute`）。由于我不预测伤害，也就不必担心客户端和服务器上随机数生成器的同步问题，因为 `ExecutionCalculation` 只会在服务器上运行。如果你想用 `MMC` 来预测性地进行伤害计算，就必须从 `GameplayEffectSpec->GameplayEffectContext->GameplayAbilityInstance` 获取 `random seed` 的引用。

可以看看 [GASShooter](https://github.com/tranek/GASShooter) 是如何实现爆头的。思路是一样的，只不过它不依赖随机数来判定几率，而是检查 `FHitResult` 中的骨骼名称。

**[⬆ 返回目录](#table-of-contents)**

<a name="cae-nonstackingge"></a>
### 5.7 不堆叠的 GameplayEffect，且只有幅度最大的那个真正作用于目标
Paragon 中的减速效果是不堆叠的。每个减速实例都会正常施加并各自跟踪自己的生命周期，但只有幅度最大的那个减速效果会真正作用于 `Character`。GAS 通过 `AggregatorEvaluateMetaData` 为这种场景提供了开箱即用的支持。详情与实现请参阅 [`AggregatorEvaluateMetaData()`](#concepts-as-onattributeaggregatorcreated)。

**[⬆ 返回目录](#table-of-contents)**

<a name="cae-paused"></a>
### 5.8 游戏暂停时生成目标数据
如果你需要在等待玩家通过 `WaitTargetData` `AbilityTask` 生成 [`TargetData`](#concepts-targeting-data) 时暂停游戏，我建议不要真的暂停，而是使用 `slomo 0`。

**[⬆ 返回目录](#table-of-contents)**

<a name="cae-onebuttoninteractionsystem"></a>
### 5.9 单键交互系统
[GASShooter](https://github.com/tranek/GASShooter) 实现了一个单键交互系统，玩家可以按下或按住“E”键与可交互的对象互动，比如复活队友、打开武器箱、打开或关闭滑动门。

**[⬆ 返回目录](#table-of-contents)**

<a name="debugging"></a>
## 6. 调试 GAS
调试 GAS 相关问题时，你通常会想知道这样一些事情：
> * “我的属性值是多少？”
> * “我拥有哪些 GameplayTag？”
> * “我当前身上有哪些 GameplayEffect？”
> * “我被授予了哪些能力，哪些正在运行，哪些被阻止激活？”

GAS 自带两种在运行时回答这些问题的手段：[`showdebug abilitysystem`](#debugging-sd) 和 [`GameplayDebugger`](#debugging-gd) 中的钩子（hook）。

**提示：** 虚幻引擎喜欢优化 C++ 代码，这让某些函数很难调试。在深入跟踪代码时你偶尔会遇到这种情况。如果把 Visual Studio 的解决方案配置设为 `DebugGame Editor` 后，仍然无法跟踪代码或查看变量，可以用 `UE_DISABLE_OPTIMIZATION` 和 `UE_ENABLE_OPTIMIZATION` 宏（或定义在 CoreMiscDefines.h 中的 ship 变体）包住被优化的函数，从而禁用所有优化。除非从源码重新编译插件，否则这对插件代码无效。对于内联函数，这招是否有效取决于它们做了什么以及位于何处。调试完毕后一定要记得删掉这些宏！

```c++
UE_DISABLE_OPTIMIZATION
void MyClass::MyFunction(int32 MyIntParameter)
{
	// My code
	// 我的代码
}
UE_ENABLE_OPTIMIZATION
```

**[⬆ 返回目录](#table-of-contents)**

<a name="debugging-sd"></a>
### 6.1 showdebug abilitysystem
在游戏内的控制台中输入 `showdebug abilitysystem`。这个功能分为三“页”，三页都会显示你当前拥有的 `GameplayTags`。在控制台中输入 `AbilitySystem.Debug.NextCategory` 可以在各页之间切换。

第一页显示你所有 `Attributes` 的 `CurrentValue`：
![showdebug abilitysystem 的第一页](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage1.png)

第二页显示你身上所有的 `Duration` 和 `Infinite` `GameplayEffects`、它们的堆叠层数、它们赋予的 `GameplayTags`，以及它们施加的 `Modifiers`。
![showdebug abilitysystem 的第二页](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage2.png)

第三页显示所有已授予你的 `GameplayAbilities`、它们当前是否正在运行、是否被阻止激活，以及当前正在运行的 `AbilityTasks` 的状态。
![showdebug abilitysystem 的第三页](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage3.png)

要在目标之间切换（目标 Actor 周围会显示一个绿色的长方体框），使用 `PageUp` 键或 `NextDebugTarget` 控制台命令切换到下一个目标，使用 `PageDown` 键或 `PreviousDebugTarget` 控制台命令切换到上一个目标。

**注意：** 为了让能力系统信息根据当前选中的调试 Actor 进行更新，需要在 `DefaultGame.ini` 中像下面这样，在 `AbilitySystemGlobals` 里设置 `bUseDebugTargetFromHud=true`：
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
bUseDebugTargetFromHud=true
```

**注意：** 要让 `showdebug abilitysystem` 生效，必须在 GameMode 中选择一个实际的 HUD 类，否则会找不到该命令，并返回“Unknown Command”。

**[⬆ 返回目录](#table-of-contents)**

<a name="debugging-gd"></a>
### 6.2 Gameplay 调试器（Gameplay Debugger）
GAS 为 Gameplay 调试器添加了功能。按撇号键（'）打开 Gameplay 调试器，按小键盘上的 3 启用 Abilities 类别。根据你安装的插件不同，类别编号可能会有所不同。如果你的键盘没有小键盘（比如笔记本电脑），可以在项目设置中修改按键绑定。

当你想查看**其他** `Characters` 身上的 `GameplayTags`、`GameplayEffects` 和 `GameplayAbilities` 时，可以使用 Gameplay 调试器。遗憾的是，它不会显示目标 `Attributes` 的 `CurrentValue`。它会以位于屏幕中央的 `Character` 为目标。你可以在编辑器的世界大纲视图（World Outliner）中选择目标，或者看向另一个 `Character` 后再按一次撇号键（'）来切换目标。当前正在查看的 `Character` 头顶会有一个最大的红色圆圈。

![Gameplay 调试器](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaydebugger.png)

**[⬆ 返回目录](#table-of-contents)**

<a name="debugging-log"></a>
### 6.3 GAS 日志
GAS 源码中包含大量以不同详细级别（verbosity）输出的日志语句。你多半会看到它们以 `ABILITY_LOG()` 语句的形式出现。默认的详细级别是 `Display`，更高级别的日志默认不会显示在控制台中。

要修改某个日志类别的详细级别，在控制台中输入：

```
log [category] [verbosity]
```

例如，要打开 `ABILITY_LOG()` 语句，在控制台中输入：
```
log LogAbilitySystem VeryVerbose
```

要恢复为默认值，输入：
```
log LogAbilitySystem Display
```

要显示所有日志类别，输入：
```
log list
```

值得关注的 GAS 相关日志类别：

| 日志类别                  | 默认详细级别            |
| ------------------------- | ----------------------- |
| LogAbilitySystem          | Display                 |
| LogAbilitySystemComponent | Log                     |
| LogGameplayCueDetails     | Log                     |
| LogGameplayCueTranslator  | Display                 |
| LogGameplayEffectDetails  | Log                     |
| LogGameplayEffects        | Display                 |
| LogGameplayTags           | Log                     |
| LogGameplayTasks          | Log                     |
| VLogAbilitySystem         | Display                 |

更多信息请参阅[关于日志的 Wiki](https://unrealcommunity.wiki/logging-lgpidy6i)。

**[⬆ 返回目录](#table-of-contents)**

<a name="optimizations"></a>
## 7. 优化

<a name="optimizations-abilitybatching"></a>
### 7.1 能力批处理
那些在一帧之内完成激活、可选地向服务器发送 `TargetData` 并结束的 [`GameplayAbilities`](#concepts-ga)，可以通过[批处理把两到三次 RPC 压缩成一次 RPC](#concepts-ga-batching)。这类能力常用于即时命中枪械。

<a name="optimizations-gameplaycuebatching"></a>
### 7.2 GameplayCue 批处理
如果你同时发送大量 [`GameplayCues`](#concepts-gc)，可以考虑[把它们批处理成一次 RPC](#concepts-gc-batching)。目标是减少 RPC 的数量（`GameplayCues` 是不可靠的 NetMulticast），并尽可能少地发送数据。

<a name="optimizations-ascreplicationmode"></a>
### 7.3 AbilitySystemComponent 的复制模式
默认情况下，[`ASC`](#concepts-asc) 处于 [`Full` 复制模式](#concepts-asc-rm)，会把所有 [`GameplayEffects`](#concepts-ge) 复制到每个客户端（这对单人游戏来说没问题）。在多人游戏中，应把玩家拥有的 `ASCs` 设为 `Mixed` 复制模式，把 AI 控制的角色设为 `Minimal` 复制模式。这样，施加在玩家角色身上的 `GEs` 只会复制给该角色的拥有者，而施加在 AI 控制角色身上的 `GEs` 永远不会复制到客户端。无论 `Replication Mode` 是什么，[`GameplayTags`](#concepts-gt) 仍会被复制，[`GameplayCues`](#concepts-gc) 仍会以不可靠 NetMulticast 的形式发给所有客户端。当并非所有客户端都需要看到 `GEs` 时，这样做可以减少复制 `GEs` 带来的网络数据量。

<a name="optimizations-attributeproxyreplication"></a>
### 7.4 属性代理复制（Attribute Proxy Replication）
在《堡垒之夜：大逃杀》（FNBR）这类玩家众多的大型游戏中，会有大量 [`ASCs`](#concepts-asc) 位于始终相关（always-relevant）的 `PlayerStates` 上，复制着大量 [`Attributes`](#concepts-a)。为了优化这个瓶颈，Fortnite 在 `PlayerState::ReplicateSubobjects()` 中，对于**玩家控制的模拟代理（simulated player-controlled proxies）**，完全禁止了 `ASC` 及其 [`AttributeSets`](#concepts-as) 的复制。自主代理和 AI 控制的 `Pawns` 仍然会按照各自的[`复制模式`](#concepts-asc-rm)完整复制。FNBR 不在始终相关的 `PlayerStates` 的 `ASC` 上复制 `Attributes`，而是在玩家的 `Pawn` 上使用一个被复制的代理结构体（proxy structure）。当服务器 `ASC` 上的 `Attributes` 发生变化时，代理结构体上的值也会随之改变。客户端从代理结构体接收被复制的 `Attributes`，再把这些变化推回自己本地的 `ASC`。这样，`Attribute` 复制就可以利用 `Pawn` 的相关性（relevancy）和 `NetUpdateFrequency`。这个代理结构体还会以位掩码（bitmask）的形式复制一小组白名单中的 `GameplayTags`。这项优化减少了网络上的数据量，并让我们能够利用 Pawn 的相关性。AI 控制的 `Pawns` 的 `ASC` 位于 `Pawn` 上，本来就使用 `Pawn` 的相关性，因此不需要这项优化。

> 我不确定在那之后又做了其他服务器端优化（Replication Graph 等）的情况下，这样做是否仍有必要，而且这也不是最易于维护的模式。

*摘自 Epic 的 Dave Ratti 对[社区问题 #3](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)的回答*

<a name="optimizations-asclazyloading"></a>
### 7.5 ASC 延迟加载
《堡垒之夜：大逃杀》（FNBR）的世界中有大量可被破坏的 `AActors`（树木、建筑等），每个都带有一个 [`ASC`](#concepts-asc)，这些累加起来的内存开销相当可观。FNBR 的优化方式是延迟加载（lazy loading）`ASCs`，只在需要时（它们第一次受到玩家伤害时）才加载。由于有些 `AActors` 在一局比赛中可能永远不会受到伤害，这就降低了整体的内存占用。

**[⬆ 返回目录](#table-of-contents)**

<a name="qol"></a>
## 8. 易用性建议

<a name="qol-gameplayeffectcontainers"></a>
### 8.1 GameplayEffect 容器
[GameplayEffectContainers](#concepts-ge-containers) 把 [`GameplayEffectSpecs`](#concepts-ge-spec)、[`TargetData`](#concepts-targeting-data)、[简单的目标选择](#concepts-targeting-containers)以及相关功能组合成易于使用的结构体。它们非常适合把 `GameplayEffectSpecs` 传给能力生成的投射物，让投射物稍后在碰撞时施加它们。

<a name="qol-asynctasksascdelegates"></a>
### 8.2 用蓝图 AsyncTask 绑定 ASC 委托
为了缩短对策划友好的迭代时间（尤其是在为 UI 设计 UMG 控件时），可以（用 C++）创建蓝图 AsyncTask，让 UMG 蓝图图表能够直接绑定 `ASC` 上常用的变化委托。唯一需要注意的是，它们必须被手动销毁（比如在控件被销毁时），否则会永远驻留在内存中。示例项目包含三个蓝图 AsyncTask。

监听 `Attribute` 的变化：

![监听属性变化的蓝图节点](https://github.com/tranek/GASDocumentation/raw/master/Images/attributeschange.png)

监听冷却的变化：

![监听冷却变化的蓝图节点](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

监听 `GE` 堆叠层数的变化：

![监听 GameplayEffect 堆叠层数变化的蓝图节点](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ 返回目录](#table-of-contents)**

<a name="troubleshooting"></a>
## 9. 疑难排查

<a name="troubleshooting-notlocal"></a>
### 9.1 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`
你需要[在客户端上初始化 `ASC`](#concepts-asc-setup)。

**[⬆ 返回目录](#table-of-contents)**

<a name="troubleshooting-scriptstructcache"></a>
### 9.2 `ScriptStructCache` 错误
你需要调用 [`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata)。

**[⬆ 返回目录](#table-of-contents)**

<a name="troubleshooting-replicatinganimmontages"></a>
### 9.3 动画蒙太奇没有复制到客户端
确保你在 [GameplayAbilities](#concepts-ga) 中使用的是 `PlayMontageAndWait` 蓝图节点，而不是 `PlayMontage`。这个 [AbilityTask](#concepts-at) 会通过 `ASC` 自动复制蒙太奇，而 `PlayMontage` 节点不会。

**[⬆ 返回目录](#table-of-contents)**

<a name="troubleshooting-duplicatingblueprintactors"></a>
### 9.4 对蓝图 Actor 执行 Duplicate（创建副本）会把 AttributeSet 置为 nullptr
虚幻引擎中有一个 [bug](https://issues.unrealengine.com/issue/UE-81109)：对于从现有蓝图 Actor 类 Duplicate（创建副本）出来的蓝图 Actor 类，类上的 `AttributeSet` 指针会被置为 nullptr。有几种变通方法。我成功的做法是：不在类上创建专门的 `AttributeSet` 指针（.h 中不声明指针，构造函数中也不调用 `CreateDefaultSubobject`），而是直接在 `PostInitializeComponents()` 中把 `AttributeSets` 添加到 `ASC` 上（示例项目中没有展示）。被复制的 `AttributeSets` 仍然会存放在 `ASC` 的 `SpawnedAttributes` 数组中。代码大致如下：

```c++
void AGDPlayerState::PostInitializeComponents()
{
	Super::PostInitializeComponents();

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->AddSet<UGDAttributeSetBase>();
		// ... any other AttributeSets that you may have
		// ……以及你可能拥有的其他 AttributeSet
	}
}
```

在这种情况下，你需要使用 `ASC` 上的函数来读取和设置 `AttributeSet` 中的值，而不是[调用由宏生成的 `AttributeSet` 函数](#concepts-as-attributes)。

```c++
/** Returns current (final) value of an attribute */
/** 返回属性的当前（最终）值 */
float GetNumericAttribute(const FGameplayAttribute &Attribute) const;

/** Sets the base value of an attribute. Existing active modifiers are NOT cleared and will act upon the new base value. */
/** 设置属性的基础值。现有的激活修改器不会被清除，并会作用于新的基础值。 */
void SetNumericAttributeBase(const FGameplayAttribute &Attribute, float NewBaseValue);
```

所以 `GetHealth()` 大致如下：

```c++
float AGDPlayerState::GetHealth() const
{
	if (AbilitySystemComponent)
	{
		return AbilitySystemComponent->GetNumericAttribute(UGDAttributeSetBase::GetHealthAttribute());
	}

	return 0.0f;
}
```

设置（初始化）生命值 `Attribute` 大致如下：

```c++
const float NewHealth = 100.0f;
if (AbilitySystemComponent)
{
	AbilitySystemComponent->SetNumericAttributeBase(UGDAttributeSetBase::GetHealthAttribute(), NewHealth);
}
```

再提醒一次，`ASC` 只预期每个 `AttributeSet` 类最多有一个 `AttributeSet` 对象。

**[⬆ 返回目录](#table-of-contents)**

<a name="troubleshooting-unresolvedexternalsymbolmarkpropertydirty"></a>
### 9.5 无法解析的外部符号 UEPushModelPrivate::MarkPropertyDirty(int,int)

如果你遇到类似下面的编译错误：

```
error LNK2019: unresolved external symbol "__declspec(dllimport) void __cdecl UEPushModelPrivate::MarkPropertyDirty(int,int)" (__imp_?MarkPropertyDirty@UEPushModelPrivate@@YAXHH@Z) referenced in function "public: void __cdecl FFastArraySerializer::IncrementArrayReplicationKey(void)" (?IncrementArrayReplicationKey@FFastArraySerializer@@QEAAXXZ)
```

这是因为在 `FFastArraySerializer` 上调用了 `MarkItemDirty()`。我在更新 `ActiveGameplayEffect`（比如更新冷却时长）时遇到过这个问题。

```c++
ActiveGameplayEffects.MarkItemDirty(*AGE);
```

原因是 `WITH_PUSH_MODEL` 在多个地方被定义了。`PushModelMacros.h` 把它定义为 0，而在多个其他地方它被定义为 1。`PushModel.h` 看到的是 1，而 `PushModel.cpp` 看到的是 0。

解决方法是在 `Build.cs` 中把 `NetCore` 添加到项目的 `PublicDependencyModuleNames` 中。

**[⬆ 返回目录](#table-of-contents)**

<a name="troubleshooting-enumnamesarenowpathnames"></a>
### 9.6 枚举名现在以路径名表示

如果你遇到类似下面的编译警告：

```
warning C4996: 'FGameplayAbilityInputBinds::FGameplayAbilityInputBinds': Enum names are now represented by path names. Please use a version of FGameplayAbilityInputBinds constructor that accepts FTopLevelAssetPath. Please update your code to the new API before upgrading to the next release, otherwise your project will no longer compile.
```

UE 5.1 弃用了在 `BindAbilityActivationToInputComponent()` 的构造函数中使用 `FString` 的方式，现在必须传入一个 `FTopLevelAssetPath`。

旧的、已弃用的写法：
```c++
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), FString("EGDAbilityInputID"), static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

新的写法：
```c++
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), AbilityEnumAssetPath, static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

更多信息请参阅 `Engine\Source\Runtime\CoreUObject\Public\UObject\TopLevelAssetPath.h`。

**[⬆ 返回目录](#table-of-contents)**

<a name="acronyms"></a>
## 10. 常见 GAS 缩写

| 名称                                                                                                   | 缩写                |
|------------------------------------------------------------------------------------------------------- | ------------------- |
| AbilitySystemComponent                                                                                 | ASC                 |
| AbilityTask                                                                                            | AT                  |
| [Epic 的 Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)       | ARPG, ARPG Sample   |
| CharacterMovementComponent                                                                             | CMC                 |
| GameplayAbility                                                                                        | GA                  |
| GameplayAbilitySystem                                                                                  | GAS                 |
| GameplayCue                                                                                            | GC                  |
| GameplayEffect                                                                                         | GE                  |
| GameplayEffectExecutionCalculation                                                                     | ExecCalc, Execution |
| GameplayTag                                                                                            | Tag, GT             |
| ModifierMagnitudeCalculation                                                                           | ModMagCalc, MMC     |

**[⬆ 返回目录](#table-of-contents)**

<a name="resources"></a>
## 11. 其他资源
* [官方文档](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)
* 源码！
   * 尤其是 `GameplayPrediction.h`
* [Epic 的 Lyra 示例项目](https://unrealengine.com/marketplace/en-US/learn/lyra)
* [Epic 的 Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)
* [Unreal Slackers Discord](https://unrealslackers.org/) 有一个专门讨论 GAS 的文字频道 `#gameplay-ability-system`
   * 记得查看置顶消息
* [Dan 'Pan' 整理的资源 GitHub 仓库](https://github.com/Pantong51/GASContent)
* [SabreDartStudios 的 YouTube 视频](https://www.youtube.com/channel/UCCFUhQ6xQyjXDZ_d6X_H_-A)

<a name="resources-daveratti"></a>
### 11.1 Epic Games 的 Dave Ratti 问答

<a name="resources-daveratti-community1"></a>
#### 11.1.1 社区问题（一）
[Dave Ratti 对 Unreal Slackers Discord 服务器社区关于 GAS 问题的回答](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)：

1. 如何在 `GameplayAbilities` 之外、或与之无关的情况下，按需创建作用域预测窗口？例如，一个“发射后不管”的投射物命中敌人时，如何在本地预测一个伤害 `GameplayEffect`？

> PredictionKey 系统其实并不是为这种用途设计的。从根本上说，这套系统的工作方式是：客户端发起一个预测性动作，用一个键（key）把它告知服务器，然后客户端和服务器都运行同样的东西，并把预测产生的副作用与这个预测键关联起来。例如，“我正在预测性地激活一个能力”，或者“我已经产生了目标数据，接下来要预测性地运行能力图表中 WaitTargetData 任务之后的部分”。
>
> 在这种模式下，PredictionKey 会从服务器“弹回”，通过 UAbilitySystemComponent::ReplicatedPredictionKeyMap（一个被复制的属性）回到客户端。一旦这个键从服务器复制回来，客户端就可以撤销所有本地预测产生的副作用（GameplayCue、GameplayEffect）：被复制过来的版本*应当已经在那里了*，如果没有，那就是预测错误。在这里，准确把握撤销预测副作用的时机至关重要：撤销得太早，你会看到空档；撤销得太晚，你会看到“双份”。（注意，这里指的是有状态的预测，比如基于持续时间的 GameplayEffect 上的循环 GameplayCue。“爆发式”的 GameplayCue 和瞬时 GameplayEffect 永远不会被“撤销”或回滚。如果它们关联了预测键，客户端只是会跳过它们。）
>
> 再强调一下要点：关键在于，预测性动作必须是服务器不会自己主动去做、只有在客户端告诉它之后才会做的事情。所以，一个通用的“按需创建一个键并告诉服务器，好让我运行某个东西”的机制是行不通的，除非那个“东西”是服务器只有在客户端告诉它之后才会做的事情。
>
> 回到最初的问题：像“发射后不管”的投射物这类东西。Paragon 和 Fortnite 都有使用 GameplayCue 的投射物 Actor 类。但我们并没有用预测键系统来做这些。我们用的是“非复制 GameplayCue”的概念——这些 GameplayCue 只在本地触发，服务器会完全跳过它们。本质上，它们都是对 UGameplayCueManager::HandleGameplayCue 的直接调用。它们不经过 UAbilitySystemComponent，所以不会进行预测键检查/提前返回。
>
> 非复制 GameplayCue 的缺点在于——嗯，它们不会被复制。所以要由投射物类/蓝图来确保调用这些函数的代码路径在每一端都会运行。我们为启动（在 BeginPlay 中调用）、爆炸、撞墙/撞到角色等情况都准备了 Cue。
>
> 这类事件本来就是在客户端生成的，所以调用非复制的 GameplayCue 并不是什么大问题。复杂的蓝图可能会比较棘手，需要作者自己确保清楚什么东西在哪一端运行。

2. 在本地预测的 `GameplayAbility` 中使用带 `OnlyServerWait` 的 `WaitNetSync` `AbilityTask` 创建作用域预测窗口时，由于服务器在等待带有预测键的 RPC，玩家是否可能通过延迟发往服务器的数据包来控制 `GameplayAbility` 的时机，从而作弊？这在 Paragon 或 Fortnite 中出现过问题吗？如果出现过，Epic 是怎么解决的？

> 是的，这是一个合理的担忧。任何在服务器上运行、等待客户端“信号”的能力蓝图，都可能受到“延迟开关（lag switch）”一类漏洞的攻击。
>
> Paragon 有一个类似 UAbilityTask_WaitTargetData 的自定义目标选择任务。在这个任务中，对于即时目标选择模式，我们设置了超时，也就是等待客户端的“最大延迟”。如果目标选择模式是在等待用户确认（按下按键），那么就会忽略这个限制，因为用户可以慢慢来。但对于那些即时确认目标的能力，我们只会等待一定的时间，之后要么 A) 在服务器端生成目标数据，要么 B) 取消该能力。
>
> 我们从来没有为 WaitNetSync 加过这样的机制，我们对它的使用也相当克制。
>
> 不过我不认为 Fortnite 用了类似的东西。Fortnite 中的武器能力做了特殊处理，被批处理成一个 Fortnite 专用的 RPC：一次 RPC 就完成激活能力、提供目标数据和结束能力。所以在大逃杀中，武器能力天生就不会受到这种攻击。
>
> 我的看法是，这个问题也许可以在系统层面解决，但我不认为我们自己会在近期做出这个改动。针对你提到的情况，给 WaitNetSync 单独加上最大延迟，可能是一个合理的任务，但还是那句话——我们不太可能在近期内自己去做。


3. Paragon 和 Fortnite 使用的是哪种 `EGameplayEffectReplicationMode`？对于何时使用哪种模式，Epic 有什么建议？

> 两款游戏基本上都是对玩家控制的角色使用 Mixed 模式，对 AI 控制的角色（AI 小兵、野怪、AI 僵尸等）使用 Minimal 模式。对于在多人游戏中使用这套系统的大多数人，我也推荐这么做。在项目中越早设置好这些越好。
>
> Fortnite 在优化上又更进了几步。对于模拟代理，它实际上根本不复制 UAbilitySystemComponent。在 Fortnite 拥有者的 PlayerState 类的 ::ReplicateSubobjects() 中，会跳过该组件和属性子对象。我们会把能力系统组件中最少量的需要复制的数据推送到 Pawn 自身上的一个结构体中（基本上是属性值的一个子集，以及一个白名单标签子集，我们用位掩码把它们复制下去）。我们称之为“代理（proxy）”。在接收端，我们取出复制到 Pawn 上的代理数据，再把它推回 PlayerState 上的能力系统组件中。所以在 FNBR 中，每个玩家确实都有一个 ASC，只是它不直接复制：而是通过 Pawn 上一个最小化的代理结构体复制数据，再在接收端路由回 ASC。这样做的好处是：A) 数据集更精简；B) 可以利用 Pawn 的相关性。
>
> 我不确定在那之后又做了其他服务器端优化（Replication Graph 等）的情况下，这样做是否仍有必要，而且这也不是最易于维护的模式。


4. 既然按照 `GameplayPrediction.h` 的说法，我们无法预测 `GameplayEffects` 的移除，那么有没有什么策略可以减轻移除 `GameplayEffects` 时延迟带来的影响？例如，在移除一个移动速度减速效果时，我们目前必须等待服务器复制 `GameplayEffect` 的移除，导致玩家角色的位置发生突跳（snap）。

> 这个问题很棘手，我没有很好的答案。我们通常是用容差和平滑处理来绕开这些问题。我完全同意，能力系统与角色移动系统之间的精确同步目前做得并不好，这是我们确实想要修复的问题。
>
> 我曾经有一个允许预测性移除 GE 的搁置改动（shelf），但在不得不转去做别的事情之前，始终没能解决所有边界情况。不过这也解决不了所有问题，因为角色移动内部还有一个保存移动（saved move）的缓冲区，它对能力系统以及可能存在的移动速度修改器等一无所知。即便不考虑无法预测 GE 移除的问题，仍然有可能陷入校正的反馈循环。
>
> 如果你认为自己的情况确实非常迫切，可以预测性地添加一个 GE，用它来抑制你的移动速度 GE。我自己从来没这么做过，但以前在理论上设想过。它也许能帮助解决某一类问题。


5. 我们知道，在 Paragon 和 Fortnite 中 `AbilitySystemComponent` 位于 `PlayerState` 上，而在 Action RPG 示例中它位于 `Character` 上。对于 AbilitySystemComponent 应该放在哪里、它的 `Owner` 应该是什么，Epic 内部有什么规则、准则或建议？

> 总的来说，我认为任何不需要重生的东西，其 Owner 和 Avatar Actor 都应该是同一个。比如 AI 敌人、建筑、世界中的道具等等。
>
> 任何会重生的东西，其 Owner 和 Avatar 都应该不同，这样能力系统组件就不需要在重生后被保存/重建/恢复。PlayerState 是一个合乎逻辑的选择，因为它会复制到所有客户端（而 PlayerController 不会）。缺点是 PlayerState 始终是相关的，所以在 100 人的游戏中可能会遇到问题（参见问题 #3 中关于 FN 做法的说明）。


6. 能否让多个 `AbilitySystemComponents` 拥有相同的 Owner、但拥有不同的 Avatar（例如分别挂在 Pawn 和武器/物品/投射物上，`Owner` 都设为 `PlayerState`）？

> 我看到的第一个问题是如何在拥有者 Actor 上实现 IGameplayTagAssetInterface 和 IAbilitySystemInterface。前者也许可行：把所有 ASC 的标签聚合起来即可（但要小心——HasAllMatchingGameplayTags 可能只有在跨 ASC 聚合后才能满足。仅仅把调用转发给每个 ASC，再把结果做 OR 运算是不够的）。但后者更加棘手：哪一个 ASC 才是权威的那个？如果有人想施加一个 GE，应该由哪一个来接收？也许你能把这些问题解决掉，但这一方面会是整个问题中最难的部分：一个 Owner 下面会挂着多个 ASC。
>
> 不过，单就 Pawn 和武器上各有一个独立的 ASC 而言，这本身可以说得通。例如，用来区分描述武器的标签和描述拥有者 Pawn 的标签。也许让授予武器的标签同时“作用于”拥有者、而其他东西不共享，是合理的（例如属性和 GE 相互独立，但拥有者会像我上面说的那样聚合其拥有的标签）。我相信这是可以实现的。但让多个 ASC 拥有同一个 Owner，可能会变得很棘手。


7. 有没有办法阻止服务器在拥有者客户端上覆盖本地预测能力的冷却时长？在高延迟的情况下，这样可以让拥有者客户端在本地冷却结束、但服务器上仍在冷却时“尝试”再次激活能力。等拥有者客户端的激活请求通过网络到达服务器时，服务器上的冷却可能已经结束，或者服务器可以把这个激活请求排队，等待剩余的几毫秒。否则按照现状，高延迟客户端能够再次激活能力之前的等待时间，会比低延迟客户端更长。这一点在冷却极短的能力上最为明显，比如冷却时间不到一秒的普通攻击。如果没有办法阻止服务器覆盖本地预测能力的冷却时长，那么 Epic 采用什么策略来减轻高延迟对再次激活能力的影响？换个举例的方式来问：Epic 是如何设计 Paragon 的普通攻击和其他能力，让高延迟玩家借助本地预测，能够以与低延迟玩家相同的速度进行攻击或激活能力的？

> 简短的回答是：没有办法阻止这一点，而 Paragon 也确实存在这个问题。延迟更高的连接，普通攻击的射速（ROF）会更低。
>
> 我曾尝试通过加入“GE 校正（GE reconciliation）”来修复这个问题，在计算 GE 持续时间时把延迟考虑进去。本质上就是允许服务器“吃掉”一部分 GE 总时长，使得无论延迟多少，GE 在客户端上的有效时长都能 100% 一致（尽管延迟波动仍可能导致问题）。但我始终没能把它做到可以发布的状态，而项目推进得很快，我们最终也没有彻底解决这个问题。
>
> Fortnite 对武器射速自己做了计数逻辑：它不用 GE 来实现武器的冷却。如果这对你的游戏来说是个关键问题，我推荐这样做。


8. Epic 对 GameplayAbilitySystem 插件有什么路线图？Epic 计划在 2019 年及以后加入哪些功能？

> 我们认为，总体来说这套系统目前已经相当稳定，现在也没有人在开发重大的新功能。偶尔会为 Fortnite 或根据 UDN/Pull Request 做一些 bug 修复和小改进，但目前也就仅此而已。
>
> 从更长远来看，我认为我们最终会做一个“V2”或者进行一些大的改动。编写这套系统让我们学到了很多，我们觉得有很多地方做对了，也有很多地方做错了。我很希望有机会纠正那些错误，并改进上面指出的一些致命缺陷。
>
> 如果真的有 V2，提供升级路径将是重中之重。我们绝不会做一个 V2，然后让 Fortnite 永远停留在 V1 上：一定会有某种路径或流程，尽可能多地自动迁移，尽管几乎肯定仍需要一些手动重做。
>
> 高优先级的修复会是：
> * 与角色移动系统更好的互通性。统一客户端预测。
> * GE 移除的预测（问题 #4）
> * GE 的延迟校正（问题 #7）
> * 通用化的网络优化，比如批处理 RPC 和代理结构体。主要是我们为 Fortnite 做过的那些东西，但要设法把它们拆解成更通用的形式，至少让各个游戏能更容易地编写自己特定于游戏的优化。
>
> 我会考虑做的、更偏向通用重构类型的改动：
> * 我想从根本上摆脱让 GE 直接引用电子表格数值的做法，改为让 GE 发出参数，再由某个绑定到电子表格数值的更高层对象来填充这些参数。当前模型的问题在于，GE 与曲线表格行紧密耦合，导致它们无法共享。我认为可以编写一个通用的参数化系统，作为 V2 系统的基础。
> * 减少 UGameplayAbility 上“策略（policy）”的数量。我会移除 ReplicationPolicy 和 InstancingPolicy。在我看来，复制几乎从来都不是真正需要的，而且会造成困惑。InstancingPolicy 应当被替换掉，做法是把 FGameplayAbilitySpec 变成一个可以被子类化的 UObject。它本该就是那个“非实例化的能力对象”，带有事件并且可以用蓝图编写。UGameplayAbility 则应当是那个“每次执行一个实例”的对象。如果你确实需要实例化，它可以是可选的：而“非实例化”的能力则通过新的 UGameplayAbilitySpec 对象来实现。
> * 系统应当提供更多“中间层”的构件，比如“带过滤的 GE 施加容器”（用数据驱动的方式，结合更高层的游戏逻辑，决定把哪些 GE 施加到哪些 Actor 上）、“重叠体积支持”（基于碰撞图元的重叠事件来应用“带过滤的 GE 施加容器”）等等。这些都是每个项目最终都会用自己的方式去实现的基础构件。要把它们做对并不容易，所以我认为我们应该更好地提供一些基础实现。
> * 总的来说，减少让项目跑起来所需的样板代码。也许可以做一个单独的模块，叫“Ex library”之类的，开箱即提供被动能力或基础即时命中武器之类的东西。这个模块是可选的，但能让你快速上手。
> * 我想把 GameplayCue 移到一个单独的、不与能力系统耦合的模块中。我认为这方面有很多可以改进的地方。


> 这只是我的个人观点，不代表任何人的承诺。我认为最现实的做法是：随着新的引擎技术计划陆续推出，能力系统也需要随之更新，那时就是做这类事情的时机。这些计划可能与脚本、网络或物理/角色移动有关。不过这些都是非常长远的展望，所以我无法给出任何承诺或时间上的估计。

**[⬆ 返回目录](#table-of-contents)**

<a name="resources-daveratti-community2"></a>
#### 11.1.2 社区问题（二）
社区成员 [iniside](https://github.com/iniside) 与 Dave Ratti 的问答：

1. 是否计划支持解耦的固定 Tick？我希望游戏线程（Game Thread）以固定频率运行（比如 30/60fps），而让渲染线程放开跑。我之所以问这是不是将来可以期待的东西，是为了对游戏玩法应该如何运作做一些假设。我这么问主要是因为现在物理已经有了固定频率的异步 Tick，这就引出了一个问题：系统的其余部分将来会如何运作。我不掩饰地说，如果能让游戏线程以固定频率 Tick，而不用同时固定引擎其余部分的 Tick 频率，那简直太棒了。

> 目前没有计划把渲染帧率和游戏线程的 Tick 帧率解耦。考虑到这些系统的复杂性，以及需要保持与之前引擎版本的向后兼容，我认为这件事已经不可能发生了。
>
> 我们转而采取的方向是：提供一个异步的“物理线程（Physics Thread）”，它以固定的 Tick 频率运行，独立于游戏线程。需要以固定频率运行的东西可以放在这里运行，而游戏线程/渲染可以照旧按原来的方式运作。
>
> 值得说明的是，Network Prediction 支持它所谓的独立 Tick（Independent Ticking）和固定 Tick（Fixed Ticking）两种模式。我的长期计划是：让独立 Tick 大体保持 Network Prediction 中现在的样子，即在游戏线程上以可变帧率运行，并且没有“组/世界”级别的预测，就是经典的“客户端预测自己的 Pawn 和自己拥有的 Actor”模型。而固定 Tick 则会使用异步物理那一套，允许你预测那些不由客户端控制/拥有的 Actor，比如物理对象以及其他客户端/Pawn/载具等。


2. 对于 Network Prediction 与能力系统的集成会是什么样子，有没有什么计划？比如基于固定帧的能力激活（让服务器拿到的是能力被激活、任务被执行时所在的帧，而不是预测键）？

> 有的，计划是重写/移除能力系统的预测键，用 Network Prediction 的构件来取代它们。NetworkPredictionExtras 中的 MockAbility 示例展示了这可能会如何工作，不过它们比 GAS 实际需要的要“写死”得多。
>
> 主要思路是：移除 ASC 的 RPC 中显式的客户端->服务器预测键交换。不再有预测窗口或作用域预测键，取而代之的是一切都以 NetworkPrediction 的帧为锚点。重要的是客户端和服务器对事情发生的时间达成一致。例如：
>
> * 能力何时被激活/结束/取消
> * GameplayEffect 何时被施加/移除
> * 属性值（某个属性在第 X 帧时的值是多少）
>
> 我认为这可以在能力系统层面以通用的方式实现。但要让 UGameplayAbility 内部由用户定义的逻辑完全可回滚，仍然需要更多工作。我们最终可能会有一个 UGameplayAbility 的子类，它完全可回滚，但只能使用一组更有限的功能，或者只能使用被标记为对回滚友好的 AbilityTask。大概是这样。此外，这对动画事件、根运动以及它们的处理方式也有很多影响。
>
> 真希望我能给出更明确的答案，但在再次改动 GAS 之前，把基础打好真的非常重要。移动和物理必须先稳固下来，才能去改动更高层的系统。


3. 是否计划把 Network Prediction 的开发转移到主分支上？说实话，我真的很想看看最新的代码，不管它处于什么状态。

> 我们正在朝这个方向努力。系统层面的工作仍然都在 NetworkPrediction 中进行（参见 NetworkPhysics.h），底层的异步物理部分应该都已经可以获取（RewindData.h 等）。但我们也有一些一直在专注处理的 Fortnite 用例，这些显然无法公开。我们正在处理 bug、进行性能优化等等。
>
> 补充一些背景：在开发这套系统的早期版本时，我们非常专注于“前端”的东西——状态和模拟是如何定义和编写的。我们在那方面学到了很多。但随着异步物理部分陆续上线，我们更专注于让这套系统真正跑起一些实际的东西，代价是抛弃了我们早期的一些抽象。我们的目标是，等实际的东西跑起来之后，再回过头来把各部分重新统一起来。例如，回到“前端”，在我们现在正在开发的核心技术之上，做出它的最终版本。


4. 有一段时间主分支上有一个用于发送 Gameplay Message 的插件（看起来像是事件/消息总线），但后来被移除了。有计划恢复它吗？有了 Game Features/Modular Gameplay 插件之后，有一个通用的事件总线分发器会非常有用。

> 我想你指的是 GameplayMessages 插件。它很可能会在某个时候回来——它的 API 还没有真正定型，作者本来也没打算这么早公开它。我同意它对模块化的玩法设计应该很有用。但这不是我负责的领域，所以我没有更多信息了。


5. 我最近在试用异步固定物理，结果很有前景。不过如果将来 NP 会有更新，我可能就先随便玩玩、等着吧，因为要让它跑起来，我仍然需要让整个引擎以固定频率 Tick；另一方面，我又想让物理保持在 33ms。如果所有东西都是 30 fps，体验可不怎么样 (:。

我注意到有一些关于异步 CharacterMovementComponent 的工作，但不确定它是会使用 Network Prediction，还是一项独立的工作？

注意到这一点后，我也顺手尝试以固定的 Tick 频率实现自己的自定义异步移动。效果还行，但在此基础上，我还需要额外加一个用于插值的更新。我的做法是：在单独的工作线程上以固定的 33ms 更新频率运行模拟 Tick，进行计算、保存结果，然后在游戏线程上对结果进行插值，以匹配当前帧率。不算完美，但能完成任务。

我的问题是：将来这会不会变得更容易配置？因为现在要写相当多的样板代码（插值那部分），而且逐个对每个移动的物体进行插值，效率也不是特别高。

异步这一套真的很有意思，因为它能让你真正以固定的更新频率运行游戏模拟（这样就不再需要固定线程了），并得到更可预测的结果。这是今后的发展方向，还是更多只是让部分特定系统受益？据我所知，Actor 的变换（transform）不是异步更新的，蓝图也不完全是线程安全的。换句话说，这是计划在框架层面提供更多支持的东西，还是需要每个游戏自己去解决的东西？

> 异步 CharacterMovementComponent
>
> 这基本上是一个早期原型/实验，把 CMC 原样移植到物理线程上。我目前还不把它看作 CMC 的未来，但它有可能演变成那样。目前它还没有网络支持，所以我不建议你真的去跟进它。做这件事的人主要关心的是测量这套系统会增加多少输入延迟，以及如何减轻这种延迟。
>
> 我仍然需要让整个引擎以固定频率 Tick；另一方面，我又想让物理保持在 33ms。如果所有东西都是 30 fps，体验可不怎么样 (:。
>
> 异步这一套真的很有意思，因为它能让你真正以固定的更新频率运行游戏模拟（这样就不再需要固定线程了）
>
> 是的。这里的目标是：启用异步物理后，你可以让引擎以可变的 Tick 频率运行，而物理和“核心”游戏模拟（比如角色移动、载具、GAS 等）则可以以固定频率运行。
>
> 目前要启用它，需要设置这些控制台变量：（我想你已经弄明白了）  
> `p.DefaultAsyncDt=0.03333`  
> `p.RewindCaptureNumFrames=64`
>
> Chaos 确实为物理状态提供了插值（例如被推回 UPrimitiveComponent、对游戏代码可见的那些变换）。现在有一个控制台变量 `p.AsyncInterpolationMultiplier` 可以控制它，如果你想研究一下的话。你应该能看到物理刚体平滑连续的运动，而无需编写任何额外代码。
>
> 如果你想对非物理状态进行插值，目前仍然需要你自己来做。举个例子：你想在异步物理线程上更新（Tick）一个冷却，但希望在游戏线程上看到平滑连续的插值，这样每一个渲染帧都会更新冷却的可视化效果。我们最终会处理这个问题，但目前还没有示例。
>
> 现在要写相当多的样板代码，
>
> 是的，到目前为止，这一直是这套系统普遍存在的一个大问题。我们想提供一个接口，让有经验的程序员可以用它来最大化性能和安全性（也就是能够编写“开箱即用”地支持预测的游戏代码，而不用面对大量隐患，以及那些“可以做但最好别做”的事情）。所以像 CharacterMovement 这样的东西可能会做很多自定义的事情来最大化性能——例如编写模板化代码并进行批量更新、并行化处理（going wide）、把更新循环拆分成不同的阶段等等。对于这种用例，我们想为异步线程和回滚系统提供一个良好的“底层”接口。而且在这种情况下——角色移动系统本身以它自己的方式提供可扩展性，仍然是合理的。例如，提供一种用蓝图编写自定义移动模式的方式，并提供线程安全的蓝图 API。
>
> 但我们也认识到，对于那些并不真正需要自己一套“系统”的较简单的游戏对象来说，这是不可接受的。我们需要的是更符合虚幻风格的东西。例如，使用反射系统、提供通用的蓝图支持等等。已经有一些在其他线程上使用蓝图的例子（参见 BlueprintThreadSafe 关键字，以及动画系统一直在努力的方向）。所以我认为总有一天会出现某种形式的此类支持。但还是那句话，我们还没走到那一步。
>
> 我知道你问的只是插值，但这就是总体上的回答：目前我们要求你手动完成所有事情，比如 NetSerialize、ShouldReconcile、Interpolate 等，但最终我们会提供一种方式，类似于“如果你只想使用反射系统，就不必手动编写这些东西”。我们只是不想*强迫*所有人都使用反射系统，因为那会带来其他一些限制，而我们认为不应该在系统的最底层承担这些限制。
>
> 最后再回到我前面说的——现在我们真的是在专注于让几个非常具体的示例跑起来并达到良好的性能，之后我们会把注意力转回前端，让东西变得易于使用和迭代，减少样板代码等等，方便其他所有人使用。

**[⬆ 返回目录](#table-of-contents)**

<a name="changelog"></a>
## 12. GAS 更新日志

这里列出了 GAS 值得关注的变化（修复、变更和新功能），整理自虚幻引擎官方的升级更新日志，以及我遇到过的、未写入文档的变化。如果你发现了这里没有列出的内容，欢迎提交 issue 或 pull request。

<a name="changelog-5.3"></a>
### 5.3
* 崩溃修复：修复了无缝旅行（seamless travel）后尝试施加 GameplayCue 时的崩溃。
* 崩溃修复：修复了使用 Live Coding 时由 GlobalAbilityTaskCount 引起的崩溃。
* 崩溃修复：修复了 UAbilityTask::OnDestroy 在被递归调用时（例如 UAbilityTask_StartAbilityState 这类情况）会崩溃的问题。
* Bug 修复：现在可以在子类中安全地调用 `Super::ActivateAbility`。以前它会调用 `CommitAbility`。
* Bug 修复：增加了对正确复制不同类型 FGameplayEffectContext 的支持。
* Bug 修复：FGameplayEffectContextHandle 现在会在获取“Actors”之前检查数据是否有效。
* Bug 修复：Gameplay Ability System 的 TargetData LocationInfo 现在会保留旋转。
* Bug 修复：Gameplay Ability System 现在只有在找到有效的 PC 时才会停止搜索 PC。
* Bug 修复：在 RemoveGameplayCue_Internal 中，如果已存在 GameplayCueParameters，就使用它，而不是使用默认的参数对象。
* Bug 修复：GameplayAbilityWorldReticle 现在朝向来源 Actor，而不是 TargetingActor。
* Bug 修复：如果通过 GiveAbilityAndActivateOnce 传入了触发事件数据，且能力列表处于锁定状态，现在会缓存该触发事件数据。
* Bug 修复：增加了支持，让 FInheritedGameplayTags 立即更新其 CombinedTags，而不是等到保存时才更新。
* Bug 修复：把 ShouldAbilityRespondToEvent 从仅客户端的代码路径移到了服务器和客户端共同的代码路径。
* Bug 修复：修复了 FAttributeSetInitterDiscreteLevels 因曲线简化（Curve Simplification）而在 Cook 后的构建中无法工作的问题。
* Bug 修复：在 GameplayAbility 中设置 CurrentEventData。
* Bug 修复：确保在可能执行回调之前，MinimalReplicationTags 已被正确设置。
* Bug 修复：修复了实例化的 GameplayAbility 上不会调用 ShouldAbilityRespondToEvent 的问题。
* Bug 修复：禁用 gc.PendingKill 时，在子 Actor（Child Actor）上执行的 GameplayCue Notify Actor 不再泄漏内存。
* Bug 修复：修复了 GameplayCueManager 中 GameplayCueNotify_Actor 可能因哈希冲突而“丢失”的问题。
* Bug 修复：即使 Actor 上没有任何 GameplayTag，WaitGameplayTagQuery 现在也会遵循其 Query。
* Bug 修复：PostAttributeChange 和 AttributeValueChangeDelegates 现在会得到正确的 OldValue。
* Bug 修复：修复了由原生代码创建的 FGameplayTagQuery 结构体无法显示正确查询描述（Query Description）的问题。
* Bug 修复：确保在使用能力系统时会调用 UAbilitySystemGlobals::InitGlobalData。以前如果用户没有调用它，Gameplay Ability System 就无法正常工作。
* Bug 修复：修复了在 UGameplayAbility::EndAbility 中链接/取消链接动画层（anim layer）时的问题。
* Bug 修复：更新了能力系统组件的函数，使其在使用 Spec 的能力指针之前先进行检查。
* 新增：在 FGameplayTagRequirements 中添加了一个 GameplayTagQuery 字段，以便指定更复杂的要求。
* 新增：引入了 FGameplayEffectQuery::SourceAggregateTagQuery，用来扩充 SourceTagQuery。
* 新增：扩展了通过控制台命令执行和取消 GameplayAbility 与 GameplayEffect 的功能。
* 新增：增加了对 GameplayAbility 蓝图执行“审计（Audit）”的功能，会显示它们是如何开发的以及预期如何使用。
* 变更：对于每个 Actor 一个实例（instanced per Actor）的 GameplayAbility，OnAvatarSet 现在会在主实例而不是 CDO 上调用。
* 变更：允许在同一个 GameplayAbility 图表中同时使用 Activate Ability 和 Activate Ability From Event。
* 变更：AnimTask_PlayMontageAndWait 现在有一个开关，允许在 BlendOut 事件之后仍触发 Completed 和 Interrupted。
* 变更：ModMagnitudeCalc 的包装函数已被声明为 const。
* 变更：FGameplayTagQuery::Matches 现在对空查询返回 false。
* 变更：更新了 FGameplayAttribute::PostSerialize，把其中包含的属性标记为可搜索名称（searchable name）。
* 变更：更新了 GetAbilitySystemComponent，使其参数默认为 Self。
* 变更：把 AbilityTask_WaitTargetData 中的一些函数标记为 virtual。
* 变更：移除了未使用的函数 FGameplayAbilityTargetData::AddTargetDataToGameplayCueParameters。
* 变更：移除了残留的 GameplayAbility::SetMovementSyncPoint。
* 变更：从 GameplayTask 和能力系统组件中移除了未使用的复制标志。
* 变更：把部分 GameplayEffect 功能移到了可选的组件中。所有现有内容在必要时会在 PostCDOCompiled 期间自动更新为使用组件。

https://docs.unrealengine.com/5.3/en-US/unreal-engine-5.3-release-notes/

<a name="changelog-5.2"></a>
### 5.2
* Bug 修复：修复了 `UAbilitySystemBlueprintLibrary::MakeSpecHandle` 函数中的一个崩溃。
* Bug 修复：修复了 Gameplay Ability System 中的一处逻辑错误：一个未被控制的 Pawn 即使是在服务器本地生成的（例如载具），也会被视为远程的。
* Bug 修复：对于被服务器拒绝的、预测的实例化能力，现在会正确设置激活信息。
* Bug 修复：修复了一个会导致 GameplayCue 卡在远程实例上的 bug。
* Bug 修复：修复了链式调用 WaitGameplayEvent 时的内存踩踏（memory stomp）问题。
* Bug 修复：在蓝图中调用 AbilitySystemComponent 的 `GetOwnedGameplayTags()` 函数时，如果同一节点被多次执行，不再保留上一次调用的返回值。
* Bug 修复：修复了 GameplayEffectContext 复制一个永远不会被复制的动态对象引用的问题。
  * 这个问题会导致 GameplayEffect 无法调用 `Owner->HandleDeferredGameplayCues(this)`，因为 `bHasMoreUnmappedReferences` 总是为 true。
* 新增：[Gameplay Targeting System](https://docs.unrealengine.com/en-US/gameplay-targeting-system-in-unreal-engine/)，一种创建数据驱动的目标选择请求的方式。
* 新增：为 GameplayTag Query 增加了自定义序列化支持。
* 新增：增加了对复制 FGameplayEffectContext 派生类型的支持。
* 新增：资产中的 Gameplay Attribute 现在会在保存时被注册为可搜索名称，因此可以在引用查看器中看到对属性的引用。
* 新增：为 AbilitySystemComponent 添加了一些基础的单元测试。
* 新增：Gameplay Ability System 的属性现在遵循 Core Redirects。这意味着你现在可以在代码中重命名 AttributeSet 及其属性，只需在 DefaultEngine.ini 中添加重定向条目，用旧名称保存的资产就能正确加载。
* 变更：允许在代码中修改 GameplayEffect 修改器的求值通道（evaluation channel）。
* 变更：从 Gameplay Abilities 插件中移除了之前未使用的变量 `FGameplayModifierInfo::Magnitude`。
* 变更：移除了能力系统组件与 Smart Object 实例标签之间的同步逻辑。

https://docs.unrealengine.com/5.2/en-US/unreal-engine-5.2-release-notes/

<a name="changelog-5.1"></a>
### 5.1
* Bug 修复：修复了被复制的松散 GameplayTag 不会复制给拥有者的问题。
* Bug 修复：修复了 AbilityTask 的一个 bug，该 bug 可能导致能力无法被及时垃圾回收。
* Bug 修复：修复了一个监听某个标签来激活的 GameplayAbility 无法被激活的问题。当有多个 GameplayAbility 监听这个标签，而列表中的第一个无效或没有激活权限时，就会出现这种情况。
* Bug 修复：修复了正确使用 Data Registry 的 GameplayEffect 在加载时仍发出警告的问题，并改进了警告文本。
* Bug 修复：移除了 UGameplayAbility 中的一段代码，该代码错误地只把最后一个实例化的能力注册到蓝图调试器中用于断点。
* Bug 修复：修复了在 ApplyGameplayEffectSpecToTarget 内部加锁期间调用 EndAbility 时，Gameplay Ability System 的能力会卡住的问题。
* 新增：增加了让 GameplayEffect 添加阻止能力标签（blocked ability tags）的支持。
* 新增：添加了 WaitGameplayTagQuery 节点。一个基于 UAbilityTask，另一个基于 UAbilityAsync。该节点指定一个 TagQuery，并根据配置在查询结果变为 true 或 false 时触发其输出引脚。
* 新增：修改了控制台变量中的 AbilityTask 调试，在非 Shipping 构建中默认开启调试录制和日志打印（并可根据需要通过热修复开启/关闭）。
* 新增：现在可以把 AbilitySystem.AbilityTask.Debug.RecordingEnabled 设为 0 来禁用，设为 1 在非 Shipping 构建中启用，设为 2 在所有构建（包括 Shipping）中启用。
* 新增：可以使用 AbilitySystem.AbilityTask.Debug.AbilityTaskDebugPrintTopNResults 只在日志中打印前 N 个结果（避免刷屏）。
* 新增：可以使用 STAT_AbilityTaskDebugRecording 测试这些默认开启的调试改动对性能的影响。
* 新增：添加了一个用于过滤 GameplayCue 事件的调试命令。
* 新增：为 Gameplay Ability System 添加了新的调试命令 AbilitySystem.DebugAbilityTags、AbilitySystem.DebugBlockedTags 和 AbilitySystem.DebugAttribute。
* 新增：添加了一个蓝图函数，用于获取 Gameplay Attribute 的调试字符串表示。
* 新增：添加了一种新的 GameplayTask 资源重叠策略，用于取消已有的任务。
* 变更：现在 AbilityTask 应确保在对 Ability 指针完成所有必要操作之后，才调用 Super::OnDestroy，因为调用它之后该指针会被置空。
* 变更：把 FGameplayAbilitySpec/Def::SourceObject 改为弱引用。
* 变更：把 AbilityTask 中对能力系统组件的引用改为弱指针，以便垃圾回收可以删除它。
* 变更：移除了多余的枚举 EWaitGameplayTagQueryAsyncTriggerCondition。
* 变更：GameplayTasksComponent 和 AbilitySystemComponent 现在支持注册子对象（registered subobject）API。
* 变更：改进了日志，用于说明 GameplayAbility 激活失败的原因。
* 变更：移除了 AbilitySystem.Debug.NextTarget 和 PrevTarget 命令，改用全局的 HUD 命令 NextDebugTarget 和 PrevDebugTarget。

https://docs.unrealengine.com/5.1/en-US/unreal-engine-5.1-release-notes/

<a name="changelog-5.0"></a>
### 5.0

https://docs.unrealengine.com/5.0/en-US/unreal-engine-5.0-release-notes/

<a name="changelog-4.27"></a>
### 4.27
* 崩溃修复：修复了一个根运动源问题：当 Actor 执行完一个使用带“随时间变化的强度修改器”的恒定力根运动任务的能力时，联网客户端可能会崩溃。
* Bug 修复：修复了使用 GameplayCue 时编辑器加载时间的退化问题。
* Bug 修复：GameplayEffectsContainer 的 `SetActiveGameplayEffectLevel` 方法在设置相同的 EffectLevel 时，不再把 FastArray 标记为脏。
* Bug 修复：修复了 GameplayEffect Mixed 复制模式中的一个边界情况：那些并非显式由某个网络连接拥有、但通过 `GetNetConnection` 使用该连接的 Actor，收不到 Mixed 复制的更新。
* Bug 修复：修复了 GameplayAbility 类方法 `EndAbility` 中出现的无限递归，该问题是由在 `K2_OnEndAbility` 中再次调用 `EndAbility` 引起的。
* Bug 修复：如果 GameplayTag 蓝图引脚在标签注册之前就被加载，它们不再会被悄悄清空。现在它们的行为与 GameplayTag 变量相同，两者的行为都可以通过项目设置中的 ClearInvalidTags 选项来更改。
* Bug 修复：提高了 GameplayTag 操作的线程安全性。
* 新增：把 SourceObject 暴露给了 GameplayAbility 的 `K2_CanActivateAbility` 方法。
* 新增：原生 GameplayTag。引入了新的 `FNativeGameplayTag`，可以定义一次性的原生标签，它们会在模块加载和卸载时被正确地注册和注销。
* 新增：更新了 `GiveAbilityAndActivateOnce`，使其可以传入 FGameplayEventData 参数。
* 新增：改进了 GameplayAbilities 插件中的 ScalableFloat，使其支持从新的 Data Registry 系统动态查找曲线表格。添加了一个 ScalableFloat 头文件，便于在能力插件之外复用这个通用结构体。
* 新增：通过 GameplayTagsEditorModule 添加了代码支持，可以在其他编辑器自定义中使用 GameplayTag UI。
* 新增：修改了 UGameplayAbility 的 PreActivate 方法，使其可以选择性地接收触发事件数据。
* 新增：增加了更多在编辑器中使用项目特定过滤器来过滤 GameplayTag 的支持。`OnFilterGameplayTag` 会提供引用该标签的属性以及标签来源，因此你可以根据是哪个资产在请求标签来过滤标签。
* 新增：添加了一个选项，在初始化之后调用 GameplayEffectSpec 的类方法 `SetContext` 时，保留最初捕获的 SourceTags。
* 新增：改进了从特定插件注册 GameplayTag 的 UI。新的标签 UI 现在允许你为新添加的 GameplayTag 来源选择磁盘上的某个插件位置。
* 新增：Sequencer 中新增了一种轨道，可以在使用 GameplayAbilitySystem 构建的 Actor 上触发通知状态（notify state）。与通知类似，GameplayCueTrack 可以使用基于范围的事件或基于触发的事件。
* 变更：修改了 GameplayCueInterface，改为按引用传递 GameplayCueParameters 结构体。
* 优化：对 GameplayTag 表的加载和重新生成做了多项性能改进，使该选项得到优化。

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_27/

<a name="changelog-4.26"></a>
### 4.26
* GAS 插件不再被标记为 Beta。
* 崩溃修复：修复了在没有选择有效标签来源的情况下添加 GameplayTag 时的崩溃。
* 崩溃修复：在一条消息中添加了路径字符串参数，修复了 UGameplayCueManager::VerifyNotifyAssetIsInValidPath 中的崩溃。
* 崩溃修复：修复了 AbilitySystemComponent_Abilities 中未经检查就使用指针导致的访问冲突崩溃。
* Bug 修复：修复了一个 bug：对于不会在施加额外实例时重置持续时间的可堆叠 GE 存在问题。
* Bug 修复：修复了导致 CancelAllAbilities 只取消非实例化能力的问题。
* 新增：为 GameplayAbility 的提交（commit）函数添加了可选的标签参数。
* 新增：为 PlayMontageAndWait 能力任务添加了 StartTimeSeconds，并改进了注释。
* 新增：在 FGameplayAbilitySpec 中添加了标签容器“DynamicAbilityTags”。这些是可选的能力标签，会随 Spec 一起复制。它们也会被施加的 GameplayEffect 作为来源标签捕获。
* 新增：GameplayAbility 的 IsLocallyControlled 和 HasAuthority 函数现在可以从蓝图中调用。
* 新增：可视化日志（Visual Logger）现在只有在当前正在录制可视化日志数据时，才会收集并存储瞬时 GE 的信息。
* 新增：为蓝图节点上的 Gameplay Attribute 引脚增加了重定向器（redirector）支持。
* 新增：新增功能：与根运动移动相关的能力任务结束时，会把移动组件的移动模式恢复为任务开始之前的移动模式。

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_26/

<a name="changelog-4.25.1"></a>
### 4.25.1
* 已修复！UE-92787：保存带有 Get Float Attribute 节点、且属性引脚被内联设置的蓝图时崩溃
* 已修复！UE-92810：生成带有实例可编辑 GameplayTag 属性、且该属性被内联修改过的 Actor 时崩溃

<a name="changelog-4.25"></a>
### 4.25
* 修复了 `RootMotionSource` `AbilityTasks` 的预测
* [`GAMEPLAYATTRIBUTE_REPNOTIFY()`](#concepts-as-attributes) 现在还会额外接收旧的 `Attribute` 值。我们必须把它作为可选参数传给自己的 `OnRep` 函数。以前，它会读取属性值来尝试获取旧值。然而，如果是从复制函数中调用，旧值在到达 SetBaseAttributeValueFromReplication 之前就已经被丢弃了，所以我们得到的会是新值。
* 为 `UGameplayAbility` 添加了 [`NetSecurityPolicy`](#concepts-ga-netsecuritypolicy)。
* 崩溃修复：修复了在没有选择有效标签来源的情况下添加 GameplayTag 时的崩溃。
* 崩溃修复：消除了攻击者通过能力系统使服务器崩溃的几种途径。
* 崩溃修复：现在在检查标签要求之前，会先确保存在 GameplayEffect 定义。
* Bug 修复：修复了一个问题：如果 GameplayTag 类别属于函数终止节点（function terminator node）的一部分，它就不会应用到蓝图中的函数参数上。
* Bug 修复：修复了在多个视口下 GameplayEffect 的标签不会被复制的问题。
* Bug 修复：修复了一个 bug：在遍历被触发的能力时，GameplayAbilitySpec 可能会被 InternalTryActivateAbility 函数置为无效。
* Bug 修复：修改了在标签计数容器中更新 GameplayTag 的处理方式。在移除 GameplayTag 并延迟更新父标签时，现在会在父标签更新完毕之后才调用与变化相关的委托。这确保了委托广播时标签表处于一致的状态。
* Bug 修复：确认目标时，现在会先复制一份生成的目标 Actor 数组再进行遍历，因为某些回调可能会修改该数组。
* Bug 修复：修复了一个 bug：对于不会在施加额外实例时重置持续时间、且使用 SetByCaller 持续时间的可堆叠 GameplayEffect，只有堆叠中的第一个实例的持续时间被正确设置，堆叠中所有其他 GE Spec 的持续时间都会是 1 秒。添加了自动化测试来检测这种情况。
* Bug 修复：修复了处理 GameplayEvent 委托时修改了 GameplayEvent 委托列表可能引发的 bug。
* Bug 修复：修复了导致 GiveAbilityAndActivateOnce 行为不一致的 bug。
* Bug 修复：重新调整了 FGameplayEffectSpec::Initialize 内部一些操作的顺序，以处理潜在的顺序依赖问题。
* 新增：UGameplayAbility 现在有一个 OnRemoveAbility 函数。它遵循与 OnGiveAbility 相同的模式，只会在能力的主实例或类默认对象上调用。
* 新增：显示被阻止的能力标签时，调试文本现在会包含被阻止标签的总数。
* 新增：把 UAbilitySystemComponent::InternalServerTryActiveAbility 重命名为 UAbilitySystemComponent::InternalServerTryActivateAbility。原来调用 InternalServerTryActiveAbility 的代码现在应改为调用 InternalServerTryActivateAbility。
* 新增：添加或删除标签时，继续使用过滤文本来显示 GameplayTag。以前的行为会清除过滤条件。
* 新增：在编辑器中添加新标签时，不再重置标签来源。
* 新增：增加了查询能力系统组件中所有带有指定标签集合的激活中 GameplayEffect 的功能。新函数名为 GetActiveEffectsWithAllTags，可以通过代码或蓝图访问。
* 新增：与根运动移动相关的能力任务结束时，现在会把移动组件的移动模式恢复为任务开始之前的移动模式。
* 新增：把 SpawnedAttributes 设为瞬态（transient），这样它就不会保存可能过期和错误的数据。添加了空指针检查，防止当前已保存的过期数据继续传播。这避免了因错误数据被存入 SpawnedAttributes 而引发的问题。
* API 变更：AddDefaultSubobjectSet 已被弃用，应改用 AddAttributeSetSubobject。
* 新增：GameplayAbility 现在可以指定在哪个 Anim Instance 上播放蒙太奇。

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_25/

<a name="changelog-4.24"></a>
### 4.24
* 修复了蓝图节点的 `Attribute` 变量在编译时被重置为 `None` 的问题。
* 需要调用 [`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata) 才能使用 [`TargetData`](#concepts-targeting-data)，否则会出现 `ScriptStructCache` 错误，客户端也会与服务器断开连接。我的建议是，现在每个项目都始终调用它，而在 4.24 之前这是可选的。
* 修复了把 `GameplayTag` setter 复制到一个之前没有定义该变量的蓝图时的崩溃。
* `UGameplayAbility::MontageStop()` 函数现在会正确使用 `OverrideBlendOutTime` 参数。
* 修复了组件上的 `GameplayTag` 查询变量在编辑后没有被修改的问题。
* 为 `GameplayEffectExecutionCalculations` 增加了支持，使其可以对不需要由属性捕获作为后备的“临时变量”使用作用域修改器（scoped modifier）。
  * 这项实现基本上是允许创建由 `GameplayTag` 标识的聚合器，作为执行计算暴露临时值、并用作用域修改器对其进行操作的手段；现在你可以构建那些需要可被操作、但又不需要从来源或目标捕获的数值的公式。
  * 使用方法：执行计算需要把一个标签添加到新的成员变量 `ValidTransientAggregatorIdentifiers` 中；这些标签会显示在作用域修改器的计算修改器数组底部，并被标记为临时变量——相应的细节面板自定义也做了更新以支持该功能
* 对受限标签（restricted tag）做了易用性改进。移除了受限 `GameplayTag` 来源的默认选项。添加受限标签时不再重置来源，以便更容易地连续添加多个。
* `APawn::PossessedBy()` 现在会把 `Pawn` 的 Owner 设置为新的 `Controller`。这很有用，因为如果 `ASC` 位于 `Pawn` 上，[Mixed 复制模式](#concepts-asc-rm)要求 `Pawn` 的 Owner 是 `Controller`。
* 修复了 `FAttributeSetInitterDiscreteLevels` 中与 POD（Plain Old Data）相关的 bug。

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_24/

**[⬆ 返回目录](#table-of-contents)**

---

<a name="translation-glossary"></a>
## 附录：术语对照表

> 本附录为译者补充，不属于原文。表中列出本译文对 GAS 与虚幻引擎术语的统一译法。原文用反引号标注的类名、类型名、函数名与资产名一律保留英文；下表主要针对正文中以自然语言出现的概念。

### GAS 核心概念

| 英文 | 本文译法 | 说明 |
| --- | --- | --- |
| GameplayAbilitySystem (GAS) | GameplayAbilitySystem（GAS） | 插件名，保留英文 |
| AbilitySystemComponent (ASC) | 能力系统组件（ASC） | 正文中多用 `ASC` |
| Ability / GameplayAbility (GA) | 能力 / GameplayAbility | 泛指时译“能力”，指类型时保留英文 |
| Attribute | 属性 | |
| AttributeSet | 属性集 / `AttributeSet` | |
| BaseValue / CurrentValue | 基础值 / 当前值 | 保留英文字段名 |
| Meta Attribute | 元属性 | |
| Derived Attribute | 派生属性 | |
| GameplayEffect (GE) | GameplayEffect | 保留英文 |
| GameplayEffectSpec (GESpec) | GameplayEffectSpec | 保留英文 |
| GameplayEffectContext | GameplayEffectContext | 保留英文 |
| Modifier | 修改器 | |
| Magnitude | 幅度 | |
| Aggregator | 聚合器 | |
| Modifier Magnitude Calculation (MMC) | 修改器幅度计算 | |
| Execution Calculation (ExecCalc / Execution) | 执行计算 | |
| Calculation Modifier | 计算修改器 | |
| Backing Data / backing Attribute | 后备数据 / 后备属性 | |
| Snapshot | 快照 | |
| Duration Policy: Instant / Duration / Infinite | 瞬时 / 持续 / 无限 | 保留英文枚举名 |
| Periodic Effect / Period | 周期效果 / 周期 | |
| Stacking / Stack Count | 堆叠 / 堆叠层数 | |
| Aggregate by Source / Target | 按来源聚合 / 按目标聚合 | |
| Granted Tags / Granted Abilities | 授予的标签 / 授予的能力 | |
| Ongoing Tag Requirements | 持续标签要求 | |
| Application Tag Requirements | 施加标签要求 | |
| Immunity | 免疫 | |
| Custom Application Requirement (CAR) | 自定义施加条件 | |
| Cost / Cooldown | 消耗 / 冷却 | |
| Cooldown Tag | 冷却标签 | |
| SetByCaller | SetByCaller | 保留英文 |
| GameplayTag | GameplayTag | 保留英文 |
| Loose GameplayTag | 松散标签 | |
| GameplayCue (GC) | GameplayCue | 保留英文 |
| GameplayCueManager | GameplayCue 管理器 | |
| AbilityTask (AT) | AbilityTask | 保留英文 |
| Ability Set | 能力集 | |
| Ability Batching | 能力批处理 | |
| AbilitySystemGlobals | 能力系统全局对象 | |
| Target Data / Target Actor | 目标数据 / 目标 Actor | |
| Targeting | 目标选择 | |
| Reticle | 准星 | |

### 策略与网络

| 英文 | 本文译法 |
| --- | --- |
| Instancing Policy | 实例化策略 |
| Instanced Per Actor / Per Execution / Non-Instanced | 每个 Actor 一个实例 / 每次执行一个实例 / 非实例化 |
| Net Execution Policy | 网络执行策略 |
| Net Security Policy | 网络安全策略 |
| Replication Policy | 复制策略 |
| Replication / replicate | 复制 |
| Replication Mode (Full / Mixed / Minimal) | 复制模式（保留英文枚举名） |
| owning client | 拥有者客户端 |
| Authority / server | 权威端 / 服务器 |
| Autonomous Proxy | 自主代理 |
| Simulated Proxy | 模拟代理 |
| Listen Server / Dedicated Server | Listen Server / 专用服务器 |
| relevancy / always relevant | 相关性 / 始终相关 |
| Multicast / NetMulticast | 多播 / NetMulticast |
| reliable / unreliable | 可靠 / 不可靠 |
| Prediction / client-side prediction | 预测 / 客户端预测 |
| Prediction Key | 预测键 |
| Scoped Prediction Window | 作用域预测窗口 |
| Synch Point | 同步点 |
| misprediction / rollback | 预测错误 / 回滚 |
| latency reconciliation | 延迟校正 |
| stale | 过期 |
| race condition | 竞态条件 |

### 引擎通用

| 英文 | 本文译法 |
| --- | --- |
| Actor / Pawn / Character / Controller | 保留英文 |
| OwnerActor / AvatarActor | 保留英文 |
| Possess | 控制（Possess） |
| Spawn | 生成 |
| Blueprint | 蓝图 |
| Class Default Object (CDO) | 类默认对象（CDO） |
| delegate / callback | 委托 / 回调 |
| override | 重写 |
| subclass | 子类 / 子类化 |
| specifier | 说明符 |
| Montage / AnimMontage | 蒙太奇 / 动画蒙太奇 |
| Root Motion Source | 根运动源 |
| trace / overlap | 射线检测 / 重叠检测 |
| hitscan | 即时命中 |
| latent action | 潜伏动作 |
| clamp | 钳制 |
| transient | 瞬态 |
| Tick | Tick |
| Widget | 控件 |
| buff / debuff | 增益 / 减益 |

**[⬆ 返回目录](#table-of-contents)**
