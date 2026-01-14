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


---

## 3. Hippo4j：动态并发治理与设计模式

**定位：** 升华 Java 并发理论，从“调参数”升级到“治线程” 11。

### 项目大纲与里程碑

- **阶段一：动态参数更新原理 (第 15-17 天)**
    
    - **步骤：** 研究如何通过反射实时修改 `ThreadPoolExecutor` 的核心参数。
        
    - **里程碑：** 集成 Hippo4j 客户端，实现通过配置中心动态调整线程池水位。
        
- **阶段二：监控指标采集逆向 (第 18-20 天)**
    
    - **步骤：** 剖析装饰器模式在任务执行耗时统计中的应用 12。
        
    - **里程碑：** 编写一个装饰器，在不修改业务代码前提下采集线程池活跃度指标。
        
    - **AI 任务：** 归纳 Hippo4j 报警策略中的“策略模式”实现方式 13。
        

---
