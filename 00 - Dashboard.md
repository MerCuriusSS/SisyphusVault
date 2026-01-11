# 🚀 个人知识指挥中心

> [!abstract] 📥 **随机回顾 (灵感)**
> ```dataview
> LIST FROM "03 - Areas" WHERE status = "沉淀" LIMIT 1
> ```

> [!abstract]- 📥 收件箱状态（`$= dv.pages('"01 - Inbox"').length`） 
> ```dataview
> LIST FROM "01 - Inbox" LIMIT 5
> ```

> [!success] ✅ 执行中心 (本周)
> ```tasks
> not done
> due before next monday
> group by priority
> ```
> > [!info] **重点提醒** > > 请优先处理高优先级 (⏫) 任务。

> [!activity] 🏗️ 领域加工
> **当前项目**
> ```dataview
> LIST FROM "03 - Areas" WHERE status != "沉淀"
> ```

> [!activity] 🏗️ 活跃监控
> **当前项目**
> ```dataview
> LIST FROM "01 - Projects" WHERE status = "运行中"
> ```