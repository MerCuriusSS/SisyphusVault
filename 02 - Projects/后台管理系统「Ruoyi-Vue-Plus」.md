---
title: 后台管理系统「Ruoyi-Vue-Plus」
status: 运行
deadline: 2026-01-18
tags: project
address: https://gitee.com/dromara/RuoYi-Vue-Plus
---
>[!example] 🏗️ Areas
> ```dataview
> LIST 
> FROM "03 - Areas"
> WHERE project=this.file.link
> ```


## 🚀 核心战略：AI 双模驱动下的“闭环学习”

在执行以下大纲时，请务必遵循：

- **归纳型 AI (GPT/Claude)：** 用于深度对齐源码逻辑。例如：_“请分析 Ruoyi-Vue-Plus 的数据权限拦截器元模型，总结其权限过滤的底层规则。”_ 1
    
- **演绎型 AI (NotebookLM/Midjourney)：** 用于将技术原理转化为面试演示文档或架构图 2。
    
- **行动逻辑：** 尝试阅读源码 -> 失败 -> 询问 AI 核心元模型 -> 动手复现关键代码（里程碑） -> 撰写技术复盘（知识库）
    
[Claude Code拆解复杂业务项目方法论](../04%20-%20Resources/Claude%20Code拆解复杂业务项目方法论.md)

[ruoyi-vue-plus项目拆解](../ruoyi-vue-plus项目拆解.md)

---


## 🎯 项目目标——「一句话定义成功」

### 找回后端开发的手感，重新掌握实施的框架以及通用配置、业务流程的CRUD操作以及常用的设计思想。

## 🍎关键成果——「实现里程碑」

- [ ] ⚠️ 拆解「Ruoyi-Vue-Plus」项目构造以及模块划分
- [ ] ⚠️ 掌握「Ruoyi-Vue-Plus」的功能模块「按需加载」设计思想
- [ ] ⚠️ 巩固「分层架构思想+BO/VO分离」
- [ ] ⚠️ 厘清「过滤器」与「拦截器」概念与应用场景
- [ ] ⚠️ 掌握「RBAC」权限设计思想以及「Sa-Token」认证框架
- [ ] ⚠️ 建立「生产环境排查」所需的认知


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

### 2026/02/26

**任务区：**

- [x] 建立「java生产项目问题排查」的基础认知，理解其 **核心定义** 与 **核心价值**。 ✅ 2026-02-26
- [x] 设立「提问钩子」，描述它解决了我哪些问题，填补了哪一块知识空白 ✅ 2026-02-26
- [x] 拆解「问题排查」所需观测的三大维度（日志、指标、追踪）内容，包括是什么？要做什么？怎么做？ ✅ 2026-02-26
- [x] 问题排查所需要掌握的方法论 ✅ 2026-02-26
- [x] 如何应用到实际应用场景。 ✅ 2026-02-26

**记录区：**
1. 问题排查需要「日志」、「指标」、「追踪」三位一体的，既要通过日志保证功能不出错，又要通过指标保证系统资源利用良好、不停顿，还要通过追踪保证分布式系统的服务依赖链路有迹可循。
2. 日志按内容可分为2种，一种是以切面形式记录的用户操作日志（谁、什么时候、做了什么？结果如何？），保存到数据库中，旨在可视化具体用户行为，可作为审计；一种则是系统日志（接口路径、入参），保存到文件系统中，旨在记录业务信息（info类型）和堆栈信息（error类型）。
3. **Actuator**功能非常强大，能采集各种系统指标信息（操作系统、JVM、连接池），配合Prometheus+Grafana还可以实现 监控可视化以及趋势变化。
4. 

---



🏁 项目收口

- [ ] ⚠️归档前清理：所有 Task 是否已完成？
- [ ] ⚠️沉淀：是否有 Areas 笔记需要转化为永久知识？
