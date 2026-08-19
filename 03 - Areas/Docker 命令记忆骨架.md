---
tags:
  - Areas/Coder/工程化思维
category: 思维
status: 加工
project:
application:
source:
---
[「记忆规律」与「知识体系建设」](「记忆规律」与「知识体系建设」.md)

```
L1: 核心问题
如何解决应用在不同环境下运行不一致、不纯净，多方通信、统一配置和健康管理？

L2: 方案
├── 打包运行环境
├── 进程要隔离
├── 容器与宿主机通信
├── 容器与容器通信
├── 多服务统一配置
└── 线上要排查

L3: 解决机制
├── 打包运行环境 -> Dockerfile + 镜像
├── 进程要隔离 -> 容器 
├── 容器与宿主机通信 -> 端口映射 / volume目录挂载 
├── 容器互相通信 -> Docker 网络
├── 线上排查 -> logs /inspect / stats / exec 
└── 多服务统一管理 -> Docker Compose

L4: 机制细节
├── Dockerfile: FROM / WORKDIR / COPY / EXPOSE / CMD / ENTRYPOINT
├── 镜像: images / pull / build / rmi
├── 容器: ps / run / stop / start / restart / rm
├── 端口: -p 宿主机端口:容器端口
├── 数据卷: -v 宿主机目录:容器目录
├── 网络: 默认网桥 / 自定义网桥 / 容器名访问
└── 排查: ps -a / logs / exec / inspect / stats

L5: 应用场景、验证和优化
├── Java 应用部署
├── 容器启动失败排查
├── 外部访问不通排查
├── 数据持久化
├── 多容器编排
└── 镜像体积、配置注入、生产部署边界优化
```
