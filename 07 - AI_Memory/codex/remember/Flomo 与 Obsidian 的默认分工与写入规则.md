---
title: Flomo 与 Obsidian 的默认分工与写入规则
type: note
permalink: main/codex/remember/flomo-与-obsidian-的默认分工与写入规则
tags:
- codex
- flomo
- obsidian
- routing
---

# Flomo 与 Obsidian 的默认分工与写入规则

本次会话最终明确了 Flomo 与 Obsidian 的非对称分工，以及以后整理内容时的默认写入依据。

## 核心分工

Flomo 是用户的认知传记，保存认知现场：遇到了什么、感受到什么、产生了什么疑问、原来怎样理解、现在怎样重新理解，以及自己发生了什么变化。Obsidian 是用户的专业工作台，维护可复用知识：问题结构、原因、机制、方案、边界、代码、图示、验证和应用场景。

核心判断：Flomo 记录“我如何变化”，Obsidian 记录“知识如何使用”。不按生活/技术、设备、篇幅或深度机械划分。

## 路由规则

重要性决定是否记录；复用性和维护性决定是否进入 Obsidian。

- 隐含主语是“我”，内容还不确定或需要立即捕捉时，优先 Flomo。
- 隐含主语是“问题/知识”，并需要持续修订、链接、代码、图示、验证或复用时，优先 Obsidian。
- 两边都有价值时，Flomo 保留触发情境、原有困惑和认知变化；Obsidian 重写去情境化的可复用成果；不全文复制成两个正式版本。
- Flomo 内容不必全部整理、升级或迁移。等同类问题反复出现，或现实中的项目、面试、排障、写作确实需要时，再提炼到 Obsidian。
- 手机端突然出现的想法默认先进入 Flomo；电脑端正在维护某个 Obsidian 知识对象时，能直接补全该对象的内容可直接写入 Obsidian。

## 默认写法

Flomo：发生了什么 -> 我想到什么 -> 它改变了什么。

Obsidian：问题是什么 -> 为什么发生 -> 原理/方案 -> 边界 -> 如何验证和应用。

回顾 Flomo 是为了重新认识自己；回顾 Obsidian 是为了更新解决问题的能力。

## 技术概念卡片的判断

纯定义或概念澄清卡处于中间状态：如果重点是“我原来为什么混淆、怎样想通、理解发生了什么变化”，写成 Flomo 的认知记录；如果未来需要用于架构分析、项目、面试或排障，则在 Obsidian 重写成包含层次、边界、场景、验证和应用的正式模型。

例如，Function Calling、MCP、Agent Skill 的三层关系：Flomo 保存从“都像工具调用”到“调用表达、工具接入、任务流程”这一理解变化；Obsidian 保存三层职责、边界、组合方式和实际场景。

## 工具与写入约束

- 用户要求操作 Flomo 时默认使用 Flomo MCP；要求操作 Obsidian 时默认使用 Obsidian MCP；只有用户明确要求其他方式或 MCP 不可用时才切换。
- 写入前先搜索并完整读取已有目标内容，避免重复创建和截断覆盖。
- 写入后重新读取，验证正文、标签和保存结果。

## Observations

- [decision] Flomo 作为个人轨迹和认知变化的事实源，Obsidian 作为可持续维护的专业知识事实源。
- [pattern] 同一主题可以出现在两边，但必须回答不同问题：Flomo 保存认知变化证据，Obsidian 保存可复用模型。
- [boundary] 重要性决定是否记录；复用性和维护性决定是否进入 Obsidian。
- [example] 事务原子性与锁原子性：Flomo 记录用户如何分清两者，Obsidian 整理事务、锁、隔离级别和 MVCC 的完整关系。

## Relations

- relates_to [[Basic Memory Cheat Sheet]]
- relates_to [[Startup Router]]