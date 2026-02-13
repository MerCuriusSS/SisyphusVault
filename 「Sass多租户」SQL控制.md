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

#### 1.Redis全局新增「租户ID」前缀
```java
@AllArgsConstructor
public class TenantKeyPrefixHandler implements NameMapper {

    private final String keyPrefix;

    @Override
    public String map(String name) {
        // 全局Key，不添加租户前缀
        if (StringUtils.contains(name, GlobalConstants.GLOBAL_REDIS_KEY)) {
            return keyPrefix + name;
        }

        // 忽略租户模式，不添加租户前缀
        if (TenantHelper.isIgnore()) {
            return keyPrefix + name;
        }

        // 获取租户ID
        String tenantId = TenantHelper.getTenantId();
        if (StringUtils.isBlank(tenantId)) {
            return keyPrefix + name;
        }

        // 添加租户前缀
        return keyPrefix + tenantId + ":" + name;
    }

    @Override
    public String unmap(String name) {
        // 移除前缀的逆操作
        String prefix = keyPrefix;
        if (StringUtils.isNotBlank(prefix) && name.startsWith(prefix)) {
            name = name.substring(prefix.length());
        }

        // 移除租户前缀
        String tenantId = TenantHelper.getTenantId();
        if (StringUtils.isNotBlank(tenantId) && name.startsWith(tenantId + ":")) {
            return name.substring((tenantId + ":").length());
        }

        return name;
    }
}
```


## ⛪ 场景设想
- **场景 A**：在处理 [XXX] 代码逻辑时可以替代原有的 [YYY] 方法。
- **场景 B**：在进行 [ZZZ] 决策时，用来规避逻辑谬误。