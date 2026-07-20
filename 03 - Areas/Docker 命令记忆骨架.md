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
为什么应用在不同环境中【运行】结果不一致，【部署】和【排查】也容易混乱？

L2: 关键原因
├── 运行环境不一致
├── 数据要持久化（开发、生产环境数据不一样，不同环境运行结果不一致）
├── 进程要隔离（排查：传统方式资源共享，错误干扰因素多，不够纯净）
├── 多服务难管理（部署：传统多部署需逐一修改生产配置，效率低、错误率高）
└── 线上要排查

L3: 解决机制
├── 运行环境不一致 -> Dockerfile + 镜像
├── 进程缺少隔离 -> 容器
├── 外部访问容器 -> 端口映射
├── 数据不能丢 -> volume / 目录挂载
├── 容器互相通信 -> Docker 网络
├── 线上排查 -> logs / exec / inspect / stats
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
