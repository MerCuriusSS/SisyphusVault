---
tags:
  - Areas/Coder/javaWeb
  - Areas/Coder/基础原理
category: 技术
status: 加工
project: "[[02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: 租户逻辑隔离
source:
---
>**笔记不是为了复述书本，而是为了**“存档当下的自己”。如果你的笔记里没有你的思考痕迹、痛苦经历和选择理由**，它就只是一份毫无生命力的说明书，自然无法在未来唤醒你的认知***


## 💥 核心结论
>核心定义是什么? 核心价值在哪里?

### 🟣 核心定义：以「**租户**」（组织/机构）为单位，使「数据、权限、配置、资源」使用相互**独立**，但整体**复用**同一套代码逻辑和部署实例。
### 🟣 核心价值：一套代码，多租户独立使用，隔离且复用。

## 🔪 我的见解
>什么要记录它？它解决了什么我以前解决不了的问题？

### 🟣 痛点：
#### 1. 人力不足，维护成本高昂：
- 多客户多部署实例，资源使用、监控成本上升。
-  一个BUG修复需要为所有客户系统都修复一次，人力不足。
#### 2. 定制化需求难管理：
- 不同用户用不同系统版本，新功能难同步上线。
- 不同定制化需求让代码分支泛滥，难管理。

### 🟣 解决方案：
#### 1. 租户id逻辑隔离，共用一套代码、一套部署实例，所有租户共用资源。
#### 2. 共用一套代码，版本统一、定制化逻辑通过配置实现，维护代码统一。

## ⚡️ 我的重构
>它的底层逻辑是什么？（尝试用最简单的类比解释给外行听）它的结构是什么?

### 🟣 底层逻辑：为大部分业务表、权限配置表统一添加`tenant_id`字段作为租户标识，在SQL执行前通过拦截器自动在WHERE 子句中添加基于租户ID的过滤条件。

>**添加tenant_id基准：该表数据是否「归属于特定租户」 + 是否「需要在租户间隔离」**

### 🟣 结构：

#### 1.核心组件
- 租户上下文（tenantContext）
- 租户拦截器（tenantInterceptor）& 租户处理器（tenantHandler）
- 用户登录状态上下文（SaToken）
- 缓存组件（Redis）
#### 2.「组件」流程概览图：[「多租户隔离」流程概览图](excalidraw/「多租户隔离」流程概览图.md)
#### 3.「数据流转」流程图：[「多租户隔离」数据流转图](excalidraw/「多租户隔离」数据流转图.md)

## 🚀 实践应用：

### 🟣 最小化实践：
#### 1.租户上下文（tenantContext）
```java
public class TenantContext {
    private static ThreadLocal<String> tenantId = new ThreadLocal<>();
    private static ThreadLocal<Boolean> ignore = new ThreadLocal<>();

    public static void setTenantId(String id) {
        tenantId.set(id);
    }

    public static String getTenantId() {
        return tenantId.get();
    }

    public static void setIgnore(Boolean flag) {
        ignore.set(flag);
    }

    public static Boolean isIgnore() {
        return Boolean.TRUE.equals(ignore.get());
    }

    public static void clear() {
        tenantId.remove();
        ignore.remove();
    }
}
```

#### 2. 租户拦截器（tenantInterceptor）
```java
@Intercepts({@Signature(type = Executor.class, method = "query", ...)})
public class TenantInterceptor implements Interceptor {

    private List<String> excludeTables = Arrays.asList("sys_tenant");

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 1. 获取原始SQL
        String originalSql = boundSql.getSql();

        // 2. 检查是否需要租户隔离
        if (TenantContext.isIgnore()) {
            return invocation.proceed(); // 忽略模式
        }

        String tenantId = TenantContext.getTenantId();
        if (tenantId == null) {
            return invocation.proceed(); // 没有租户ID
        }

        // 3. 检查表是否在排除列表中
        if (isExcludeTable(originalSql)) {
            return invocation.proceed();
        }

        // 4. 拼接租户条件
        String newSql = appendTenantCondition(originalSql, tenantId);

        // 5. 执行新SQL
        return executeNewSql(newSql);
    }

    private String appendTenantCondition(String sql, String tenantId) {
        if (sql.toUpperCase().contains("WHERE")) {
            return sql + " AND tenant_id = '" + tenantId + "'";
        } else {
            return sql + " WHERE tenant_id = '" + tenantId + "'";
        }
    }
}
```

### 🟣 ruoyi 租户原实现
>特点：MyBatis-Plus + Redis隔离 + 缓存隔离

#### 1.租户配置
- 自动装配开启「租户过滤」功能
- 租户拦截器&处理器
- Redis全局key添加「租户ID前缀」配置
```java
@EnableConfigurationProperties(TenantProperties.class)
@AutoConfiguration(after = {RedisConfig.class})
@ConditionalOnProperty(value = "tenant.enable", havingValue = "true")
public class TenantConfig {

    /**
     * 多租户插件
     */
    @Bean
    public TenantLineInnerInterceptor tenantLineInnerInterceptor(
            TenantProperties tenantProperties) {
        return new TenantLineInnerInterceptor(
            new PlusTenantLineHandler(tenantProperties)
        );
    }

    /**
     * 多租户 Redis Key 前缀处理器
     */
    @Bean
    public RedissonAutoConfigurationCustomizer tenantRedissonCustomizer(
            RedissonProperties redissonProperties) {
        return config -> {
            TenantKeyPrefixHandler nameMapper =
                new TenantKeyPrefixHandler(redissonProperties.getKeyPrefix());

            SingleServerConfig singleServerConfig = config.useSingleServer();
            if (ObjectUtil.isNotNull(singleServerConfig)) {
                singleServerConfig.setNameMapper(nameMapper);
            }

            ClusterServersConfig clusterServersConfig = config.useClusterServers();
            if (ObjectUtil.isNotNull(clusterServersConfig)) {
                clusterServersConfig.setNameMapper(nameMapper);
            }
        };
    }

    /**
     * 多租户缓存管理器
     */
    @Primary
    @Bean
    public CacheManager tenantCacheManager() {
        return new TenantSpringCacheManager();
    }

    /**
     * 多租户鉴权 DAO 实现
     */
    @Primary
    @Bean
    public SaTokenDao tenantSaTokenDao() {
        return new TenantSaTokenDao();
    }
}
```

#### 2.Redis缓存全局添加「租户ID前缀」逻辑
```java
public class TenantSpringCacheManager extends RedisCacheManager {

    public TenantSpringCacheManager(RedisCacheWriter cacheWriter,
                                    RedisCacheConfiguration defaultCacheConfiguration) {
        super(cacheWriter, defaultCacheConfiguration);
    }

    @Override
    public Cache getCache(String name) {
        // 全局缓存，不添加租户前缀
        if (StringUtils.contains(name, GlobalConstants.GLOBAL_REDIS_KEY)) {
            return super.getCache(name);
        }

        // 忽略租户模式，不添加租户前缀
        if (TenantHelper.isIgnore()) {
            return super.getCache(name);
        }

        // 获取租户ID
        String tenantId = TenantHelper.getTenantId();
        if (StringUtils.isBlank(tenantId)) {
            return super.getCache(name);
        }

        // 添加租户前缀
        return super.getCache(tenantId + ":" + name);
    }
}
```

#### 3.SaToken 中「用户登录状态KEY」的「global前缀」全局修改
```java
public class TenantSaTokenDao extends SaTokenDaoRedis {

    /**
     * 重写所有Key操作方法，添加全局前缀
     */

    @Override
    public String get(String key) {
        return super.get(GlobalConstants.GLOBAL_REDIS_KEY + key);
    }

    @Override
    public void set(String key, String value, long timeout) {
        super.set(GlobalConstants.GLOBAL_REDIS_KEY + key, value, timeout);
    }

    @Override
    public void delete(String key) {
        super.delete(GlobalConstants.GLOBAL_REDIS_KEY + key);
    }

    // ... 其他方法类似
}
```

#### 4.TenantHelper租户上下文逻辑：

##### 1.获取当前租户ID
```java
public static String getTenantId() {
    if (!isEnable()) {
        return null;
    }

    // 优先获取动态租户ID
    String tenantId = TenantHelper.getDynamic();
    if (StringUtils.isBlank(tenantId)) {
        // 其次从登录用户获取
        tenantId = LoginHelper.getTenantId();
    }

    return tenantId;
}
```

`获取优先级：动态租户（ThreadLocal/Redis） > 登录用户租户ID`
##### 2. 忽略租户隔离
```java
/**
 * 开启忽略租户（开关）
 */
private static final ThreadLocal<Boolean> IGNORE = new ThreadLocal<>();

public static void enableIgnore() {
    IGNORE.set(true);
}

public static void disableIgnore() {
    IGNORE.remove();
}

public static boolean isIgnore() {
    return Convert.toBool(IGNORE.get(), false);
}

/**
 * 忽略租户隔离执行
 */
public static void ignore(Runnable handle) {
    enableIgnore();
    try {
        handle.run();
    } finally {
        disableIgnore();
    }
}

public static <T> T ignore(Supplier<T> handle) {
    enableIgnore();
    try {
        return handle.get();
    } finally {
        disableIgnore();
    }
}
```

##### 3.动态切换租户
```java
/**
 * 临时动态租户（ThreadLocal）
 */
private static final ThreadLocal<String> TEMP_DYNAMIC_TENANT = new ThreadLocal<>();

/**
 * 动态租户Key前缀（Redis）
 */
private static final String DYNAMIC_TENANT_KEY = "global:dynamicTenant";

/**
 * 设置动态租户
 *
 * @param tenantId 租户ID
 * @param global   是否全局（true-存储到Redis，false-仅ThreadLocal）
 */
public static void setDynamic(String tenantId, boolean global) {
    if (!isEnable()) {
        return;
    }

    // 未登录或非全局模式，使用ThreadLocal
    if (!LoginHelper.isLogin() || !global) {
        TEMP_DYNAMIC_TENANT.set(tenantId);
        return;
    }

    // 全局模式，存储到Redis
    String cacheKey = DYNAMIC_TENANT_KEY + ":" + LoginHelper.getUserId();
    RedisUtils.setCacheObject(cacheKey, tenantId);

    // 同时存储到SaToken会话中（避免重复查询Redis）
    SaHolder.getStorage().set(cacheKey, tenantId);
}

/**
 * 获取动态租户
 */
public static String getDynamic() {
    if (!isEnable()) {
        return null;
    }

    // 1. 优先从ThreadLocal获取
    String tenantId = TEMP_DYNAMIC_TENANT.get();
    if (StringUtils.isNotBlank(tenantId)) {
        return tenantId;
    }

    // 2. 未登录，返回null
    if (!LoginHelper.isLogin()) {
        return null;
    }

    // 3. 从SaToken会话获取（请求级缓存）
    String cacheKey = DYNAMIC_TENANT_KEY + ":" + LoginHelper.getUserId();
    tenantId = (String) SaHolder.getStorage().get(cacheKey);
    if (StringUtils.isNotBlank(tenantId)) {
        return tenantId;
    }

    // 4. 从Redis获取（持久化存储）
    tenantId = RedisUtils.getCacheObject(cacheKey);
    if (StringUtils.isNotBlank(tenantId)) {
        // 缓存到SaToken会话
        SaHolder.getStorage().set(cacheKey, tenantId);
        return tenantId;
    }

    return null;
}

/**
 * 清除动态租户
 */
public static void clearDynamic() {
    if (!isEnable()) {
        return;
    }

    // 清除ThreadLocal
    TEMP_DYNAMIC_TENANT.remove();

    if (!LoginHelper.isLogin()) {
        return;
    }

    // 清除Redis和SaToken会话
    String cacheKey = DYNAMIC_TENANT_KEY + ":" + LoginHelper.getUserId();
    RedisUtils.deleteObject(cacheKey);
    SaHolder.getStorage().delete(cacheKey);
}

/**
 * 在动态租户中执行
 */
public static void dynamic(String tenantId, Runnable handle) {
    setDynamic(tenantId);
    try {
        handle.run();
    } finally {
        clearDynamic();
    }
}

public static <T> T dynamic(String tenantId, Supplier<T> handle) {
    setDynamic(tenantId);
    try {
        return handle.get();
    } finally {
        clearDynamic();
    }
}
```

`动态租户存储层次：ThreadLocal（临时） > SaToken会话（请求级） > Redis（持久化）`
#### 5.拦截处理器逻辑：
```java
@Slf4j
@AllArgsConstructor
public class PlusTenantLineHandler implements TenantLineHandler {

    private final TenantProperties tenantProperties;

    /**
     * 获取租户ID
     * 返回Expression对象，MyBatis-Plus会自动将其添加到SQL中
     */
    @Override
    public Expression getTenantId() {
        String tenantId = TenantHelper.getTenantId();
        if (StringUtils.isBlank(tenantId)) {
            log.error("无法获取有效的租户id -> Null");
            return new NullValue();
        }
        // ��回租户ID的字符串值
        return new StringValue(tenantId);
    }

    /**
     * 判断表是否需要忽略租户隔离
     *
     * @param tableName 表名
     * @return true-忽略，false-不忽略
     */
    @Override
    public boolean ignoreTable(String tableName) {
        String tenantId = TenantHelper.getTenantId();

        // 判断是否有租户
        if (StringUtils.isNotBlank(tenantId)) {
            // 从配置文件读取排除表列表
            List<String> excludes = tenantProperties.getExcludes();

            // 内置排除表（代码生成器相关）
            List<String> tables = ListUtil.toList(
                "gen_table",
                "gen_table_column"
            );
            tables.addAll(excludes);

            // 判断当前表是否在排除列表中
            return StringUtils.equalsAnyIgnoreCase(tableName,
                tables.toArray(new String[0]));
        }

        // 没有租户ID，忽略所有表
        return true;
    }
}
```

#### 6.调用方式
##### 1.忽略租户&动态切换租户（定时任务）
```java
@Scheduled(cron = "0 0 2 * * ?")
public void dailyTask() {
    // 查询所有启用的租户
    List<SysTenant> tenants = TenantHelper.ignore(() -> {
        return tenantMapper.selectList(
            new LambdaQueryWrapper<SysTenant>()
                .eq(SysTenant::getStatus, "0")
        );
    });

    // 遍历每个租户执行任务
    for (SysTenant tenant : tenants) {
        TenantHelper.dynamic(tenant.getTenantId(), () -> {
            // 在当前租户下执行任务
            log.info("执行租户{}的定时任务", tenant.getTenantName());
            // ... 业务逻辑
        });
    }
}
```

### 🟣 「最小实践」与「ruoyi实现」比对


| 特性   | 最小化实践            | ruoyi实现 | 核心逻辑                                 |     |
| ---- | ---------------- | ------- | ------------------------------------ | --- |
| 用户使用 | 依赖登录用户，切换用户会登录失效 | 不依赖登录态  | 1）Redis专门保留动态租户ID<br>2）Redis全局存Token |     |
|      |                  |         |                                      |     |


## ⛪ 场景设想
- **场景 A**：在处理 [XXX] 代码逻辑时可以替代原有的 [YYY] 方法。
- **场景 B**：在进行 [ZZZ] 决策时，用来规避逻辑谬误。