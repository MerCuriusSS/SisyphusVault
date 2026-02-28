---
title: A股投资团队项目
status: 运行
deadline:
tags: project
address:
---
>*实操指南：*
>*A. 「推进日志」推进 每日待办 & 踩坑记录*
>*B.  项目积累的笔记记录到「Areas」和「Resources」，并通过`yaml` 标签project:{project_name}进行关联*
>*C. 「待办任务」实现项目所有待办任务的统一汇总*

## 🎯 项目目标——「一句话定义成功」

**项目完整落地并自动化运行**

## 🍎关键成果——「实现里程碑」

- [ ] ⚠️ 总结并借鉴「AI员工一人公司」的运作模式，建立「A股投资」项目实现大纲
- [ ] ⚠️ 开展阶段性的实施工作，从「AI指导」、「A股投资」、「具体实施工具」三个维度建立相关认知。


## ✅ 待办任务

>[!note] tasks
> ```tasks
> not done
> path includes {{query.file.path}}
> group by priority
> description does not include ⚠️
> ```


## 🧠 关联知识 

>[!example] Areas
> ```dataview
> LIST 
> FROM "03 - Areas"
> WHERE project=this.file.link
> ```


## 📚 参考资源

>[!question] resources
>```dataview
>LIST FROM "04 - Resources"
>WHERE project = this.file.link
>```


## 📝 推进日志

### 2026-02-28

**任务区：**

- [ ] 
- [ ] 待办任务2:..

**记录区：**

踩坑记录。。。。

---



🏁 项目收口

- [ ] ⚠️归档前清理：所有 Task 是否已完成？
- [ ] ⚠️沉淀：是否有 Areas 笔记需要转化为永久知识？