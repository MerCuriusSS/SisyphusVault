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
	- SQL解析器（[JSQLParser](JSQLParser.md)）
- 「组件」流程概览图：[「数据权限SQL拼接」流程概览图](excalidraw/「数据权限SQL拼接」流程概览图.md)
- 「数据流转」流程图：[「数据权限SQL拼接」数据流转图](excalidraw/「数据权限SQL拼接」数据流转图.md)



## ⛪ 应用场景

🔴 最小化实践：
- 步骤：注册拦截器->自定义`interceptor`->实现`intercet`方法->获取用户信息上下文->解析并拼接权限SQL->修改后SQL执行
- 核心代码：
	-  注册拦截器：
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
		//DataPermission annotation = getDataPermission(ms);  
		//if (annotation == null) {  
		//    return invocation.proceed(); // 没有注解，直接执行  
		//}  
		  
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
	- 拦截器规则：
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
		//DataPermission annotation = getDataPermission(ms);  
		//if (annotation == null) {  
		//    return invocation.proceed(); // 没有注解，直接执行  
		//}  
		  
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
	-  生成权限SQL：
	```java
	private String buildPermissionSql(DataPermission annotation, LoginUser user) {  
    String column = annotation.column();    String tableAlias = annotation.tableAlias();    String fullColumn = tableAlias.isEmpty() ? column : tableAlias + "." + column;  
    switch (user.getRoleType()) {        case 1: // 超级管理员  
            return null;        case 2: // 部门管理员  
            return fullColumn + " = " + user.getDeptId();        case 3: // 普通用户  
            return fullColumn + " = " + user.getUserId();        default:            return "1 = 0"; // 无权限  
    }}
	```
	- 拼接SQL：
	```java
	private String appendPermissionSql(String originalSql, String permissionSql) {  
    if (originalSql.toUpperCase().contains("WHERE")) {        // 已有WHERE，使用AND连接  
        return originalSql + " AND (" + permissionSql + ")";    } else {        // 没有WHERE，添加WHERE子句  
        return originalSql + " WHERE " + permissionSql;    }}
	```

🔴 RuoYi-Vue-Plus 源码深度解析
- 步骤：
- 核心代码：
- 