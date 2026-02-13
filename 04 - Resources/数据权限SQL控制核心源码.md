#Resources/coder/核心源码

## 最小化实践

- 步骤：注册拦截器->自定义`interceptor`->实现`intercet`方法->获取用户信息上下文->解析并拼接权限SQL->修改后SQL执行
- 核心代码：
	-  注册拦截器：
	```java
	@Configuration  
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
  
    /**  
     * 方式2：通过配置文件注册  
     *  
     * 在application.yml中配置：  
     *  
     * mybatis:     *   configuration:     *     interceptors:     *       - com.example.datapermission.interceptor.SimpleDataPermissionInterceptor     */
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


---
## ruoyi-vue-plus原实现

- 步骤：继承Plus拦截器->实现不同SQL类型拦截逻辑->SQL加工处理【检查数据权限->获取当前用户->跳过超级管理员->获取用户角色->解析并拼接权限SQL->执行修改后SQL】

### 1. 继承Plus拦截器+拦截逻辑：
```java
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

### 2. SQL加工处理
```java
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

### 3. 获取权限注解
```java
	  @Target({ElementType.METHOD, ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
public @interface DataPermission {  
  
    /**  
     * 数据权限配置数组，用于指定数据权限的占位符关键字和替换值  
     *  
     * @return 数据权限配置数组  
     */  
    DataColumn[] value();  
  
    /**  
     * 权限拼接标识符(用于指定连接语句的sql符号)  
     * 如不填 默认 select 用 OR 其他语句用 AND  
     * 内容 OR 或者 AND  
     */    String joinStr() default "";  
  
}
=========================================================
 //dataColumn内容描述映射哪个权限列
 //dataColumn个数描述单角色多维度 or 多角色多维度 的权限条件 
@@DataPermission({  
    @DataColumn(key = "deptName", value = "dept_id"),  
    @DataColumn(key = "userName", value = "create_by")  
})  
default Page<SysUserVo> selectPageUserList(Page<SysUser> page, Wrapper<SysUser> queryWrapper) {  
    return this.selectVoPage(page, queryWrapper);  
}
```

### 4. 解析并拼接权限SQL
- 步骤：拼接符→上下文→权限列过滤→角色 SQL 解析→结果拼接

```java
	  /**
* 使用select时 默认通过 or 连接多个角色权限SQL
**/
private String buildDataFilter(DataPermission dataPermission, boolean isSelect) {
    // 拼接符
    String joinStr = " " + (dataPermission.joinStr() != null ? dataPermission.joinStr() : (isSelect ? "OR" : "AND")) + " ";
    LoginUser user = DataPermissionHelper.getVariable("user");
    // 初始化上下文
    NullSafeStandardEvaluationContext context = new NullSafeStandardEvaluationContext("-1");
    context.addPropertyAccessor(new NullSafePropertyAccessor(context.getPropertyAccessors().get(0), "-1"));
    context.setBeanResolver(beanResolver);
    DataPermissionHelper.getContext().forEach(context::setVariable);

    Set<String> conditions = new HashSet<>();
    List<String> keys = new ArrayList<>();
    Map<DataColumn, Boolean> ignoreMap = new HashMap<>();

    // 权限列处理
    for (DataColumn col : dataPermission.value()) {
        if (CollUtil.contains(user.getMenuPermission(), col.permission())) {
            ignoreMap.put(col, true);
            continue;
        }
        for (int i = 0; i < col.key().length; i++) context.setVariable(col.key()[i], col.value()[i]);
        keys.addAll(Arrays.stream(col.key()).map(k -> "#" + k).toList());
    }

    // 角色权限处理
    for (RoleDTO role : user.getRoles()) {
        user.setRoleId(role.getRoleId());
        DataScopeType type = DataScopeType.findCode(role.getDataScope());
        if (type == DataScopeType.ALL) return "";

        boolean isSuccess = false;
        for (DataColumn col : dataPermission.value()) {
            if (ignoreMap.containsKey(col)) {
                conditions.add(joinStr + "1=1");
                isSuccess = true;
                continue;
            }
            if (!StringUtils.containsAny(type.getSqlTemplate(), keys.toArray(String[]::new)) || !StringUtils.containsAny(type.getSqlTemplate(), col.key())) continue;
            // 解析SQL
            String sql = DataPermissionHelper.ignore(() -> parser.parseExpression(type.getSqlTemplate(), parserContext).getValue(context, String.class));
            conditions.add(joinStr + sql);
            isSuccess = true;
        }
        if (!isSuccess) conditions.add(joinStr + type.getElseSql());
    }

    // 拼接结果
    return StreamUtils.join(conditions, Function.identity(), "").substring(joinStr.length());
}
```

### 5. 角色权限SQL替换（SpEL表达式）
```java
  
 public enum DataScopeType {  
  
    /**  
     * 全部数据权限  
     */  
    ALL("1", "", ""),  
  
    /**  
     * 自定数据权限  
     */  
    CUSTOM("2", " #{#deptName} IN ( #{@sdss.getRoleCustom( #user.roleId )} ) ", " 1 = 0 "),  
  
    /**  
     * 部门数据权限  
     */  
    DEPT("3", " #{#deptName} = #{#user.deptId} ", " 1 = 0 "),  
  
    /**  
     * 部门及以下数据权限  
     */  
    DEPT_AND_CHILD("4", " #{#deptName} IN ( #{@sdss.getDeptAndChild( #user.deptId )} )", " 1 = 0 "),  
  
    /**  
     * 仅本人数据权限  
     */  
    SELF("5", " #{#userName} = #{#user.userId} ", " 1 = 0 "),  
  
    /**  
     * 部门及以下或本人数据权限  
     */  
    DEPT_AND_CHILD_OR_SELF("6", " #{#deptName} IN ( #{@sdss.getDeptAndChild( #user.deptId )} ) OR #{#userName} = #{#user.userId} ", " 1 = 0 ");  
  
    private final String code;  
  
    /**  
     * SpEL 模板表达式，用于构建 SQL 查询条件  
     */  
    private final String sqlTemplate;  
  
    /**  
     * 如果不满足 {@code sqlTemplate} 的条件，则使用此默认 SQL 表达式  
     */  
    private final String elseSql;  
  
    /**  
     * 根据枚举代码查找对应的枚举值  
     *  
     * @param code 枚举代码  
     * @return 对应的枚举值，如果未找到则返回 null  
     */    public static DataScopeType findCode(String code) {  
        if (StringUtils.isBlank(code)) {  
            return null;  
        }  
        for (DataScopeType type : values()) {  
            if (type.getCode().equals(code)) {  
                return type;  
            }  
        }  
        return null;  
    }  
}

======================================================================
@RequiredArgsConstructor  
@Service("sdss")  
public class SysDataScopeServiceImpl implements ISysDataScopeService {  
  
    private final SysRoleDeptMapper roleDeptMapper;  
    private final SysDeptMapper deptMapper;  
  
    /**  
     * 获取角色自定义权限  
     *  
     * @param roleId 角色Id  
     * @return 部门Id组  
     */  
    @Cacheable(cacheNames = CacheNames.SYS_ROLE_CUSTOM, key = "#roleId", condition = "#roleId != null")  
    @Override  
    public String getRoleCustom(Long roleId) {  
        if (ObjectUtil.isNull(roleId)) {  
            return "-1";  
        }  
        List<SysRoleDept> list = roleDeptMapper.selectList(  
            new LambdaQueryWrapper<SysRoleDept>()  
                .select(SysRoleDept::getDeptId)  
                .eq(SysRoleDept::getRoleId, roleId));  
        if (CollUtil.isNotEmpty(list)) {  
            return StreamUtils.join(list, rd -> Convert.toStr(rd.getDeptId()));  
        }  
        return "-1";  
    }  
  
    /**  
     * 获取部门及以下权限  
     *  
     * @param deptId 部门Id  
     * @return 部门Id组  
     */  
    @Cacheable(cacheNames = CacheNames.SYS_DEPT_AND_CHILD, key = "#deptId", condition = "#deptId != null")  
    @Override  
    public String getDeptAndChild(Long deptId) {  
        if (ObjectUtil.isNull(deptId)) {  
            return "-1";  
        }  
        List<Long> deptIds = deptMapper.selectDeptAndChildById(deptId);  
        return CollUtil.isNotEmpty(deptIds) ? StreamUtils.join(deptIds, Convert::toStr) : "-1";  
    }  
  
}

=============================================================
	  //底层逻辑：
	  //将#{dataColumn.key} 替换成 DataColumn.value
	  //将#{user.xx} 替换成 用户信息(SysUser)具体变量
	  //将#{@sdss.getDeptAndChild()} 替换成对应@service("sdss")的方法调用
	  parser.parseExpression(type.getSqlTemplate(), parserContext).getValue(context, String.class)
====================================================================	
```
