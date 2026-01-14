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

## 2. Apollo：配置中心深度原理与容灾演进

**定位：** 攻克分布式通讯模型，理解 AP 架构下的极致可用性 7。

### 项目大纲与里程碑

- **阶段一：长轮询（Long Polling）机制模拟 (第 8-10 天)**
    
    - **步骤：** 研究 `ConfigController` 中 `DeferredResult` 的使用 8。
        
    - **里程碑：** 手写一个简易的长轮询 Demo，模拟服务端挂起 60s 并在配置变更时即时返回。
        
    - **AI 任务：** 演绎长轮询与 WebSocket 的资源消耗对比图表 9。
        
- **阶段二：多级缓存冗余设计 (第 11-12 天)**
    
    - **步骤：** 追踪 `RemoteConfigRepository` 的缓存流转。
        
    - **里程碑：** 模拟配置中心宕机，验证本地磁盘文件（二级缓存）的启动加载逻辑 10。
        
- **阶段三：分布式一致性博弈 (第 13-14 天)**
    
    - **步骤：** 分析 ReleaseMessage 消息发布订阅逻辑。
        


