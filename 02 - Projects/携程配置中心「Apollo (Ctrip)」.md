---
title: 携程配置中心「Apollo (Ctrip)」
status: 新建
deadline: 2026-01-26
tags: project
address: https://github.com/apolloconfig/apollo
---
>[!example] 🏗️ Areas
> ```dataview
> LIST 
> FROM "03 - Areas"
> WHERE project=this.file.link
> ```


---

### 项目二：Apollo (分布式配置中心) —— 攻克“分布式一致性与底层原理”

这个阶段的目标是找回你作为 5 年工程师的**底层逻辑感**。面试官最爱问：配置改了，客户端怎么实时感知的？

- **第 1-2 天：【长轮询架构】**
    
    - **里程碑：** 彻底搞懂 `Http Long Polling`。
        
    - **核心动作：** 研究 `ConfigController.java`，看服务端如何 Hold 住请求，以及 `DeferredResult` 的使用。
        
    - **面试点：** “为什么不用 WebSocket？为什么不用简单的定时轮询？”
        
- **第 3-4 天：【多级缓存机制】**
    
    - **里程碑：** 掌握 Apollo 的“四级读写”流程（内存 -> 数据库 -> 本地文件 -> 内存镜像）。
        
    - **核心动作：** 观察客户端在断网或宕机重启时，如何通过本地缓存实现“配置高可用”。
        
- **第 5 天：【灰度发布实现】**
    
    - **里程碑：** 理解配置的分片发布逻辑。
        
    - **核心动作：** 查看 Apollo 的“灰度（Release）”源码，理解如何针对不同 IP/AppId 下发不同配置。
        
    - **面试点：** “如何设计一个支持灰度发布的系统？”
        
