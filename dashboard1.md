# 🚀 个人知识指挥中心

> [!success] ✅ 执行中心 (本周)
> ```tasks
> not done
> due before next monday
> group by priority
> ```
> > [!info] **重点提醒** > > 请优先处理高优先级 (⏫) 任务。

> [!abstract] 📥 入口与状态
> `$= dv.pages('"01 - Inbox"').length` 个待分流
> ```dataview
> LIST FROM "01 - Inbox" LIMIT 5
> ```
> 
> **快速链接**
> - [[05 - Inbox|📂 打开收件箱]]
> - [[Daily Notes/2026-01-10|📅 今日日记]]
> 
> **随机回顾 (灵感)**
> ```dataview
> LIST FROM "03 - Areas" WHERE status = "沉淀" LIMIT 1
> ```

> [!activity] 🏗️ 活跃监控
> **当前项目**
> ```dataview
> LIST FROM "01 - Projects" WHERE status = "运行" LIMIT 3
> ```
> 
> 
> **最近资源**
> ```dataview
> LIST FROM "03 - Resources" SORT file.ctime DESC LIMIT 3
> ```



