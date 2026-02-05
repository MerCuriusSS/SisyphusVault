---
tags:
  - Areas/开发/javaWeb
  - Areas/开发/基础原理
category: 技术
status: 加工
project: "[[02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: 数据权限划分
source:
---


## 💥 核心定义

 🔴 概念：控制不同用户 / 角色只能看到自己权限范围内的数据
 
 🔴 核心目的：将「业务逻辑」与「权限规则」进行**解耦**，以**非硬编码**、非侵入的**配置**方式实现**降低思考负担**【在SQL执行前，通过拦截器自动在WHERE子句中添加基于用户角色的权限过滤条件】


## 🔪 解构

🔴 核心思想：
- **AOP / 拦截器**：在 SQL 执行前进行拦截。
- **上下文获取**：从当前登录用户中获取其角色、部门、数据范围等权限信息。
- **权限规则生成**：根据不同角色/用户生成不同SQL条件。

🔴 核心组件及流程图：
- 核心组件
	- 拦截器（interceptor）
	- 用户上下文（threadLocal）
	- SQL解析器（JSQLParser）
- 「组件」流程概览图：[「数据权限SQL拼接」流程概览图](excalidraw/「数据权限SQL拼接」流程概览图.md)
- 「数据流转」流程图：[「数据权限SQL拼接」数据流转图](excalidraw/「数据权限SQL拼接」数据流转图.md)



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

🔴 数据权限注解
```java
@Target(ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
public @interface DataPermission {  
  
    /**  
     * 需要过滤的字段名  
     * 例如：dept_id、create_by等  
     */  
    String column() default "dept_id";  
  
    /**  
     * 表别名（可选）  
     * 例如：u.dept_id 中的 "u"  
     */    String tableAlias() default "";  
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
    Object[] args = invocation.getArgs();  
    MappedStatement ms = (MappedStatement) args[0];  
    Object parameter = args[1];  
  
    BoundSql boundSql = ms.getBoundSql(parameter);  
    String originalSql = boundSql.getSql();  
  
    // 检查是否需要数据权限控制  
    DataPermission dataPermission = getDataPermission(ms);  
    if (dataPermission == null) {  
        return invocation.proceed();  
    }  
  
    LoginUser currentUser = UserContext.getUser();  
    if (currentUser == null || currentUser.isSuperAdmin()) {  
        return invocation.proceed();  
    }  
  
    // 使用JSQLParser解析和修改SQL  
    try {  
        Statement statement = CCJSqlParserUtil.parse(originalSql);  
  
        if (statement instanceof Select) {  
            Select select = (Select) statement; 
            //拼接SQL 
            processSelect(select, dataPermission, currentUser);  
  
            String newSql = select.toString();  
  
            // 创建新的BoundSql  
            BoundSql newBoundSql = new BoundSql(  
                ms.getConfiguration(),  
                newSql,  
                boundSql.getParameterMappings(),  
                parameter  
            );  
  
            // 创建新的MappedStatement  
            MappedStatement newMs = copyMappedStatement(ms, new BoundSqlSqlSource(newBoundSql));  
            args[0] = newMs;  
        }  
    } catch (Exception e) {  
        // SQL解析失败，记录日志但不影响原SQL执行  
        System.err.println("数据权限SQL解析失败: " + e.getMessage());  
    }  
  
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
 -  能够正确解析复杂SQL结构、支持子查询、JOIN、UNION等
 - 本质：将完整的SQL结构拆解成不同对象
```java  
/**  
 * 处理SELECT语句  
 */  
private void processSelect(Select select, DataPermission dataPermission, LoginUser user) {  
  
    if (select.getSelectBody() instanceof PlainSelect) {  
        PlainSelect plainSelect = (PlainSelect) select.getSelectBody();  
        Expression where = plainSelect.getWhere();  
  
        // 构建权限条件  
        Expression permissionCondition = buildPermissionCondition(dataPermission, user);  
        if (permissionCondition != null) {  
            if (where != null) {  
                // 已有WHERE条件，使用AND连接  
                AndExpression andExpression = new AndExpression(where, permissionCondition);  
                plainSelect.setWhere(andExpression);  
            } else {  
                // 没有WHERE条件，直接设置  
                plainSelect.setWhere(permissionCondition);  
            }  
        }  
    }  
}
```