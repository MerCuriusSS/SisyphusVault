---
title: 动态线程池框架「Hippo4j」
status: 新建
deadline: 2026-02-01
tags: project
address: https://github.com/opengoofy/hippo4j
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



**核心目标**：突破并发编程的“经验主义”，建立“感知-反馈-调节”的闭环治理思维。

- **行动方案**：
    - **动态参数研究**：深入原生`ThreadPoolExecutor`源码，理解`setCorePoolSize`在JVM层面的线程启停逻辑。
    - **监控中台搭建**：集成Nacos配置中心，实现生产环境不重启即可动态调整核心参数、最大线程数及拒绝策略。
    - **风险干预**：设计基于水位线（如80%活跃度）的告警响应机制，利用Hippo4j对核心交易线程池进行全生命周期管理。
- **KPI**：
    - 在不重启应用的情况下，通过配置中心成功干预一个模拟OOM风险的线程池。
    - 输出一份基于$T_{waiting}$与$T_{total}$公式的线程池状态评估报告。
- **截止日期**：第14天。