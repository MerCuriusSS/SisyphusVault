# 🚀 个人知识指挥中心

--- start-multi-column: ID_dashboard_1
search: [column: 1]

> [!abstract] 📥 入口与状态
> `$= dv.pages('"05 - Inbox"').length` 个待分流
>
> **快速链接**
> - [[05 - Inbox|📂 打开收件箱]]
> - [[Daily Notes/2026-01-10|📅 今日日记]]

--- column-break ---

> [!success] ✅ 执行中心 (本周)
> ```tasks
> not done
> due before next monday
> group by priority
> ```

--- column-break ---

> [!activity] 🏗️ 活跃监控
> **当前项目**
> ```dataview
> LIST FROM "01 - Projects" WHERE status = "运行中" LIMIT 3
> ```
>
> **最近资源**
> ```dataview
> LIST FROM "03 - Resources" SORT file.ctime DESC LIMIT 3
> ```

--- end-multi-column ---