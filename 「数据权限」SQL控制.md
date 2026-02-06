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
- 步骤：继承Plus拦截器->实现不同SQL类型拦截逻辑->SQL加工处理【检查数据权限->获取当前用户->跳过超级管理员->获取用户角色->解析并拼接权限SQL->执行修改后SQL】
- 核心代码：
	- 继承Plus拦截器+拦截逻辑：
	- ```java
	  public class PlusDataPermissionInterceptor extends BaseMultiTableInnerInterceptor implements InnerInterceptor {  
  
    private final PlusDataPermissionHandler dataPermissionHandler = new PlusDataPermissionHandler();  
  
    /**  
     * [查询类型SQL]，检查并处理数据权限相关逻辑  
     *  
     * @param executor      MyBatis 执行器对象  
     * @param ms            映射语句对象  
     * @param parameter     方法参数  
     * @param rowBounds     分页对象  
     * @param resultHandler 结果处理器  
     * @param boundSql      绑定的 SQL 对象  
     * @throws SQLException 如果发生 SQL 异常  
     */  
    @Override  
    public void beforeQuery(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {  
        // 检查是否需要忽略数据权限处理  
        if (InterceptorIgnoreHelper.willIgnoreDataPermission(ms.getId())) {  
            return;  
        }  
        // 检查是否缺少有效的数据权限注解  
        if (dataPermissionHandler.invalid()) {  
            return;  
        }  
        // 解析 sql 分配对应方法  
        PluginUtils.MPBoundSql mpBs = PluginUtils.mpBoundSql(boundSql);  
        //parserSingle内部会调用processSelect方法完成where插入操作  
        mpBs.sql(parserSingle(mpBs.sql(), ms.getId()));  
    }  
  
    /**  
     * [删改类型SQL]，检查并处理更新和删除操作的数据权限相关逻辑  
     *  
     * @param sh                 MyBatis StatementHandler 对象  
     * @param connection         数据库连接对象  
     * @param transactionTimeout 事务超时时间  
     */  
    @Override  
    public void beforePrepare(StatementHandler sh, Connection connection, Integer transactionTimeout) {  
        PluginUtils.MPStatementHandler mpSh = PluginUtils.mpStatementHandler(sh);  
        MappedStatement ms = mpSh.mappedStatement();  
        // 获取 SQL 命令类型（增、删、改、查）  
        SqlCommandType sct = ms.getSqlCommandType();  
  
        // 只处理更新和删除操作的 SQL 语句  
        if (sct == SqlCommandType.UPDATE || sct == SqlCommandType.DELETE) {  
            if (InterceptorIgnoreHelper.willIgnoreDataPermission(ms.getId())) {  
                return;  
            }  
            // 检查是否缺少有效的数据权限注解  
            if (dataPermissionHandler.invalid()) {  
                return;  
            }  
            PluginUtils.MPBoundSql mpBs = mpSh.mPBoundSql();  
            //parserMulti会调用processUpdate或者processDelete完成where插入操作  
            mpBs.sql(parserMulti(mpBs.sql(), ms.getId()));  
        }  
    }  
  
    /**  
     * 处理 SELECT 查询语句中的 WHERE 条件（在基类层面已实现递归调用逻辑）  
     *  
     * @param select SELECT 查询对象  
     * @param index  查询语句的索引  
     * @param sql    查询语句  
     * @param obj    WHERE 条件参数  
     */  
    @Override  
    protected void processSelect(Select select, int index, String sql, Object obj) {  
        if (select instanceof PlainSelect) {  
            this.setWhere((PlainSelect) select, (String) obj);  
        } else if (select instanceof SetOperationList setOperationList) {  
            List<Select> selectBodyList = setOperationList.getSelects();  
            selectBodyList.forEach(s -> this.setWhere((PlainSelect) s, (String) obj));  
        }  
    }  
  
    /**  
     * 处理 UPDATE 语句中的 WHERE 条件  
     *  
     * @param update UPDATE 查询对象  
     * @param index  查询语句的索引  
     * @param sql    查询语句  
     * @param obj    WHERE 条件参数  
     */  
    @Override  
    protected void processUpdate(Update update, int index, String sql, Object obj) {  
        Expression sqlSegment = dataPermissionHandler.getSqlSegment(update.getWhere(), false);  
        if (null != sqlSegment) {  
            update.setWhere(sqlSegment);  
        }  
    }  
  
    /**  
     * 处理 DELETE 语句中的 WHERE 条件  
     *  
     * @param delete DELETE 查询对象  
     * @param index  查询语句的索引  
     * @param sql    查询语句  
     * @param obj    WHERE 条件参数  
     */  
    @Override  
    protected void processDelete(Delete delete, int index, String sql, Object obj) {  
        Expression sqlSegment = dataPermissionHandler.getSqlSegment(delete.getWhere(), false);  
        if (null != sqlSegment) {  
            delete.setWhere(sqlSegment);  
        }  
    }  
  
    /**  
     * 设置 SELECT 语句的 WHERE 条件  
     *  
     * @param plainSelect       SELECT 查询对象  
     * @param mappedStatementId 映射语句的 ID  
     */    protected void setWhere(PlainSelect plainSelect, String mappedStatementId) {  
        Expression sqlSegment = dataPermissionHandler.getSqlSegment(plainSelect.getWhere(), true);  
        if (null != sqlSegment) {  
            plainSelect.setWhere(sqlSegment);  
        }  
    }  
  
    /**  
     * 构建表达式，用于处理表的数据权限  
     *  
     * @param table        表对象  
     * @param where        WHERE 条件表达式  
     * @param whereSegment WHERE 条件片段  
     * @return 构建的表达式  
     */  
    @Override  
    public Expression buildTableExpression(Table table, Expression where, String whereSegment) {  
        // 只有新版数据权限处理器才会执行到这里  
        final MultiDataPermissionHandler handler = (MultiDataPermissionHandler) dataPermissionHandler;  
        return handler.getSqlSegment(table, where, whereSegment);  
    }  
}
	  ```
	- SQL加工处理
	- ```java
	  public Expression getSqlSegment(Expression where, boolean isSelect) {
    // 1.获取权限与用户
    DataPermission dataPermission = getDataPermission();
    LoginUser currentUser = DataPermissionHelper.getVariable("user");
    currentUser = LoginHelper.getLoginUser();
    DataPermissionHelper.setVariable("user", currentUser);

    // 2.管理员放行
    if (LoginHelper.isSuperAdmin() || LoginHelper.isTenantAdmin()) {
        return where;
    }

    // 3.构建并拼接数据权限SQL
    String dataFilterSql = buildDataFilter(dataPermission, isSelect);
    Expression expression = CCJSqlParserUtil.parseExpression(dataFilterSql);
    ParenthesedExpressionList<Expression> parenthesis = new ParenthesedExpressionList<>(expression);
    return new AndExpression(where, parenthesis);
}
}
	  ```
	- 解析并拼接权限SQL
	- ```java
	  
	  ```