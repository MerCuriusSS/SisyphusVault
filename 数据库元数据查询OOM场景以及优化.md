#Resources/coder/核心源码 


## OOM场景代码

```java
OOM环节：
1.一次性返回全量表对象集合，集合常驻堆内存；表数量越多，内存占用线性上涨
2.遍历所有表，逐个组装 TableMetaAgg，并全部保存在内存不释放（最高危）


//获取库中所有表数据
List<TableBasic> allTableBasic = queryAllTableBasic(dbId);
List<TableMetaAgg> fullAggList = new ArrayList<>();
//遍历表列表，获取对应字段、索引、约束，聚合成元数据对象
for(TableBasic table : allTableBasic) {
    List<Column> columns = queryColumns(table);
    List<Index> indexes = queryIndexes(table);
    List<Constraint> constraints = queryConstraints(table);
    TableMetaAgg agg = build(table, columns, indexes, constraints);
    fullAggList.add(agg); // 重点：所有agg持续持有，不丢弃
}


```

## OOM排查过程

```
1.报出OOM异常
2.下载dump文件
3.MAT解析
4.从静态引用链中找到大对象为「全量表对象集合」
5.分析得出 表数量庞多、与表关联的字段、索引、约束全部保存在内存中不释放
6.FullGC 失败直接OOM错误
```

## 解决OOM优化代码——游标分页&流式处理

```java
批次1：加载表1~100 → 逐个计算MD5、落库、比对
批次1处理结束 → 全部对象引用清空 → GC回收
批次2：加载表101~200 → 处理 → 回收
……
任意时刻内存最多只存在100张表的元数据
内存占用存在硬性上限，和总表数量无关
本地采集比对时 用 拿到相关数据
//========================================
int BATCH = 100;
for (db : dataSources) {
    String cursor = "";
    while(true) {
        List<String> batchTables = queryRemoteTableByCursor(db, cursor, BATCH);
        if(batchTables.isEmpty()) break;
        cursor = batchTables.getLast();

        // 复用当前批次表名，批量拉本地快照（自动同批次对齐） 
        Map<String,Snapshot> localSnapshotMap = snapshotDao.batchSelect(db, batchTables);

        for(String tbl : batchTables) {
            // 远端采集元数据、计算MD5
            agg = collectAndBuildMetaAgg(db, tbl);
            // 从map取出对应本地快照比对
            Snapshot old = localSnapshotMap.get(tbl);
            diffResult = compare(agg, old);
            saveSnapshot(agg);
            if(diffResult.hasChange()) saveDiff(diffResult);
        }
        // 批次结束释放引用
        batchTables = null;
        localSnapshotMap = null;
    }
}
```


### 结论：

- **游标分页**：以“table_name”作为游标条件，表名称改变时会出现漏查、错查，下个采集周期修正，当前业务场景可接受。
- **流式处理**：按批次查询，不追加列表，处理一批释放一批引用。