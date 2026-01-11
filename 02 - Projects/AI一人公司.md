---
title: Obsidian语法笔记
tags: [Obsidian, 语法, #Dataview]  # 数组格式（推荐）
# 或单行字符串格式：tags: Obsidian 语法 Dataview
---

```dataview
LIST
FROM "03_Areas" OR "04_Resources"
WHERE project = this.file.link
AND status != "沉淀"
SORT file.mtime DESC
```


## 1. 核心结论 (3秒原则)
> 用一句话描述这个技术点或思维方式的核心价值。

## 2. 我的见解/重构
- 为什么要记录它？它解决了什么我以前解决不了的问题？
- 它的底层逻辑是什么？（尝试用最简单的类比解释给外行听）

## 3. 场景设想
- **场景 A**：在处理 [XXX] 代码逻辑时可以替代原有的 [YYY] 方法。
- **场景 B**：在进行 [ZZZ] 决策时，用来规避逻辑谬误。