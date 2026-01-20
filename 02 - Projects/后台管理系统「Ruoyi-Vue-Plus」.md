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
    

---


**核心目标**：从“业务实现者”向“体系构建者”转型，掌握分布式与多租户隔离的工程化标准。

- **行动方案**：
    1. **架构解构**：剖析admin、common、system模块化设计，理解依赖隔离的物理逻辑。
    2. **安全平替**：对比Sa-Token与Spring Security，实操多端登录与二级认证逻辑，摒弃臃肿的Filter链思维。
    3. **数据隔离**：通过MyBatis-Plus拦截器实现行级物理隔离，手动调试JSqlParser产生的SQL冲突问题。
    4. **多级缓存**：落地Caffeine+Redis二级缓存架构，利用Redis Pub/Sub实现本地缓存失效机制。
- **KPI**：
    - 独立完成一个基于Sa-Token的自定义权限策略扩展。
    - 在复杂JOIN查询下，成功解决多租户拦截器导致的语法错误。
- **截止日期**：第7天。
        
