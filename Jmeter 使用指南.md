---
tags:
  - Areas/Coder/工具
category: 技术
status: 加工
project:
application:
source:
---
## 核心价值

测试系统在资源耗尽时的崩溃行为

## 组成部分

🔴 线程组：模拟用户（一线程 = 一用户）
- Number of Threads：启动线程数量
- Ramp-up period：几秒内完成

🔴 配置元件：公共配置信息——IP地址、端口等

🔴 前置处理器：发请求前的操作

🔴 逻辑控制器：如何执行——分支、循环

🔴 定时器：请求延迟

**🔴 取样器：发请求，取样本。**

🔴 后置处理器：发请求后的操作

🔴 断言：判断返回的数据是否正确

🔴 监听器：收集测试结果

🔴 录制流量：录制用户网页操作

- 测试计划 → 添加 → 用户组 
- 用户组 → 添加 → Recording Controller
- 测试计划 → 添加 → **非测试元件 → HTTP (S) Test Script Recorder**
- HTTP(S) Test Script Recorder → grouping → put each ....
- 浏览器设置代理（127.0.0.1:8888）
- 启动 「HTTP (S) Test Script Recorder」 开始录制 → 网页操作

![](assets/Jmeter%20使用指南/file-20260731182816889.png)

## 压测方法论

1. 所有接口过单机基准
2. 对核心业务做压测（单链路） 
3. 对所有核心业务+其余冷门接口做压测（全链路）【中小项目可选】


