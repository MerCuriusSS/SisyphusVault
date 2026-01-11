# 🚀 个人知识指挥中心

> [!tip] 🎉 **随机回顾 (灵感)**
> ```dataview
> LIST FROM "03 - Areas" WHERE status = "沉淀" LIMIT 1
> ```


> [!todo] 📂 **[[01 - Inbox|收件箱]]**（`$= dv.pages('"01 - Inbox"').length`） 
> > [!example]- 最近5个
> >```dataview
> >LIST FROM "01 - Inbox" LIMIT 5
> >```


> [!warning] ✅ 执行中心 
> >[!info] **重点提醒** > > 请优先处理高优先级 (⏫) 任务。
> 
> >[!quote]- 本周
> >```tasks
> >not done
> >due before next monday
> >group by priority
> >```


> [!activity] 🎃 项目监控
> ```dataview
> 
> TABLE file.mtime AS "最后修改时间", file.tags AS "标签" FROM "02 - Projects" SORT file.mtime DESC
> ```

> [!example]- 🏗️ 领域加工
> **🍬 进度**
> ```dataview
> LIST FROM "03 - Areas" WHERE status != "沉淀"
> ```

