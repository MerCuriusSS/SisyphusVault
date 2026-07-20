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
Linux 命令 = 观察服务器现场

L1：服务器/应用出问题时如何定位？

L2：七个观察面
这个问题在哪里（位置）
这个问题描述是什么（内容）
是否有操作限制（权限）
运行是否正常（进程）
CPU、内存、磁盘（资源）
网络通不通（网络）
服务为什么不能启动（服务）

L3：每层对应命令
位置 -> pwd / ls / cd / find
内容 -> less / tail / grep
权限 -> chmod / chown
进程 -> ps / top / kill
资源 -> top / free / jstat / jmap / df / du / lsof
网络 -> ping / curl / telnet / ss
服务 -> systemctl / journalctl

L4：命令细节
cmd [参数] 操作对象
参数 = 增强观察能力
输出 = 验证当前假设

L5：场景触发
CPU 高 -> top -> top -Hp -> jstack
内存高 -> free -> top -> jstat -> jmap
磁盘满 -> df -> du -> find -> lsof deleted
端口不通 -> ps -> ss -> curl -> telnet -> 防火墙/代理/Docker
服务失败 -> systemctl -> journalctl
日志异常 -> tail -> grep -> less
权限失败 -> ls -lh -> chmod/chown
```