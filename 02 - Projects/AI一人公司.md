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
