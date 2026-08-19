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
L1：核心问题
如何构建JVM内存、GC和线上OOM排查的定位模型？

L2：方案
├─ 明确JVM的内存区域
├─ 对象如何创建、分配、流转
├─ GC 如何判断对象是否可回收
├─ GC 算法和收集器如何影响吞吐、停顿和碎片
├─ 如何通过GC日志判断趋势
├─ 通过 dump 找到静态引用链并落实到业务修复

L3：机制
├─ 内存区域 -> 区分“线程运行”还是“对象存储”区域
├─ 对象分配流转 -> 对象创建过程 & 对象的流转
├─ GC 回收 -> GC的可达性分析
├─ GC 算法 -> 标记-复制；标记-清除；标记-整理；分代收集
├─ GC 收集器 -> Serial类、Parallel类、ParNew、CMS、G1
├─ GC 日志 -> GC频率、耗时、Full GC 原因、回收前后 old变化以及发展趋势
├─ Dump 分析 -> Retained Heap 和 Path to GC Roots 找 “谁占内存大” 和 “谁还引用它”

L4：机制细节
├─ 线程运行区域
	├─ 程序计数器：线程私有、记录下一条字节码指令；无OOM
	├─ 虚拟机栈：线程私有、存栈帧、局部变量、操作数栈，常见stackoverflow
	├─ 本地方法栈：线程私有、服务native调用
├─ 对象存储区域
	├─ 堆：线程共享、存对象实例和数组、常见为java heap Error
	├─ 方法区/元空间：线程共享、存类元数据、字段元信息、方法元信息、常量池，常见 metaspace
├─ 对象创建过程&流转
	├─ 创建过程：类加载 -> 分配内存 -> 默认零值 -> 设置对象头 -> 执行构造/初始化代码 -> 返回引用
	├─ 流转：新对象优先存Eden区域 -> Eden不足触发 minor GC 
		-> 存活进入survivor区 -> 多轮存活到 Old区 
		-> 对象太大进入 old区 -> old不足或者晋升失败触发 Full GC
├─ GC的可达性分析
	├─ 可达性分析：取决于能否从 GC Roots 出发到达。
	├─ GC Roots：局部变量、静态变量、常量、活跃线程
	├─ 引用类型：强引用可达不回收；软引用内存不足才回收；弱引用GC即回收；虚引用用于释放监听。
├─ GC 算法
	├─ 标记-复制：复制存活对象，需要额外空间
	├─ 标记-清除：直接清除垃圾，缺点是产生碎片
	├─ 标记-整理：移动存活对象到一边，减少碎片但成本高
	├─ 分代收集：年轻代用复制算法；老年代用标记清除；需要处理跨代引用
├─ GC 收集器：
	├─ Serial：单线程，简单，适合小堆
	├─ Parallel：关注吞吐量；适合批处理
	├─ CMS：并发标记清除；停顿短但有碎片，并发失败需用serial old 兜底
	├─ G1：整堆管理、逻辑分代、局部增量回收、适合大堆和停顿可控
├─ GC 日志：(涵盖GC时间、GC内容、分代信息)
	├─ GC日志命令（JDK8、JDK9）
	├─ 关注内容：GC时间/频率；FULL GC 原因和耗时；GC后Old的是否下降和积压趋势
├─ Dump 分析：
	├─ dump文件
	├─ MAT分析路径：leak suspect -> dominator tree -> retained heap 排序 -> Path to GC Roots 找出引用

L5：应用场景
Java heap space/GC overhead limit exceeded
Metaspace OOM
StackOverflowError
线程池无界队列导致 OOM
```