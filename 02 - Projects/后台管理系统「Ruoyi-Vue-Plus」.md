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
    
- **行动逻辑：** 尝试阅读源码 -> 失败 -> 询问 AI 核心元模型 -> 动手复现关键代码（里程碑） -> 撰写技术复盘（知识库） 3。
    

---

## 1. Ruoyi-Vue-Plus：规范化工业级框架重塑

**定位：** 找回工程语感，对齐分布式单体开发的“标准模板” 4。

### 项目大纲与里程碑

- **阶段一：基础设施与权限元模型 (第 1-3 天)**
    
    - **步骤：** 搭建 SpringBoot 3.x 环境，接入 Sa-Token。
        
    - **里程碑：** 实现一个基于注解的自定义权限校验拦截器。
        
    - **AI 任务：** 归纳 Sa-Token 与 Spring Security 的设计差异，总结其在分布式 Session 管理上的优势 5。
        
- **阶段二：防御性编程实战 (第 4-5 天)**
    
    - **步骤：** 剖析 `ruoyi-common-idempotent` 模块。
        
    - **里程碑：** 手写一个基于 Redis 的接口防重组件（RepeatSubmitInterceptor） 6。
        
- **阶段三：多租户与数据隔离 (第 6-7 天)**
    
    - **步骤：** 调研 Mybatis-Plus 拦截器如何实现租户 ID 自动填充。
        
    - **里程碑：** 在本地模拟一套 SQL 自动注入过滤逻辑。
        
