---
tags:
  - Areas/Coder/基础原理
category: 技术
status: 加工
project:
application:
source:
---
### 核心困惑点：
既然 nginx、redis、应用、mysql 等服务为了可用性和资源隔离，应该分开部署，
那为什么用了 Docker 之后，反而把这些服务都放到同一台服务器里的不同容器中？
如果这台服务器挂了，所有容器都会挂，那 Docker 的意义在哪里？
如果要做一主一从，是在同一台服务器里再起备用容器，还是在另一台服务器起一套？
不同服务器上的容器主从关系又怎么管理？


### 概念区分：

🔴**Docker**：运行在同宿主机中的「独立进程」，主要解决部署不方便、服务不隔离、无法自动重启问题。
🔴**Docker-compose**：解决如何「一次性」拉起各个容器，实现快速部署的问题。
🔴**高可用架构**：「多实例跨服务器」，解决服务器故障、服务故障、数据故障、流量切换等问题。

### 高可用方式：

**核心结论**：不同服务的高可用方式不同，主从架构只针对于「有状态」组件（数据库等）

| 组件      | 高可用方式                                  |
| ------- | -------------------------------------- |
| Java 应用 | 多副本 + 负载均衡                             |
| Nginx   | 多实例 + SLB/VIP                          |
| MySQL   | 主从/主备 + 自动故障转移，或直接用 RDS                |
| Redis   | 主从 + Sentinel，或 Redis Cluster，或云 Redis |


### 高可用方案：

🔴**Docker Compose 多机各自部署 + 外部负载均衡/HA 组件**
结构：
```
                ┌────────────┐
用户 ─────────▶ │ SLB / VIP   │
                └─────┬──────┘
                      │
          ┌───────────┴───────────┐
          │                       │
      server-1                server-2
   order-service-1         order-service-2
   redis-primary           redis-replica
   mysql-primary           mysql-replica

```
组件描述：
- SLB / VIP：
- 无状态组件：
- 有状态组件：依赖组件本身的主从机制（mysql复制机制、redis哨兵机制）

🔴K8S+云平台托管
结构：
```
用户
  ↓
云负载均衡 SLB
  ↓
Ingress / Gateway
  ↓
order-service Deployment 多副本
  ↓
RDS MySQL 高可用版
  ↓
Redis 高可用版

```

组件描述：
- 无状态组件：Kubernetes 管
- 有状态组件：交由云服务厂商管理（）