---
tags:
  - Areas/Coder/工程化思维
category: 思维
status: 加工
project:
application:
source:
---
[「记忆规律」与「知识体系建设」](../../03%20-%20Areas/「记忆规律」与「知识体系建设」.md)

```
L1：核心问题
如何在java中正确地控制并发资源，并排查线上并发故障？

L2：方案
├─ 保护共享变量：避免原子性、可见性、有序性问题
├─ 管理线程协作：让线程等待、唤醒、限流、排队
├─ 选择并发容器：减少手写锁、使用成熟并发数据结构
├─ 管理任务执行：用线程池控制资源和流量
├─ 编排异步任务：CompletableFuture 和 @Async 组织依赖关系
├─ 传递线程上下文：解决异步线程traceId、用户上下文丢失问题
├─ 定位线上故障：用监控、线程栈、链路解决线上问题

L3：机制
├─ 保护共享变量：happens-before规则 / volatile / CAS / Atomic 
├─ 管理线程协作：Synchronized / AQS / ReentrantLock / CountdownLatch / Semaphore / Condition 
├─ 选择并发容器：BlockingQueue / ConcurrentHashMap / CopyOnWriteArrayList
├─ 管理任务执行：ThreadPoolExecutor 参数、队列、拒绝策略、监控、关闭
├─ 编排异步任务：CompletableFuture 编排方法 + 自定义线程池 + 异常处理
├─ 传递线程上下文：显式传参 / Runnable 包装 / TaskDecorator / MDC
├─ 定位线上故障：top + jstack + jcmd + jstat + GC 日志 + 链路追踪 + 池化指标

L4：机制细节
├─ 保护共享变量：
	├─ happens-before：判断一个线程的写对于另一个线程的读是否可见；常见规则：volatile、锁、start
	├─ volatile：保证可见性，并限制特定重排序；典型场景是 DCL 单例中防止“引用先赋值、对象未初始化完”
	├─ CAS：乐观更新，先比较旧值再替换新值；低竞争快；高竞争会自旋消耗CPU
	├─ Atomic：CAS封装的各类变量赋值类：AtomicInteger、AtomicLong、AtomicReference、AtomicStampedReference
├─ 管理线程协作：
	├─ Synchronized：自动加锁 / 解锁，适合简单临界区；竞争时线程阻塞
	├─ AQS：JUC 底层基石；核心是：state + CAS + FIFO 队列 + park/unpark
	├─ ReentrantLock：
		手动 lock / unlock，支持 tryLock、超时、可中断、公平锁、多个 Condition；
		state表示可重入次数、exclusiveThread表示持有线程。
	├─ CountdownLatch：
		表示一个线程等待多个线程完成；
		state 表示剩余计数，countDown 减到 0 后唤醒 await 线程
	├─ Semaphore：
		表示控制同时执行某段代码的线程数量；
		state 表示许可证数量，acquire 减许可证，release 加许可证并唤醒等待线程。
	├─ Condition：一个 Lock 可创建多个条件队列，常用于 notEmpty / notFull；await 必须配合 while 重新检查条件，不能脱离lock
├─ 并发容器：
	├─ BlockingQueue：封装 Lock 和 Condition，解决生产者消费者问题；put/take 阻塞，offer/poll可超时
	├─ ConcurrentHashMap：JDK 1.7 是 Segment 分段锁；JDK 1.8 是数组 + CAS + synchronized 锁桶头
	├─ CopyOnWriteArrayList：读不加锁，写时复制，适合读多写少的快照类场景
├─ 线程池：
	├─ ThreadPoolExecutor 核心参数：coreSize、maxSize、aliveTime、unit、workQueue、threadFactory、handler
	├─ 执行流程：核心线程未满先建核心线程；否则入队；队列满后建非核心线程；最大线程满后拒绝。
	├─ 拒绝策略：AbortPolicy 抛异常，CallerRunsPolicy 调用方执行，DiscardPolicy 丢新任务，DiscardOldestPolicy 丢最老任务。
	├─ 生产禁止使用 Executors：Fixed / Single 无限队列；Cached 无限线程；
	├─ 参数配置：
		CPU密集型：接近CPU核数；
		IO密集型：线程数 = CPU 核数 * (1 + IO执行时间 / CPU执行时间)
	├─ 监控指标：poolSize（池内线程总数）、activeSize（活跃总数）、queueSize（队列长度）、completedTaskCount（已完成任务数）、taskExecutionTime（任务执行时间）、taskWaitTime（任务等待时间）
├─ 异步编排：
	├─ 创建任务：runAsync：无返回值；supplyAsync：有返回值
	├─ 处理"任务结果"：thenApply / thenAccept / thenRun
	├─ 接收"异步任务"并转化结果：thenCompose:串联 ； thenCombine：并行
	├─ 接收多个异步任务并返回结果：
		allOf：所有"异步任务'执行完毕后取结果
		anyof：任一"异步任务'执行完毕后取结果
	├─ 异常处理：
		exceptionally：类似于try-catch，要结果
		handle：类似于finally，要结果
		whenComplete；类似于finally，无结果
├─ 传递线程上下文:
	├─ ThreadLocal：每个线程保存独立副本，避免共享可变变量。
	├─ 内存泄漏根因：key 是 ThreadLocal 弱引用，value 是强引用；线程长期存活时 value 可能无法释放。
	├─ 使用原则：set 后必须 finally remove。
	├─ static ThreadLocal：static 只是共享同一个 key，不是共享同一个 value。
	├─ 跨线程传递：显式传参最清晰；包装 Runnable / Callable 或 Spring TaskDecorator 适合统一处理。
	├─ MDC：日志上下文，常用来把 traceId 打进日志；异步线程中要复制并清理 MDC。
├─ 线上问题排查:
	├─ top -Hp 找线程；jstack找线程死锁、细节
	├─ jstat -gc 找GC当前动态日志；GC dump 找线程任务细节
	├─ 池化指标监控线程池状态，结合 GC dump 找线程任务细节
	
L5：应用场景

场景 1: 高并发计数 -> 使用 AtomicInteger / LongAdder / CAS -> 看 CPU 是否因自旋升高，竞争高时避免单点 CAS 热点。
场景 2: 控制第三方接口并发 -> 使用 Semaphore -> 看下游 QPS、超时率、等待队列，许可证数要匹配下游承载。
场景 3: 主线程等多个子任务完成 -> 使用 CountDownLatch / CompletableFuture.allOf -> 检查是否有任务异常未 countDown 或 join 卡死。
场景 4: 生产者消费者 -> 使用 BlockingQueue -> 看队列长度、put/take 阻塞、消费耗时，队列必须有界。
场景 5: 订单异步通知 -> @Async + 自定义线程池 -> 看线程名、队列、拒绝次数、异常是否被 Future 吞掉。
场景 6: 日志 traceId 丢失 -> 使用 TaskDecorator / MDC 复制 -> 验证异步日志是否带 traceId，并确保 finally clear。
场景 7: 接口突然大量超时 -> 链路追踪 + 线程栈 + 池化指标 -> 区分是排队慢、执行慢、下游慢、锁竞争还是 GC。
```