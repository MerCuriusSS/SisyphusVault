---
title: 铁路购票高仿系统「12306 (jOpenGoofy)」
status: 新建
deadline: 2026-02-08
tags: project
address: https://gitee.com/nageoffer/12306
---
>[!example] 🏗️ Areas
> ```dataview
> LIST 
> FROM "03 - Areas"
> WHERE project=this.file.link
> ```



---
**核心目标**：掌握在亿级数据与高并发场景下的分布式协同与代码艺术。

- **行动方案**：
    1. **分布式锁深度实践**：利用Redisson实现占座逻辑，必须掌握Watchdog自动续期与Pub/Sub等待通知机制，解决锁提前释放难题。
    2. **海量数据治理**：使用ShardingSphere进行分库分表，设计复合分片算法及雪花算法，确保通过订单号实现$O(1)$级别查询。
    3. **代码重构艺术**：应用策略模式与模板方法模式，将复杂的计费与下单流程从硬编码重构为可扩展的架构。
- **KPI**：
    - 模拟网络波动环境，证明Redisson Watchdog机制的有效性。
    - 画出订单中心的分片逻辑图，并解释如何避免跨分片查询带来的性能灾难。
- **截止日期**：第21天。