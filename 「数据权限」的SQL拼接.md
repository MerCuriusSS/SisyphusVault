---
tags:
  - Areas/开发/javaWeb
  - Areas/开发/基础原理
category: 技术
status: 加工
project: "[[../../02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: 数据权限划分
source:
---


## 💥 核心定义

 🔴 概念：通过**非硬编码**的方式将**不同权限**角色的**查询范围**SQL**拼接**到业务SQL后面。【在SQL执行前，通过MyBatis拦截器自动在WHERE子句中添加基于用户角色的权限过滤条件】
 
 🔴 核心目的：将「业务逻辑」与「权限规则」进行**解耦**，以**非硬编码**、非侵入的**配置**方式实现**降低思考负担**。
【大白话：手动拼接权限SQL会导致100个mapper要改动100次的屎山代码，动作重复效率低，不如“一次配置，自动拼接”】


## 🔪 解构

🔴 「组件」流程概览图：[「数据权限SQL拼接」流程概览图](excalidraw/「数据权限SQL拼接」流程概览图.md)

🔴 「数据流转」流程图：[「数据权限SQL拼接」数据流转图](excalidraw/「数据权限SQL拼接」数据流转图.md)

🔴 底层逻辑：

🔴 失效边界：

## ⛪ 最小化实践

🔴 注册拦截器
```java
public class MyBatisConfig {  
  
    /**  
     * 注册数据权限拦截器  
     *  
     * 方式1：通过代码注册（推荐）  
     */  
    @Bean  
    public String registerDataPermissionInterceptor(SqlSessionFactory sqlSessionFactory) {  
        // 注册简化版拦截器  
        sqlSessionFactory.getConfiguration()  
            .addInterceptor(new SimpleDataPermissionInterceptor());  
  
        // 或者注册JSQLParser版拦截器  
        // sqlSessionFactory.getConfiguration()  
        //     .addInterceptor(new JsqlParserDataPermissionInterceptor());  
        return "DataPermissionInterceptor registered";  
    }
}
```

🔴 定义拦截器规则
```java
@Intercepts({  
    @Signature(  
        type = Executor.class,  
        method = "query",  
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}  
    )  
})  
public class SimpleDataPermissionInterceptor implements Interceptor {  
    @Override  
    public Object intercept(Invocation invocation) throws Throwable {  
        // 1. 获取原始SQL    
MappedStatement ms = (MappedStatement) invocation.getArgs()[0];  
        BoundSql boundSql = ms.getBoundSql(parameter);  
        String originalSql = boundSql.getSql();  
        // 2. 检查是否需要数据权限    
DataPermission annotation = getDataPermission(ms);  
        if (annotation == null) {  
            return invocation.proceed(); // 没有注解，直接执行    
}  
        // 3. 获取当前用户    
LoginUser user = UserContext.getUser();  
        if (user == null || user.isSuperAdmin()) {  
            return invocation.proceed(); // 超级管理员，直接执行    
}  
        // 4. 生成权限SQL    
String permissionSql = buildPermissionSql(annotation, user);  
        // 5. 拼接SQL    
String newSql = appendPermissionSql(originalSql, permissionSql);  
        // 6. 替换SQL并执行    
// ... 创建新的BoundSql和MappedStatement    
return invocation.proceed();  
    }  
}
```

🔴 权限SQL生成逻辑
```java  
private String buildPermissionSql(DataPermission annotation, LoginUser user) {  
    String column = annotation.column();    String tableAlias = annotation.tableAlias();    String fullColumn = tableAlias.isEmpty() ? column : tableAlias + "." + column;  
    switch (user.getRoleType()) {        case 1: // 超级管理员  
            return null;        case 2: // 部门管理员  
            return fullColumn + " = " + user.getDeptId();        case 3: // 普通用户  
            return fullColumn + " = " + user.getUserId();        default:            return "1 = 0"; // 无权限  
    }}  
```

🔴 SQL拼接逻辑（使用JSQLParser）
 -  能够正确解析复杂SQL结构
 - 支持子查询、JOIN、UNION等
 - 更安全，不会破坏SQL语法
```java  
private String appendPermissionSql(String originalSql, String permissionSql) {  
    if (originalSql.toUpperCase().contains("WHERE")) {        // 已有WHERE，使用AND连接  
        return originalSql + " AND (" + permissionSql + ")";    } else {        // 没有WHERE，添加WHERE子句  
        return originalSql + " WHERE " + permissionSql;    }}  
```