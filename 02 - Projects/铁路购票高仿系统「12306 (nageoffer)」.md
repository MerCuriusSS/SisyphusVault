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
## 4. 12306 (OpenGoofy)：高并发一致性攻坚

**定位：** 应对大流量面试场景，掌握分布式锁的底层逻辑 14。

### 项目大纲与里程碑

- **阶段一：Redisson 锁深度复刻 (第 21-24 天)**
    
    - **步骤：** 研读 Lua 脚本加锁逻辑与看门狗（Watchdog）源码 15。
        
    - **里程碑：** 模拟“锁过期但任务未执行完”场景，手动触发续期逻辑。
        
- **阶段二：缓存穿透与拦截策略 (第 25-27 天)**
    
    - **步骤：** 在购票流程中集成布隆过滤器 (Bloom Filter)。
        
    - **里程碑：** 实现一套基于 Redis 状态位与数据库唯一索引的金融级幂等保障。
        
- **阶段三：系统性复盘与面试演绎 (第 28-30 天)**
    
    - **AI 任务：** 利用演绎型 AI，将抢票流程中的分布式锁嵌套与 RocketMQ 削峰逻辑转化为结构化的系统设计图 16。
        
