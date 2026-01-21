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
    - 独立完成一个基于Sa-Token的自定义权限策略扩展。与**AI 协同产出**一套由 AI 辅助生成的“Sa-Token 深度集成元模型手册”。
    - 在复杂JOIN查询下，成功解决多租户拦截器导致的语法错误,在复杂 JOIN 查询解决后，在公开平台发布一篇技术剖析博文，逻辑必须清晰到让外行也能看懂 70% 。
- **截止日期**：第7天。

---
模块层级结构分析

  1️⃣ 根父模块：ruoyi-vue-plus

  - 作用：顶层聚合 POM，统一管理所有子模块
  - 职责：
    - 版本统一管理（当前版本：5.5.2）
    - 依赖版本管理
    - Maven 插件配置
    - 编译规范配置（Java 16+）

  2️⃣ 第一层子模块（4个聚合模块）：

  | 模块          | 类型 | 说明                                    |
  |---------------|------|-----------------------------------------|
  | ruoyi-admin   | JAR  | 主应用入口模块，包含 Spring Boot 启动类 |
  | ruoyi-common  | POM  | 通用模块聚合器，包含 24 个子模块        |
  | ruoyi-modules | POM  | 业务模块聚合器，包含 5 个业务模块       |
  | ruoyi-extend  | POM  | 扩展服务聚合器，包含 2 个扩展服务       |

  3️⃣ 核心依赖链（ruoyi-common 的层级关系）：

  底层 → 顶层：
  common-core (最底层)
      ↓
  common-json
      ↓
  common-redis
      ↓
  common-satoken → common-mybatis
      ↓                 ↓
  common-security   common-tenant
      ↓                 ↓
  业务模块依赖 ← ← ← ← ← ←

  关键依赖说明：
  - common-core: 最底层，只依赖 Spring 框架和第三方工具类（Hutool、Lombok、MapStruct）
  - common-redis: 依赖 core，提供 Redis 缓存功能
  - common-satoken: 依赖 core 和 redis，提供 Sa-Token 认证
  - common-mybatis: 依赖 core 和 satoken，提供数据库操作
  - common-tenant: 依赖 mybatis 和 redis，提供多租户功能
  - common-security: 依赖 satoken，提供安全控制
  - common-web: 依赖 json，提供 Web 服务基础


  模块分类总结

  🔴 核心公共模块（最底层，被其他所有模块依赖）

  - ruoyi-common-core: 提供基础工具类、常量、异常处理、验证注解等

  🟡 基础服务模块（构建在核心模块之上）

  - ruoyi-common-json: JSON 序列化（Jackson）
  - ruoyi-common-redis: Redis 缓存服务
  - ruoyi-common-satoken: Sa-Token 认证框架
  - ruoyi-common-mybatis: MyBatis-Plus 数据库操作
  - ruoyi-common-web: Web 服务基础

  🟢 功能增强模块（依赖基础服务）

  - ruoyi-common-tenant: 多租户支持
  - ruoyi-common-security: 安全控制
  - ruoyi-common-log: 日志记录
  - ruoyi-common-excel: Excel 导入导出
  - ruoyi-common-oss: 对象存储
  - ruoyi-common-sms: 短信服务
  - ruoyi-common-mail: 邮件服务
  - 其他功能模块...

  🔵 业务模块（最终实现）

  - ruoyi-system: 系统管理核心模块（用户、角色、菜单、部门等）
  - ruoyi-job: 定时任务管理
  - ruoyi-generator: 代码生成器
  - ruoyi-demo: 演示模块
  - ruoyi-workflow: 工作流模块

  🟣 应用入口模块

  - ruoyi-admin: Spring Boot 应用启动入口，聚合所有业务模块和通用模块，最终打包为可执行 JAR

  这种模块化设计的优点是：
  1. 层次清晰：核心 → 基础 → 增强 → 业务
  2. 职责单一：每个模块只负责特定功能
  3. 低耦合：通过依赖管理避免循环依赖
  4. 易扩展：新模块可以按需引入依赖
  5. 易维护：修改某个功能只需更新对应的 common 模块