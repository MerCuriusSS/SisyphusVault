---
tags:
  - Areas/Coder/javaWeb
  - Areas/Coder/基础原理
category: 技术
status: 加工
project: "[[../02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: 数据权限划分
source:
---


## 💥 核心定义

 🔴 概念：控制不同用户 / 角色只能看到自己权限范围内的数据
 
 🔴 核心价值：将「业务逻辑」与「权限规则」进行**解耦**，以**非硬编码**、非侵入的**配置**方式实现**降低思考负担**


## 🔪我的见解：
>什么要记录它？它解决了什么我以前解决不了的问题？

🔴 **RABC权限**模型只解决了「能不能看」的问题，无法解决「看多少」的问题。

🔴 **数据权限SQL控制**模型解决了「看多少」的问题——个人数据、部门数据、全公司数据。


## ⚡️ 我的重构

🔴 底层逻辑：
核心业务表都新增权限范围字段`部门id`、`创建人id`,在 SQL 执行前，通过拦截器自动在 WHERE 子句中添加基于用户角色的权限过滤条件

🔴 核心组件及流程图：
- 核心组件
	-  拦截器（interceptor）：在 SQL 执行前进行拦截
	- 用户上下文（threadLocal）：从当前登录用户中获取其角色、部门、数据范围等权限信息
	- SQL解析器（[JSQLParser](JSQLParser.md)）
- 「组件」流程概览图：[「数据权限SQL拼接」流程概览图](../excalidraw/「数据权限SQL拼接」流程概览图.md)
- 「数据流转」流程图：[「数据权限SQL拼接」数据流转图](../excalidraw/「数据权限SQL拼接」数据流转图.md)

## 🧻 实践应用

🔴 最小化实践&RuoYi-Vue-Plus 源码深度解析：[数据权限SQL控制源码](../04%20-%20Resources/数据权限SQL控制源码.md)

🔴 「RuoYi-Vue-Plus 源码」与「最小化实践 源码」比对

| 特性        | JSQLParser版                         | RuoYi原实现                         |
| --------- | ----------------------------------- | -------------------------------- |
| 拦截器使用     | mybatis配置文件注册                       | 继承mybatis-plus拦截基类，内封装JSQLParser |
| SQL拼接     | 只支持外层Select、不支持子查询、union、join复杂查询操作 | 自带递归调用，支持复杂查询                    |
| 权限规则      | 硬编码                                 | SpEL表达式替换                        |
| 角色权限      | 只支持一种角色的判定                          | 支持多个角色取并集（or 连接）                 |
| 自定义权限（函数） | 不能                                  | 支持                               |
|           |                                     |                                  |
🔴 实战演练——通过完整CRUD体现数据权限
### 流程
```  
用户请求 → Controller（功能权限校验） → Service → Mapper（数据权限注解）  
                                                    ↓                                          MyBatis拦截器拦截SQL  
                                                    ↓                                          数据权限处理器生成过滤条件  
                                                    ↓                                          自动拼接到原始SQL  
                                                    ↓                                          执行带权限过滤的SQL  
```

### Controller权限校验
```java
/**  
 * 获取项目详细信息  
 *  
 * 说明：只能查看有权限的项目详情，否则返回null  
 */@SaCheckPermission("demo:project:query")  
@GetMapping("/{id}")  
public R<ProjectVo> getInfo(@NotNull(message = "主键不能为空")  
                             @PathVariable Long id) {  
    return R.ok(projectService.queryById(id));  
}  
  
/**  
 * 新增项目  
 */  
@SaCheckPermission("demo:project:add")  
@Log(title = "项目管理", businessType = BusinessType.INSERT)  
@PostMapping()  
public R<Void> add(@Validated(AddGroup.class) @RequestBody ProjectBo bo) {  
    return toAjax(projectService.insertByBo(bo));  
}  
  
/**  
 * 修改项目  
 *  
 * 说明：数据权限会限制只能修改有权限的项目  
 * 如果尝试修改无权限的项目，更新操作会返回0（影响行数为0）  
 */  
@SaCheckPermission("demo:project:edit")  
@Log(title = "项目管理", businessType = BusinessType.UPDATE)  
@PutMapping()  
public R<Void> edit(@Validated(EditGroup.class) @RequestBody ProjectBo bo) {  
    return toAjax(projectService.updateByBo(bo));  
}  
  
/**  
 * 删除项目  
 *  
 * 说明：数据权限会限制只能删除有权限的项目  
 */  
@SaCheckPermission("demo:project:remove")  
@Log(title = "项目管理", businessType = BusinessType.DELETE)  
@DeleteMapping("/{ids}")  
public R<Void> remove(@NotEmpty(message = "主键不能为空")  
                      @PathVariable Long[] ids) {  
    return toAjax(projectService.deleteWithValidByIds(List.of(ids), true));  
}  
  
/**  
 * 查询即将到期的项目  
 *  
 * 说明：返回N天内到期的项目，受数据权限限制  
 */  
@SaCheckPermission("demo:project:list")  
@GetMapping("/expiring/{days}")  
public R<List<ProjectVo>> getExpiringSoonProjects(  
    @PathVariable Integer days) {  
    return R.ok(projectService.queryExpiringSoonProjects(days));  
}
```


### service实现
```java
/**  
 * 查询项目详情  
 */  
@Override  
public ProjectVo queryById(Long id) {  
    // 此方法会自动应用数据权限，用户只能查看有权限的项目  
    return baseMapper.selectVoById(id);  
}

/**  
 * 新增项目  
 */  
@Override  
public Boolean insertByBo(ProjectBo bo) {  
    Project add = MapstructUtils.convert(bo, Project.class);  
    validEntityBeforeSave(add);  
    boolean flag = baseMapper.insert(add) > 0;  
    if (flag) {  
        bo.setId(add.getId());  
    }  
    return flag;  
}

/**  
 * 修改项目  
 */  
@Override  
public Boolean updateByBo(ProjectBo bo) {  
    Project update = MapstructUtils.convert(bo, Project.class);  
    validEntityBeforeSave(update);  
    // 更新操作会自动应用数据权限（使用AND连接）  
    // 用户只能更新自己部门且自己负责的项目  
    return baseMapper.updateById(update) > 0;  
}

/**  
 * 查询即将到期的项目  
 */  
@Override  
public List<ProjectVo> queryExpiringSoonProjects(Integer days) {  
    // 自定义SQL查询也会自动应用数据权限  
    return baseMapper.selectExpiringSoonProjects(days);  
}


```

### mapper（数据权限注解）
```java
/**  
 * 更新项目（带数据权限）  
 *  
 * 数据权限规则：使用 AND 连接，确保用户只能更新自己部门且自己负责的项目  
 * 这比查询更严格，防止越权修改  
 */  
@Override  
@DataPermission(value = {  
    @DataColumn(key = "deptName", value = "dept_id"),  
    @DataColumn(key = "userName", value = "owner_id")  
}, joinStr = "AND")  
int updateById(@Param(Constants.ENTITY) Project entity);

/**  
 * 删除项目（带数据权限）  
 *  
 * 数据权限规则：使用 AND 连接，确保用户只能删除自己部门且自己负责的项目  
 */  
@Override  
@DataPermission(value = {  
    @DataColumn(key = "deptName", value = "dept_id"),  
    @DataColumn(key = "userName", value = "owner_id")  
}, joinStr = "AND")  
int deleteById(Long id);

/**  
 * 自定义查询：查询即将到期的项目（带数据权限）  
 *  
 * 演示：自定义SQL也可以应用数据权限  
 */  
@DataPermission({  
    @DataColumn(key = "deptName", value = "dept_id"),  
    @DataColumn(key = "userName", value = "owner_id")  
})  
List<ProjectVo> selectExpiringSoonProjects(@Param("days") Integer days);
```

```xml
<!-- 通用查询结果列 -->  
<sql id="selectProjectVo">  
    select p.id, p.project_name, p.project_code, p.dept_id, p.owner_id,  
           p.status, p.budget, p.start_date, p.end_date, p.description,  
           p.create_time, p.update_time,  
           d.dept_name,  
           u.nick_name as owner_name  
    from project p  
    left join sys_dept d on p.dept_id = d.dept_id  
    left join sys_user u on p.owner_id = u.user_id  
</sql>  
  
<!-- 查询即将到期的项目 -->  
<select id="selectExpiringSoonProjects" resultType="org.dromara.demo.domain.vo.ProjectVo">  
    <include refid="selectProjectVo"/>  
    where p.del_flag = '0'  
      and p.status = '0'  
      and p.end_date between CURDATE() and DATE_ADD(CURDATE(), INTERVAL #{days} DAY)  
    order by p.end_date asc  
</select>
```

### 数据权限处理器（handler）生成过滤条件
```java
public Expression getSqlSegment(){
	// 获取当前登录用户信息  
	Long userId = LoginHelper.getUserId();  
	Long deptId = LoginHelper.getDeptId();  
	String roleKey = LoginHelper.getUserType(); // 获取用户角色  
	  
	// 获取数据权限配置  
	String[] keys = dataColumn.key();  
	String[] values = dataColumn.value();  
	  
	// 构建SQL过滤条件  
	Expression expression = 。。。
	
	return expression;
}
```

## ⛪ 应用场景

🔴 组织架构、人事管理类：
### 1. 人员管理：
- 角色：超级管理员、公司HR、部门HR、普通员工
- 权限规则
	- 超级管理员/公司HR：所有员工信息
	- 部门HR：部门所有员工信息
	- 普通员工：本人信息
- 管控维度：部门ID+员工ID
### 2.岗位管理：
- 角色：超级管理员、组织架构管理员、部门负责人
- 权限规则：
	- 超级管理员/组织架构管理员：公司所有岗位
	- 部门负责人：部门所有岗位
- 管控维度：部门ID

🔴 办公、审批类：
### 请假/报销/审批：
- 角色：超级管理员、财务人员、部门审批人、普通员工
- 权限规则：
	- 超级管理员：查看 / 导出全公司所有审批单数据；
	- 财务人员：查看 / 操作所有报销类审批单（含全部门），负责审核付款
	- 部门审批人：仅查看 / 审批本部门员工的所有审批单
	- 普通员工：仅查看 / 操作本人发起的审批单，可查看审批进度。
- 管控维度：
	- **创建人 ID**（本人）+ **部门 ID**（审批范围）+ 业务类型（报销 / 请假）

🔴 客户、销售类：
### 1.客户管理
- 角色：
	- 销售总监、销售经理、一线销售
- 权限规则：
	- 销售总监：查看 / 操作全公司所有客户数据
	- 销售经理：仅查看 / 操作本区域的所有客户数据
	- 一线销售：仅查看 / 操作本人跟进的客户数据
- 管控维度：
	- 客户归属人员ID+区域ID
### 2.订单管理
- 角色：
	- 采购总监、部门采购专员、仓库管理员
- 权限规则：
	- 采购总监：所有采购信息
	- 部门采购专员：部门采购信息
	- 仓库管理员：仅查看需入库的采购订单数据
- 管控维度：
	- 部门ID+人员ID+订单状态

🔴 Saas多租户管理：
- **角色划分**：SaaS 平台管理员、租户超级管理员、租户部门管理员、租户普通员工
- **数权管控规则**：
    1. **租户间绝对隔离**：A 租户的所有用户，无法查看 / 操作 B 租户的任何数据（平台级核心数权）；
    2. **租户内部精细化管控**：租户超级管理员可查看 / 操作本租户全量数据，租户部门管理员仅查看本部门数据，普通员工仅查看本人数据。
    
- **核心管控维度**：**租户 ID**（平台级，基础隔离）+ 部门 ID / 创建人 ID（租户内部，精细化管控）

