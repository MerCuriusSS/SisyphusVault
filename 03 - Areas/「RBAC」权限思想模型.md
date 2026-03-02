---
tags:
  - Areas/Coder/基础原理
  - Areas/Coder/javaWeb
category: 技术
status: 加工
project: "[[../02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: 权限划分
source:
---
>**笔记不是为了复述书本，而是为了**“存档当下的自己”。如果你的笔记里没有你的思考痕迹、痛苦经历和选择理由**，它就只是一份毫无生命力的说明书，自然无法在未来唤醒你的认知***


## 💥 核心结论
>核心定义是什么? 核心价值在哪里?
### 🔴 核心定义：基于角色的权限控制（Based Role Access Control）的简称，是一种解决实现人员权限如何分配的设计思想。
### 🔴 核心价值：简化用户权限分配的逻辑，解决为用户分配权限时多次出现重复的问题。【解耦用户与权限关系，避免权限配置爆炸】

## 🔪 我的见解
>什么要记录它？它解决了什么我以前解决不了的问题？

### 🔴直接为每一个人员分配权限时，人员定位相似，权限会「**重复分配**」，移除时也需要全数据遍历删除，操作非常「**繁琐**」**。

## ⚡️ 我的重构
>它的底层逻辑是什么（尝试用最简单的类比解释给外行听）？它的结构是什么?

### 🔴 底层逻辑：只为角色分配权限，人员关联对应一个或多个角色，接收请求时，根据人员所属角色具备的权限决定是否允许访问。

### 🔴 核心组件
-  **sys_user（用户表）**

| 字段名       | 描述  |
| --------- | --- |
| user_id   |     |
| user_name | 用户名 |
| password  | 密码  |

-  **sys_role（角色表）**

| 字段名       | 描述  |
| --------- | --- |
| role_id   |     |
| role_name | 角色名 |

-  **sys_menu（权限表）**
- 权限标识符**命名规则**：模块:子模块:操作
	- system:user:list（菜单/列表查询）
	- system:user:query（查询详情）
	- system:user:edit（编辑）
	- system:user:remove（移除）
	- system:user:add（添加）

| 字段名       | 描述      | 作用                       |
| --------- | ------- | ------------------------ |
| menu_id   |         |                          |
| parent_id | 父菜单id   | 建立树形结构                   |
| menu_name | 菜单名称    |                          |
| menu_type | 菜单类型    | 定义权限分配类型（M:目录；C:菜单；F：按钮） |
| perm      | 权限标识字符串 | 控制能否访问标识：前端路由守卫以及后端校验    |
```sql
-- 一级菜单
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('1', '系统管理', 'M', '');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('6', '租户管理', 'M', '');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('2', '系统监控', 'M', '');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('3', '系统工具', 'M', '');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('4', 'PLUS官网', 'M', '');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('5', '测试菜单', 'M', '');

-- 二级菜单
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('100',  '用户管理',     'C', 'system:user:list');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('101',  '角色管理',     'C', 'system:role:list');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('102',  '菜单管理',     'C', 'system:menu:list');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('103',  '部门管理',     'C', 'system:dept:list');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('104',  '岗位管理',     'C', 'system:post:list');

-- 按钮权限（F类型）
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('1001', '用户查询', 'F', 'system:user:query');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('1002', '用户新增', 'F', 'system:user:add');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('1003', '用户修改', 'F', 'system:user:edit');
insert into sys_menu (menu_id, menu_name, menu_type, perms) values('1004', '用户删除', 'F', 'system:user:remove');
```
- **sys_user_role（用户-角色关联表）**
- **sys_menu_role（权限-角色关联表）**
- **权限认证框架（Sa-Token）**：用户鉴权

### 流程图
- 「组件」流程概览图：[「RABC」流程概览图](../excalidraw/「RABC」流程概览图.md)
- 「数据流转」流程图：[「RABC」数据流转图](../excalidraw/「RABC」数据流转图.md)

## ⌚️ 实践应用

### 1. 为用户分配角色：
```java
private void insertUserRole(Long userId, Long[] roleIds, boolean clear){
	//roleId判空校验
	
	//超级管理员角色非法勾选校验
	
	//清除原有绑定
	if (clear) {  
    userRoleMapper.delete(new LambdaQueryWrapper<SysUserRole>().eq(SysUserRole::getUserId, userId));  
}
	//批量插入用户-角色关联
	List<SysUserRole> list = StreamUtils.toList(roleList,  
    roleId -> {  
        SysUserRole ur = new SysUserRole();  
        ur.setUserId(userId);  
        ur.setRoleId(roleId);  
        return ur;  
    });  
userRoleMapper.insertBatch(list)
}
```

### 2. 角色分配权限
```java
/**  
 * 新增角色菜单信息  
 *  
 * @param role 角色对象  
 */  
private int insertRoleMenu(SysRoleBo role) {  
    int rows = 1;  
    // 新增用户与角色管理  
    List<SysRoleMenu> list = new ArrayList<>();  
    for (Long menuId : role.getMenuIds()) {  
        SysRoleMenu rm = new SysRoleMenu();  
        rm.setRoleId(role.getRoleId());  
        rm.setMenuId(menuId);  
        list.add(rm);  
    }  
    if (CollUtil.isNotEmpty(list)) {  
        rows = roleMenuMapper.insertBatch(list) ? list.size() : 0;  
    }  
    return rows;  
}
```

### 3. 登录时查询权限集合
```java
public class SaPermissionImpl implements StpInterface{
/**  
 * 获取菜单权限列表  
 */
@Override  
public List<String> getPermissionList(Object loginId, String loginType){
	permissionService.getMenuPermission();
}

/**  
 * 获取角色权限列表  
 */  
@Override  
public List<String> getRoleList(Object loginId, String loginType) {
	permissionService.getRolePermission();
}
}
```

```java
/**  
 * 获取角色数据权限  
 *  
 * @param userId  用户id  
 * @return 角色权限信息  
 */  
@Override  
public Set<String> getRolePermission(Long userId) {  
    Set<String> roles = new HashSet<>();  
    // 管理员拥有所有权限  
    if (LoginHelper.isSuperAdmin(userId)) {  
        roles.add(TenantConstants.SUPER_ADMIN_ROLE_KEY);  
    } else {  
        roles.addAll(roleService.selectRolePermissionByUserId(userId));  
    }  
    return roles;  
}  
  
/**  
 * 获取菜单数据权限  
 *  
 * @param userId  用户id  
 * @return 菜单权限信息  
 */  
@Override  
public Set<String> getMenuPermission(Long userId) {  
    Set<String> perms = new HashSet<>();  
    // 管理员拥有所有权限  
    if (LoginHelper.isSuperAdmin(userId)) {  
        perms.add("*:*:*");  
    } else {  
        perms.addAll(menuService.selectMenuPermsByUserId(userId));  
    }  
    return perms;  
}
```

### 4.鉴权验证(@SaCheckPermission)
```java
/**  
 * 拦截后比较 SaCheckPermission的权限标识，存在则放行，否则抛出403错误 
 */  
@SaCheckPermission("system:menu:query")  
@GetMapping("/treeselect")  
public R<List<Tree<Long>>> treeselect(SysMenuBo menu) {  
 
}
```

## ⛪ 场景设想

### 🔴 总结：适用于一切需要多角色、多岗位、多部门、多层级协同工作的系统

- **私营企业：**
	- **后台管理系统（OA、ERP、数据平台、运营平台）**
	- **审批类系统**：（OA、报销、请假）
	- **电商后台：（商家、运营、客服、财务）**
	- **Saas租户系统（租户隔离）**
- **公营企业：**
	- **医疗系统：（医生、护士、药师、收费员）**
	- **金融系统：（柜员、主管、风控、财务、审计）**
	- **政府/事业单位系统（分级权限）**