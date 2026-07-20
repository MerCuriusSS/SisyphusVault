---
tags:
  - Areas/Coder/工程化思维
category: 思维
status: 加工
project:
application:
source:
---
[「记忆规律」与「知识体系建设」](03%20-%20Areas/「记忆规律」与「知识体系建设」.md)

```
L1: 核心问题
使用事务时，为什么会出现「业务方法“提交/回滚”」与「MySQL事务ACID」不一致导致业务数据不一致的情况？

L2: 关键原因
事务边界不清
代理没生效
异常回滚误判
事务传播关系复杂
持久化与崩溃恢复
快照读不想阻塞写
当前读与阻塞写

L3：关键机制
事务边界不清 -> Spring AOP 事务拦截器
Spring 事务可能没有经过代理 -> 必须通过 Spring 代理对象调用 @Transactional 方法
异常回滚规则容易误判 -> rollbackFor + 抛出可感知异常
方法之间需要不同事务关系 -> propagation 传播行为
多事务并发读写会互相影响 -> MySQL 隔离级别
事务失败需要撤销已执行修改 -> undo log
事务提交后需要崩溃可恢复 -> redo log
普通查询既要一致性又不能阻塞写 -> MVCC + ReadView
当前读和写操作必须读最新数据 -> 行锁 / 间隙锁 / 临键锁

L4: 机制细节
├── Spring AOP 事务拦截器:
│   ├── 调用代理对象
│   ├── 进入事务拦截器
│   ├── 开启事务
│   ├── 执行业务方法
│   └── 根据返回结果提交或根据异常回滚
│
├── Spring 代理对象调用:
│   ├── 外部 Bean 调用代理对象才生效
│   ├── this.method() 内部调用不经过代理
│   ├── 方法通常需要 public
│   └── 类必须交给 Spring 容器管理
│
├── rollbackFor + 可感知异常:
│   ├── 默认回滚 RuntimeException
│   ├── 默认回滚 Error
│   ├── checked exception 默认不回滚
│   ├── rollbackFor = Exception.class 可指定回滚
│   └── catch 后必须继续抛出或手动标记回滚
│
├── propagation 传播行为:
│   ├── REQUIRED: 默认，有事务就加入，没有就新建
│   └── REQUIRES_NEW: 挂起外层事务，开启独立新事务
│
├── MySQL 隔离级别:
│   ├── RU 读未提交: 可能脏读、不可重复读、幻读
│   ├── RC 读已提交: 可能不可重复读、幻读
│   ├── RR 可重复读: InnoDB 默认，快照读通常保持一致视图
│   └── Serializable 串行化: 隔离最强，并发最低
│
├── undo log:
│   ├── 保存修改前的旧版本
│   ├── 支持事务回滚
│   └── 支持 MVCC 读取历史版本
│
├── redo log:
│   ├── 记录已提交事务的物理修改
│   ├── 支持崩溃恢复
│   └── 保证事务提交后的持久性
│
├── MVCC + ReadView:
│   ├── MVCC 解决读-写并发
│   ├── ReadView 判断行版本是否可见
│   ├── trx_id 表示行版本所属事务
│   ├── m_ids 表示生成 ReadView 时的活跃事务集合
│   ├── m_up_limit_id 表示活跃事务中最小事务 id
│   └── m_low_limit_id 表示下一个将分配的事务 id
│
├── ReadView 与隔离级别:
│   ├── RU: 不生成 ReadView，直接读最新数据
│   ├── RC: 每次查询生成新的 ReadView
│   └── RR: 首次查询生成 ReadView，后续复用同一个快照
│
├── 快照读:
│   ├── 普通 select
│   ├── 通过 MVCC 读取历史版本
│   ├── 通常不加锁
│   └── 通过一致性快照减少变化影响
│
├── 当前读:
│   ├── 读取最新数据
│   ├── select ... for update
│   ├── select ... lock in share mode
│   ├── update / delete
│   └── 通过锁防止并发修改或插入影响判断
│
└── 锁机制:
    ├── 功能维度: 读锁 / 写锁
    ├── 范围维度: 行锁 / 间隙锁 / 临键锁 / 表锁
    ├── 行锁: 锁索引行，粒度小，并发高
    ├── 间隙锁: 锁索引行之间的空白区间，防止范围插入
    ├── 临键锁: 行锁 + 间隙锁，常见理解是左开右闭
    └── 表锁: 锁整张表，并发低，可能由主动 lock table 或索引失效等导致影响范围扩大

L5；应用场景
@Transactional 没回滚
同一事务两次查询结果变化
普通 select 不想阻塞 update
出现锁等待或死锁
```