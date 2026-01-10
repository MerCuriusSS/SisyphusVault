---
cssclasses: cards, cards-cols-3, max
---

# 🛰️ 系统监控中心

> [!multi-column]
>
> > [!todo] ### 📥 1. Inbox 待分流
> > **待处理：** `$=dv.pages("01 - Inbox").length` 个
> > ```dataview
> > LIST FROM '"01 - Inbox"'
> > LIMIT 5
> > ```
> 
> > [!quote] ### ⛏️ 2. 待采知识矿
> > ```dataview
> > LIST FROM "Resources"
> > WHERE !processed
> > LIMIT 5
> > ```
> 
> > [!work] ### 🏗️ 3. 运行中项目
> > ```dataview
> > TABLE status AS "状态"
> > FROM "Projects"
> > WHERE status = "active" OR status = "进行中"
> > ```

---

> [!multi-column]
>
> > [!warning] ### 📂 4. Area 状态异常
> > *提醒：查看未处于“沉淀”状态的笔记*
> > ```dataview
> > LIST FROM "Areas"
> > WHERE status != "沉淀" 
> > LIMIT 8
> > ```
> 
> > [!check] ### ✅ 5. 本周待办
> > ```dataview
> > TASK
> > WHERE !completed 
> > AND (due <= date(today) + dur(7 days) OR !due)
> > LIMIT 10
> > ```

---

### 🛠️ 实施要点说明

#### 1. 颜色与图标含义 (Color Coding)
为了让你一眼扫过去就知道哪里出了问题，我配置了不同的 Callout 类型：
* **`[!todo]` (蓝色):** 代表输入端，需要快速处理。
* **`[!quote]` (绿色/黄色):** 代表资源，是你的储备仓库。
* **`[!work]` (紫色):** 代表项目，这是你现在的注意力焦点。
* **`[!warning]` (橙色/红色):** 代表异常，提醒你 Area 区域有笔记需要整理或沉淀。
* **`[!check]` (绿色):** 代表行动，清楚当前的任务压力。

#### 2. CSS 样式增强 (可选)
在笔记的最上方 YAML 区，我加入了 `cssclasses: cards`。
* **如果你安装了 Minimal 主题：** 这些列表会自动变成整洁的卡片状。
* **如果你使用的是默认主题：** 它会保持为精简的列表，依然整齐。

#### 3. 动态计数
在 Inbox 板块，我加了一行 `$=dv.pages('"Inbox"').length`。这是 **DataviewJS** 代码，它会实时显示收件箱里文件的总数。如果数字变红（通过脑补或简单的 CSS），你就知道该“大扫除”了。

---

### 💡 建议的下一步动作
你可以尝试在 **Area 区域** 的 Dataview 代码中，加入按 `mday` (修改日期) 排序：
`SORT file.mday desc`

这样你就能优先看到那些最近动过、但还没来得及归档到“沉淀”状态的笔记。

**需要我帮你微调某个板块的显示内容（比如在项目里增加显示进度条）吗？**
