
## 1. GC频率

- **指标**

```text
Young GC 频率
Mixed GC 频率
Full GC 频率
Concurrent GC 周期频率
```

- **目的**

判断 GC 发生得是否过于频繁，从而判断内存分配压力、Young 区压力、Old 区压力是否异常。

```text
日志内容示例

JDK 8 示例：

2026-06-05T10:00:01.000+0800: 10.000: [GC (Allocation Failure) [PSYoungGen: 65536K->8192K(76288K)] 120000K->70000K(251392K), 0.0100000 secs]
2026-06-05T10:00:03.000+0800: 12.000: [GC (Allocation Failure) [PSYoungGen: 65536K->8192K(76288K)] 128000K->78000K(251392K), 0.0120000 secs]
2026-06-05T10:00:05.000+0800: 14.000: [GC (Allocation Failure) [PSYoungGen: 65536K->8192K(76288K)] 136000K->86000K(251392K), 0.0110000 secs]

G1 示例：

[2026-06-05T10:00:01.000+0800][10.000s][info][gc] GC(10) Pause Young (Normal) (G1 Evacuation Pause) 512M->220M(1024M) 12.345ms
[2026-06-05T10:00:03.000+0800][12.000s][info][gc] GC(11) Pause Young (Normal) (G1 Evacuation Pause) 540M->235M(1024M) 13.000ms
```

```text
计算/体现方式

GC频率 = GC次数 / 观察时间窗口

例如：

10分钟内 Young GC 300次

Young GC频率 = 300 / 10 = 30次/分钟

也可以看两次GC之间的间隔：

10.000s -> 12.000s -> 14.000s

间隔约 2秒一次，即：

Young GC频率 ≈ 1次/2秒 ≈ 30次/分钟
```

```text
正常数值：

Young GC：
几秒一次到几十秒一次，通常比较常见。
如果每次耗时很短、Old区稳定、GC总耗时占比低，即使频率偏高也不一定是问题。

Full GC：
正常运行期最好为 0。
偶发 1 次需要结合启动、发布、定时任务、大批量加载判断。

Mixed GC：
G1 中周期性出现是正常的，重点看 Mixed GC 后 Old 是否下降、耗时是否可接受。
```

```text
异常数值以及理由：

Young GC 每秒多次：
说明对象分配压力很大，Young区很快被填满。
如果同时 GC耗时占比高，会影响吞吐和延迟。

Full GC 短时间多次：
严重异常。
Full GC 通常 STW 时间长，说明 Old区、Metaspace、晋升失败或显式 System.gc() 可能存在问题。

连续 Full GC：
非常严重。
可能接近 OOM，或者 Old区大量对象无法回收，应用基本处于 GC 风暴状态。

Mixed GC 很频繁但 Old 降不下来：
说明 G1 对 Old区回收效果差，Old区存活对象多，可能存在长期对象过多或堆容量不足。
```

---

## 2. GC耗时

- **指标**

单次 GC 暂停时间、平均耗时、最大耗时、P95/P99、GC 总耗时占比。

- **目的**

判断 GC 对业务延迟和吞吐的影响。

```text
日志内容示例

JDK 8 Young GC：

2026-06-05T10:00:01.000+0800: 10.000: [GC (Allocation Failure) [PSYoungGen: 65536K->8192K(76288K)] 120000K->70000K(251392K), 0.0100000 secs]

JDK 8 Full GC：

2026-06-05T10:00:06.500+0800: 16.500: [Full GC (Ergonomics) [PSYoungGen: 8192K->0K(76288K)] [ParOldGen: 150000K->100000K(174592K)] 158192K->100000K(251392K), [Metaspace: 45000K->45000K(106496K)], 0.8000000 secs]

G1：

[2026-06-05T10:00:01.000+0800][10.000s][info][gc] GC(10) Pause Young (Normal) (G1 Evacuation Pause) 512M->220M(1024M) 12.345ms
```

```text
计算/体现方式

JDK 8：

0.0100000 secs = 10ms
0.8000000 secs = 800ms

G1：

12.345ms = 12.345ms

常用统计：

平均GC耗时 = 总GC耗时 / GC次数

GC总耗时占比 = 观察窗口内GC总耗时 / 观察窗口总时长

例如：

10分钟内 GC总耗时 12秒

GC耗时占比 = 12 / 600 = 2%
```

```text
正常数值：

Young GC：
常规服务中，几十毫秒以内通常比较健康。
例如 5ms、10ms、30ms。

Full GC：
最好没有。
如果偶发一次且在可接受业务窗口内，需要结合业务判断。

GC总耗时占比：
低于 1% 通常较好。
1% - 5% 需要结合业务 SLA 判断。
```

```text
异常数值以及理由：

Young GC 经常超过 100ms：
可能影响接口延迟，尤其是低延迟服务。

Young GC 最大值达到几百毫秒甚至秒级：
说明存在长尾暂停，容易造成请求超时。

Full GC 超过 1s：
需要重点关注。
Full GC 是 Stop-The-World，1s 以上通常会直接影响在线服务。

Full GC 达到数秒、十几秒：
严重异常。
通常说明 Old 区对象多、回收成本高、堆太大、对象存活率高或 GC 算法不适合。

GC总耗时占比超过 5%：
说明应用有明显时间花在 GC 上，吞吐可能受影响。

GC总耗时占比超过 10%：
通常已经是严重 GC 问题。
```

---

## 3. 回收前后内存

- **指标**

每次 GC 前后的堆内存、Young 区、Old 区、Metaspace 使用变化。

- **目的**

判断 GC 是否有效、每次回收了多少内存、GC 后内存基线是否持续升高。

```text
日志内容示例

Young GC：

2026-06-05T10:00:01.000+0800: 10.000: [GC (Allocation Failure) [PSYoungGen: 65536K->8192K(76288K)] 120000K->70000K(251392K), 0.0100000 secs]

Full GC：

2026-06-05T10:00:06.500+0800: 16.500: [Full GC (Ergonomics) [PSYoungGen: 8192K->0K(76288K)] [ParOldGen: 150000K->100000K(174592K)] 158192K->100000K(251392K), [Metaspace: 45000K->45000K(106496K)], 0.8000000 secs]
```

```text
计算/体现方式

格式：

before->after(total)

Young区：

[PSYoungGen: 65536K->8192K(76288K)]

含义：

GC前 Young区使用 65536K
GC后 Young区使用 8192K
Young区容量 76288K

整个堆：

120000K->70000K(251392K)

含义：

GC前堆使用 120000K
GC后堆使用 70000K
堆容量 251392K

本次回收内存：

120000K - 70000K = 50000K

约 48.8MB
```

```text
正常数值：

Young GC：
GC 后 Young 区明显下降。
例如：

65536K -> 8192K

说明大部分短生命周期对象被回收。

Full GC：
如果是周期性积压，Full GC 后整个堆或 Old 区应有明显下降。

例如：

Old：1800M -> 900M

说明 Full GC 能回收较多对象。
```

```text
异常数值以及理由：

Young GC 后堆下降很少：

800M -> 760M
820M -> 780M
850M -> 810M

说明大量对象存活，可能晋升到 Old 区，Old 压力会增加。

Full GC 后堆下降很少：

3800M -> 3650M

说明大量对象仍然可达，GC 无法回收。
可能是内存泄漏、缓存无界增长、队列积压、长生命周期对象过多。

GC 后内存基线持续上涨：

700M -> 900M -> 1200M -> 1500M

说明应用保留下来的对象越来越多，需要进一步看 Old 区趋势和 heap dump。
```

---

## 4. Old区趋势

- **指标**

每次 GC 后 Old 区使用量变化，尤其是 GC 后基线是否持续上涨。

- **目的**

判断是否存在 Old 区压力、对象晋升过多、长期对象过多、缓存增长或内存泄漏。

```text
日志内容示例

JDK 8 有 Old 明细：

2026-06-05T10:00:06.500+0800: 16.500: [Full GC (Ergonomics) [PSYoungGen: 8192K->0K(76288K)] [ParOldGen: 150000K->100000K(174592K)] 158192K->100000K(251392K), 0.8000000 secs]

JDK 8 没有 Old 明细，但可推算：

10.000: [GC [PSYoungGen: 65536K->8192K(76288K)] 120000K->70000K(251392K)]
12.000: [GC [PSYoungGen: 65536K->8192K(76288K)] 128000K->78000K(251392K)]
14.000: [GC [PSYoungGen: 65536K->8192K(76288K)] 136000K->86000K(251392K)]

G1：

[10.000s][info][gc,heap] GC(10) Old regions: 180->195
[12.000s][info][gc,heap] GC(11) Old regions: 195->207
[14.000s][info][gc,heap] GC(12) Old regions: 207->220
```

```text
计算/体现方式

如果日志有 Old 明细：

[ParOldGen: 150000K->100000K(174592K)]

说明：

Old区 GC前 150000K
Old区 GC后 100000K
Old区容量 174592K

如果日志没有 Old 明细，可以用：

Old after ≈ Heap after - Young after

例如：

10.000s：

Heap after = 70000K
Young after = 8192K

Old after ≈ 70000K - 8192K = 61808K

12.000s：

Old after ≈ 78000K - 8192K = 69808K

14.000s：

Old after ≈ 86000K - 8192K = 77808K

Old区趋势：

61808K -> 69808K -> 77808K

说明 Old 区持续上涨。
```

```text
正常数值：

Old区 GC 后使用量在一个范围内波动：

800M -> 830M -> 790M -> 850M -> 810M

说明长期存活对象相对稳定。

Full GC 或 Mixed GC 后 Old 能明显下降：

1800M -> 900M

说明 Old 区仍然有较多可回收对象。
```

```text
异常数值以及理由：

Old区 GC 后持续上涨：

800M -> 1000M -> 1300M -> 1600M -> 1900M

说明长期存活对象越来越多，可能存在缓存增长、队列积压、批处理堆积或内存泄漏。

Full GC 后 Old 几乎不下降：

Old：3800M -> 3650M

说明大部分 Old 对象仍然被引用，GC 无法释放。
这是内存泄漏或长生命周期对象过多的强信号。

Old区快速接近上限：

Old使用 90% 以上并持续增长

容易触发 Full GC、Promotion failed、G1 to-space exhausted，甚至 OOM。
```

---

## 5. Full GC次数

- **指标**

Full GC 次数、频率、触发原因、Full GC 后回收效果。

- **目的**

判断 GC 问题严重程度，并定位是堆空间、晋升失败、Metaspace、System.gc 还是 GC 算法回收不及时导致。

```text
日志内容示例

JDK 8：

2026-06-05T10:01:01.000+0800: 72.345: [Full GC (Ergonomics) ...]
2026-06-05T10:02:15.000+0800: 146.789: [Full GC (Allocation Failure) ...]
2026-06-05T10:03:40.000+0800: 231.456: [Full GC (Metadata GC Threshold) ...]

CMS：

2026-06-05T10:05:00.000+0800: 300.000: [Full GC (Allocation Failure) ...
2026-06-05T10:05:10.000+0800: 310.000: [GC (CMS Initial Mark) ...
2026-06-05T10:05:15.000+0800: 315.000: [Full GC (concurrent mode failure) ...

G1：

[2026-06-05T10:10:00.000+0800][600.000s][info][gc] GC(120) Pause Full (G1 Compaction Pause) 3900M->3600M(4096M) 8.500s
```

```text
计算/体现方式

直接统计日志中的关键字：

Full GC
Pause Full
G1 Compaction Pause

例如：

10分钟内出现 5 次 Full GC

Full GC频率 = 5次 / 10分钟 = 0.5次/分钟

还要记录触发原因：

Allocation Failure
Metadata GC Threshold
System.gc()
Promotion failed
Concurrent mode failure
G1 Compaction Pause
to-space exhausted
```

```text
正常数值：

正常在线服务运行期最好为 0。

偶发 Full GC：
需要结合启动、发布、定时任务、手动运维操作判断。
如果发生在非业务高峰且没有影响，可继续观察。

启动阶段 Full GC：
有时可以接受，但仍需看是否由 Metaspace、类加载、堆参数不合理引起。
```

```text
异常数值以及理由：

短时间多次 Full GC：
严重异常，通常会造成明显业务停顿。

Full GC 耗时数秒以上：
会直接导致接口超时、线程阻塞、吞吐下降。

Full GC 后 Old 几乎不下降：
说明 GC 回收无效，可能是内存泄漏或长期对象过多。

Full GC 原因是 Metadata GC Threshold：
说明 Metaspace 达到阈值，方向应转向类加载、动态代理、ClassLoader 泄漏。

Full GC 原因是 System.gc()：
说明代码或第三方库显式触发 GC，需要定位调用方或配置禁用显式 GC。

Full GC 原因是 Promotion failed：
说明 Young GC 时对象要晋升到 Old，但 Old 空间不足。

CMS concurrent mode failure：
说明 CMS 并发回收来不及，Old 区在并发回收完成前已经满了。

G1 to-space exhausted / evacuation failure：
说明 G1 疏散对象时空间不足，通常和 Old 压力、堆余量不足、大对象有关。
```

---

## 6. 分配速率

- **指标**

Allocation Rate，单位时间内应用新分配对象的内存量。

- **目的**

判断应用是否存在过高的对象创建压力，以及 Young GC 频繁是否由分配过快导致。

```text
日志内容示例

JDK 8：

10.000: [GC [PSYoungGen: 65536K->8192K(76288K)] 120000K->70000K(251392K), 0.010 secs]
12.000: [GC [PSYoungGen: 65536K->8192K(76288K)] 128000K->78000K(251392K), 0.012 secs]
14.000: [GC [PSYoungGen: 65536K->8192K(76288K)] 136000K->86000K(251392K), 0.011 secs]

G1：

[10.000s][info][gc,heap] GC(10) Eden regions: 300->0(320)
[12.000s][info][gc,heap] GC(11) Eden regions: 310->0(330)
[14.000s][info][gc,heap] GC(12) Eden regions: 305->0(330)
```

```text
计算/体现方式

粗略算法：

分配速率 ≈ 两次 Young GC 之间新分配的内存 / 两次 Young GC 的时间间隔

JDK 8 简化估算：

Young GC 前 PSYoungGen 使用约 65536K
Young GC 间隔 2秒

分配速率 ≈ 65536K / 2s = 32768K/s ≈ 32MB/s

G1 估算：

假设 G1 region size = 1MB

GC(11) Eden regions: 310->0

说明 GC(10) 到 GC(11) 之间 Eden 大约分配了 310MB。

两次 GC 间隔 2秒：

分配速率 ≈ 310MB / 2s = 155MB/s
```

```text
正常数值：

几十 MB/s：
常见，很多后端服务都可能达到。

几百 MB/s：
高分配应用中也可能正常，需要结合 GC耗时、Old趋势、晋升速率判断。

分配速率高但晋升速率低、Old稳定、GC耗时低：
通常不是严重问题，说明大部分对象很快被回收。
```

```text
异常数值以及理由：

分配速率很高，同时 Young GC 每秒多次：
说明 Young 区很快被填满，GC频率会升高。

分配速率高，同时 GC总耗时占比高：
说明对象创建已经明显消耗 CPU 和暂停时间。

分配速率高，同时晋升速率也高：
说明大量对象活过 Young GC，被晋升到 Old 区，Old 压力会持续增加。

分配速率突然升高：
可能和流量突增、批处理、缓存加载、JSON/序列化、日志、临时集合、大对象创建有关。

GB/s 级分配：
通常需要重点关注对象创建热点，除非这是非常高吞吐、专门优化过的系统。
```

---

## 7. 晋升速率

- **指标**

Promotion Rate，对象从 Young 区晋升到 Old 区的速度。

- **目的**

判断 Young 区压力是否正在传导到 Old 区，是 GC 分析中判断 Old 压力来源的关键指标。

```text
日志内容示例

JDK 8：

10.000: [GC [PSYoungGen: 65536K->8192K(76288K)] 120000K->70000K(251392K)]
12.000: [GC [PSYoungGen: 65536K->8192K(76288K)] 128000K->78000K(251392K)]
14.000: [GC [PSYoungGen: 65536K->8192K(76288K)] 136000K->86000K(251392K)]

G1：

[10.000s][info][gc,heap] GC(10) Old regions: 180->195
[12.000s][info][gc,heap] GC(11) Old regions: 195->207
[14.000s][info][gc,heap] GC(12) Old regions: 207->220
```

```text
计算/体现方式

算法：

晋升量 ≈ 本次 GC 后 Old 使用量 - 上次 GC 后 Old 使用量

晋升速率 ≈ 晋升量 / 两次 GC 间隔时间

JDK 8 示例：

10.000s：

Old after ≈ Heap after - Young after
Old after ≈ 70000K - 8192K = 61808K

12.000s：

Old after ≈ 78000K - 8192K = 69808K

14.000s：

Old after ≈ 86000K - 8192K = 77808K

晋升量：

69808K - 61808K = 8000K
77808K - 69808K = 8000K

间隔 2秒：

晋升速率 = 8000K / 2s = 4000K/s ≈ 4MB/s

G1 示例：

假设 region size = 1MB

Old regions：

195 -> 207 -> 220

晋升量：

207 - 195 = 12MB
220 - 207 = 13MB

间隔 2秒：

晋升速率约 6MB/s 到 6.5MB/s
```

```text
正常数值：

晋升速率较低：
说明大部分对象在 Young 区死亡，Old 压力较小。

Old区 GC 后稳定：
说明晋升量在 Old 回收能力范围内。

分配速率高但晋升速率低：
这是比较健康的短生命周期对象模型。
```

```text
异常数值以及理由：

晋升速率持续较高：
说明大量对象跨过 Young GC 存活并进入 Old 区，会造成 Old 区持续增长。

晋升速率高 + Old区持续上涨：
说明 Old 区压力越来越大，后续容易出现 Mixed GC、Full GC 或 OOM。

晋升速率高 + Promotion failed：
说明对象晋升时 Old 区空间不足。

晋升速率高 + Young区/Survivor区较小：
可能是 Survivor 空间不足，导致对象过早晋升。

晋升速率突然升高：
可能和批处理、缓存构建、请求堆积、队列积压、大量中生命周期对象有关。
```

---

## 8. Metaspace使用

- **指标**

Metaspace 使用量、提交量、GC 前后变化、是否触发 Metadata GC Threshold。

- **目的**

判断类元数据是否增长异常，是否存在动态类生成、ClassLoader 泄漏或 Metaspace 参数过小。

```text
日志内容示例

JDK 8：

2026-06-05T10:03:40.000+0800: 231.456: [Full GC (Metadata GC Threshold) [PSYoungGen: 10240K->0K(76288K)] [ParOldGen: 120000K->118000K(174592K)] 130240K->118000K(251392K), [Metaspace: 98000K->97000K(112640K)], 0.7654321 secs]

JDK 9+：

[2026-06-05T10:00:01.000+0800][12.345s][info][gc,metaspace] GC(10) Metaspace: 75264K(76800K)->75264K(76800K) NonClass: 66000K(67072K)->66000K(67072K) Class: 9264K(9728K)->9264K(9728K)
```

```text
计算/体现方式

JDK 8：

[Metaspace: 98000K->97000K(112640K)]

含义：

GC前 Metaspace 使用 98000K
GC后 Metaspace 使用 97000K
当前容量/提交量 112640K

JDK 9+：

Metaspace: 75264K(76800K)->75264K(76800K)

含义：

GC前使用 75264K，提交量 76800K
GC后使用 75264K，提交量 76800K

重点看：

1. GC后 Metaspace 是否持续上涨
2. Full GC 原因是否为 Metadata GC Threshold
3. 类加载数量是否持续增长
```

```text
正常数值：

应用启动阶段 Metaspace 上涨：
正常，因为类在启动阶段大量加载。

应用稳定后 Metaspace 基本稳定：
正常。

GC 前后 Metaspace 变化不大，但整体水位稳定：
通常可接受。
```

```text
异常数值以及理由：

Metaspace 持续上涨：

200M -> 260M -> 330M -> 420M -> 550M

说明类元数据持续增加，可能存在动态类生成或 ClassLoader 泄漏。

频繁出现：

Full GC (Metadata GC Threshold)

说明 Metaspace 达到触发阈值，导致 Full GC。

Full GC 后 Metaspace 几乎不下降：

98000K -> 97900K

如果后续仍持续上涨，说明类或 ClassLoader 没有释放。

Metaspace 接近 MaxMetaspaceSize：
容易触发 Full GC 或 OOM: Metaspace。

常见原因：

动态代理类持续生成
CGLIB / ByteBuddy / Javassist 使用不当
Groovy / Janino / 脚本编译持续产生类
应用热部署导致 ClassLoader 泄漏
线程上下文 ClassLoader 持有旧应用
第三方框架生成类未释放
MaxMetaspaceSize 设置过小
```

---

可以。这个文件的核心可以简明概括为一句话：

> **分析 GC 日志不是逐个指标孤立判断，而是先确认 GC 是否影响业务，再通过 Full GC、Old 区趋势、回收效果、分配速率、晋升速率和 Metaspace，把问题归类为：短命对象分配过高、Old 区长期对象过多、内存泄漏、堆容量不足、Metaspace/ClassLoader 问题或显式 GC 问题。**

下面是整理后的方法论版本。

---

## GC 日志分析方法论

### 1. 先确认 GC 是否是业务问题的直接原因

分析 GC 日志的第一步，不是看参数，也不是看某一次 GC，而是先对齐业务问题时间点。

重点看：

```text
业务卡顿、RT 升高、接口超时、服务不可用的时间点
是否和 GC 日志中的暂停时间点重合
```

重点指标：

```text
GC频率
GC耗时
GC总耗时占比
最大暂停时间
Full GC次数
```

判断逻辑：

```text
如果业务慢的时间点没有明显 GC 暂停：
    GC 可能不是主因，应继续排查 CPU、锁、IO、数据库、线程池、网络等问题。

如果业务慢的时间点正好出现长时间 GC：
    GC 很可能是直接原因，需要继续分析 GC 类型、内存趋势和触发原因。
```

典型判断：

```text
接口 10:05:12 大量超时
GC 日志 10:05:12 出现 Full GC，暂停 8.5s

结论：
业务超时和 Full GC 时间重合，GC 是高优先级排查方向。
```

---

### 2. 先看有没有 Full GC

Full GC 是 GC 分析中的高优先级信号。

重点看：

```text
Full GC次数
Full GC频率
Full GC耗时
Full GC触发原因
Full GC后 Old 区是否下降
Full GC后 Heap 是否下降
```

判断逻辑：

```text
没有 Full GC：
    通常风险相对低，继续看 Young GC 是否频繁、是否影响吞吐。

偶发 Full GC：
    结合启动、发布、定时任务、批处理、业务峰值判断。

短时间频繁 Full GC：
    严重问题，通常说明 Old 区、Metaspace、晋升失败或显式 GC 存在异常。

连续 Full GC：
    非常严重，可能接近 OOM 或已经处于 GC 风暴。
```

Full GC 常见原因：

| Full GC 原因 | 初步含义 |
|---|---|
| `Allocation Failure` | 堆空间不足或 Old 区无法分配 |
| `Promotion failed` | Young 对象晋升 Old 失败 |
| `Metadata GC Threshold` | Metaspace 达到触发阈值 |
| `System.gc()` | 代码或第三方库显式触发 GC |
| `Concurrent mode failure` | CMS 并发回收来不及 |
| `G1 Compaction Pause` | G1 退化为 Full GC |
| `to-space exhausted` | G1 疏散空间不足 |

---

### 3. 用 Full GC 后 Old 区变化判断问题性质

Full GC 后 Old 区是否下降，是判断问题方向的关键。

#### 情况一：Full GC 后 Old 明显下降

例如：

```text
Full GC前 Old：3000M
Full GC后 Old：1200M
```

说明：

```text
Old 区里有大量可回收对象。
```

常见原因：

```text
周期性业务积压
批处理任务
缓存周期性刷新
流量峰值
堆容量偏小
Old 区回收触发偏晚
G1 Mixed GC 回收不及时
```

处理方向：

```text
结合业务时间点分析批任务或峰值流量
控制批处理大小
控制队列长度
优化缓存策略
适当调整堆容量或 G1 回收触发参数
```

#### 情况二：Full GC 后 Old 几乎不下降

例如：

```text
Full GC前 Old：3800M
Full GC后 Old：3650M
```

说明：

```text
大量对象仍然被引用，GC 无法回收。
```

这通常是更危险的情况。

常见原因：

```text
内存泄漏
无界缓存
集合持续增长
队列积压
ThreadLocal 未清理
ClassLoader 泄漏
大量长生命周期对象
全量数据一次性加载并长期持有
```

处理方向：

```text
抓 heap dump
用 MAT 分析 Dominator Tree
查看 Retained Heap 最大的对象
通过 Path to GC Roots 分析引用链
重点排查 Map、List、缓存、队列、ThreadLocal、业务上下文、ClassLoader
```

结论：

```text
Full GC 后 Old 降不下来时，不应优先调 Young 区。
因为根因通常不是 Young GC 太频繁，而是 Old 区对象仍然存活。
```

---

### 4. 用回收前后内存判断 GC 是否有效

GC 日志中常见格式：

```text
before->after(total)
```

例如：

```text
120000K->70000K(251392K)
```

含义：

```text
GC前堆使用：120000K
GC后堆使用：70000K
堆容量：251392K
本次回收：120000K - 70000K = 50000K
```

判断逻辑：

```text
GC后内存明显下降：
    回收有效，说明有较多对象可以释放。

GC后内存下降很少：
    回收效果差，说明大量对象仍然存活。

GC后内存基线持续上涨：
    应用保留下来的对象越来越多，需要重点看 Old 区趋势。
```

典型异常：

```text
Young GC后：
800M -> 760M
820M -> 780M
850M -> 810M

说明：
大量对象在 Young GC 后仍然存活，可能持续晋升到 Old 区。
```

```text
Full GC后：
3800M -> 3650M

说明：
Full GC 回收效果很差，高度怀疑长期对象过多或内存泄漏。
```

---

### 5. 用 Old 区趋势判断是否有长期风险

Old 区趋势比单次内存值更重要。

重点看：

```text
每次 GC 后 Old 区是否持续上涨
Full GC 或 Mixed GC 后 Old 区是否下降
Old 区是否接近上限
```

正常趋势：

```text
800M -> 830M -> 790M -> 850M -> 810M
```

说明：

```text
Old 区在稳定范围内波动，长期对象数量相对稳定。
```

异常趋势：

```text
800M -> 1000M -> 1300M -> 1600M -> 1900M
```

说明：

```text
Old 区 GC 后基线持续上涨。
可能存在长期对象增长、缓存增长、队列积压、批处理堆积或内存泄漏。
```

如果再叠加：

```text
Full GC 后 Old 仍然下降很少
```

则更偏向：

```text
内存泄漏或业务对象长期持有。
```

---

### 6. 用 Young GC 判断是否是短命对象分配压力

Young GC 频繁不一定严重，要结合耗时、Old 区和晋升速率一起看。

#### 情况一：Young GC 频繁，但 Old 稳定

指标组合：

```text
Young GC频繁
Young GC耗时短
Full GC很少或没有
Old区稳定
分配速率高
晋升速率低
```

判断：

```text
短生命周期对象很多。
对象创建压力大，但大部分对象在 Young 区被回收。
```

常见原因：

```text
JSON 序列化/反序列化
临时集合
字符串拼接
日志构造
循环内频繁创建对象
高 QPS 下正常对象创建
```

处理方式：

```text
如果 GC总耗时占比低，可以先观察。
如果 GC总耗时占比高，可以用 JFR 或 async-profiler 分析 allocation hot spots。
必要时优化对象创建或适当增大 Young 区。
```

#### 情况二：Young GC 频繁，Old 也持续上涨

指标组合：

```text
Young GC频繁
Old after 持续上涨
晋升速率高
Mixed GC 或 Full GC 开始变多
```

判断：

```text
Young 区压力正在传导到 Old 区。
大量对象活过 Young GC，并晋升到 Old。
```

常见原因：

```text
对象生命周期偏长
Survivor 区过小
对象过早晋升
批处理对象暂存时间长
缓存、队列、集合增长
请求堆积导致对象存活时间变长
```

处理方式：

```text
优先查对象为什么活得久。
重点看缓存、队列、批处理、中间结果集合、请求上下文。
必要时再调整 Survivor、Young 区或 MaxTenuringThreshold。
```

---

### 7. 用分配速率和晋升速率判断对象生命周期

这两个指标必须一起看。

#### 分配速率

含义：

```text
单位时间内应用新创建对象占用的内存量。
```

粗略计算：

```text
分配速率 ≈ 两次 Young GC 之间新分配的内存 / 两次 Young GC 的间隔
```

例如：

```text
Young GC前使用 65536K
两次 Young GC 间隔 2s

分配速率 ≈ 65536K / 2s = 32768K/s ≈ 32MB/s
```

#### 晋升速率

含义：

```text
对象从 Young 区晋升到 Old 区的速度。
```

粗略计算：

```text
晋升速率 ≈ 两次 GC 后 Old 使用量增量 / 两次 GC 间隔
```

例如：

```text
Old after：
61808K -> 69808K -> 77808K

每次增长：
8000K

间隔：
2s

晋升速率 ≈ 8000K / 2s = 4000K/s ≈ 4MB/s
```

#### 组合判断

| 指标组合 | 判断 | 优先动作 |
|---|---|---|
| 分配速率高，晋升速率低，Old 稳定 | 短命对象多，Young 区可回收 | 看 GC 占比，必要时优化分配热点 |
| 分配速率高，晋升速率高，Old 上涨 | 大量对象活过 Young GC | 查缓存、队列、批处理、对象生命周期 |
| 分配速率不高，但 Old 持续上涨 | 对象释放不了 | 抓 heap dump，查引用链 |
| 晋升速率高且 Promotion failed | Old 区空间不足以容纳晋升对象 | 查 Old 压力、堆余量、对象生命周期 |
| 分配速率突然升高 | 可能有流量峰值、批处理、序列化或大对象创建 | 结合业务时间点和 allocation profile |

---

### 8. 用 Metaspace 判断是否是类加载问题

如果日志中出现：

```text
Full GC (Metadata GC Threshold)
```

或者：

```text
Metaspace 持续上涨
```

就不能只盯 Java Heap。

重点看：

```text
Metaspace 使用量是否持续上涨
Full GC 原因是否是 Metadata GC Threshold
Full GC 后 Metaspace 是否下降
类加载数量是否持续增长
ClassLoader 是否无法释放
```

正常情况：

```text
应用启动阶段 Metaspace 上涨
稳定运行后 Metaspace 基本稳定
```

异常情况：

```text
200M -> 260M -> 330M -> 420M -> 550M
```

说明：

```text
类元数据持续增加，可能存在动态类生成或 ClassLoader 泄漏。
```

常见原因：

```text
动态代理类持续生成
CGLIB / ByteBuddy / Javassist 使用不当
Groovy / Janino / 脚本编译持续生成类
频繁热部署
ClassLoader 泄漏
MaxMetaspaceSize 设置过小
```

处理方式：

```text
jcmd <pid> VM.metaspace
jcmd <pid> VM.classloader_stats
检查动态代理、脚本编译、热部署、ClassLoader 引用链
```

---

## 常规分析决策树

可以把整个 GC 日志分析压缩成下面这个顺序：

```text
1. 业务问题是否和 GC 时间点重合？
   是：继续分析 GC。
   否：GC 可能不是主因，转查 CPU、锁、IO、数据库、线程池。

2. 是否有 Full GC？
   是：优先看 Full GC 原因、耗时、Old 回收效果、Metaspace。
   否：继续看 Young GC。

3. Full GC 后 Old 是否明显下降？
   是：可能是周期性积压、堆偏小、Old 回收触发晚。
   否：高度怀疑内存泄漏或长期对象过多。

4. Full GC 原因是否是 Metadata GC Threshold？
   是：查 Metaspace、类加载、ClassLoader。
   否：继续看 Old 区和晋升。

5. Young GC 是否频繁？
   是：看分配速率。
   否：看单次 GC 耗时和 Old 区空间。

6. 分配速率高但晋升速率低？
   是：短生命周期对象多，优化分配热点或适当调整 Young 区。
   否：继续看晋升速率和 Old 趋势。

7. 晋升速率高且 Old 持续上涨？
   是：对象生命周期偏长，查缓存、队列、批处理、Survivor 配置。
   否：继续看大对象、Humongous、堆容量、GC 参数。
```

---

## 常见问题与指标组合

| 问题现象 | 关键指标 | 初步判断 | 优先动作 |
|---|---|---|---|
| 接口偶发超时 | GC 最大耗时高、Full GC 耗时长 | GC 长暂停影响业务 | 对齐时间点，分析 Full GC 原因 |
| Young GC 很频繁 | GC 频率高、分配速率高 | 对象创建压力大 | 查 allocation hot spots |
| Young GC 频繁但 Old 稳定 | 分配速率高、晋升速率低 | 短命对象多 | 优化临时对象或调 Young 区 |
| Young GC 频繁且 Old 上涨 | 分配速率高、晋升速率高 | 中长期对象多 | 查缓存、队列、批处理 |
| Full GC 频繁 | Full GC 次数高、耗时高 | 严重 GC 问题 | 看 Full GC 原因和 Old 回收效果 |
| Full GC 后 Old 降不下来 | Old 趋势上涨、Full GC 回收差 | 泄漏或长期对象过多 | 抓 heap dump |
| Full GC 后 Old 能下降 | Full GC 回收有效 | 周期性积压或堆偏小 | 结合业务峰值，调堆或控制批量 |
| Metaspace 持续上涨 | Metaspace 趋势上涨、Metadata GC Threshold | 类加载问题 | 查 ClassLoader、动态代理 |
| G1 Mixed GC 后 Old 不下降 | Old regions 下降少、Mixed GC 频繁 | Old 存活率高 | 查长期对象、大对象、堆余量 |
| System.gc 触发 Full GC | Full GC 原因是 `System.gc()` | 显式 GC | 定位调用方或禁用显式 GC |

---

## 指标使用优先级

实际排查时，建议按这个优先级看：

```text
Full GC 频繁
> Full GC 后 Old 降不下来
> Old 区持续上涨
> 晋升速率高
> Young GC 频繁
> 分配速率高但晋升低
```

最危险的组合是：

```text
Full GC 频繁
Full GC 耗时长
Old 区持续上涨
Full GC 后 Old 下降很少
Heap 或 Metaspace 接近上限
```

这种情况通常不能靠简单调参解决，需要结合：

```text
heap dump
MAT Dominator Tree
Path to GC Roots
JFR
allocation profile
业务对象生命周期
缓存/队列/批处理逻辑
```

---

## GC 日志分析的核心方法论是：

```text
先用 GC 频率和 GC 耗时判断 GC 是否影响业务；
再用 Full GC 次数、原因和 Old 区回收效果判断问题严重程度；
再用回收前后内存和 Old 区趋势判断 GC 是否有效；
再用分配速率和晋升速率判断对象是短命还是长期存活；
最后用 Metaspace 判断是否是类加载或 ClassLoader 问题。
```

更简化地说：

```text
先确认影响，再判断严重程度；
先看 Full GC，再看 Old；
先看回收效果，再看对象生命周期；
最后决定是查代码、查 dump、调参数，还是查类加载。
```