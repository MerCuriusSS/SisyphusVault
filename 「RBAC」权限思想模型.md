---
tags:
  - Areas/Coder/基础原理
  - Areas/Coder/javaWeb
category: 技术
status: 加工
project: "[[02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: 权限划分
source:
---
>**笔记不是为了复述书本，而是为了**“存档当下的自己”。如果你的笔记里没有你的思考痕迹、痛苦经历和选择理由**，它就只是一份毫无生命力的说明书，自然无法在未来唤醒你的认知***


## 💥 核心结论
>核心定义是什么? 核心价值在哪里?
### 🔴 核心定义：基于角色的权限控制（Based Role Access Control）的简称，用于实现人员权限如何分配。
### 🔴 核心价值：简化用户权限分配的逻辑，解决为用户分配权限时多次出现重复的问题。

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
- 权限规则：模块:子模块:操作
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
- **sys_user_role（用户-角色关联表）**
- **sys_menu_role（权限-角色关联表）**
- **权限认证框架（Sa-Token）**：用户鉴权

### 流程图
- 「组件」流程概览图：[「RABC」流程概览图](excalidraw/「RABC」流程概览图.md)
- 「数据流转」流程图：[「RABC」数据流转图](excalidraw/「RABC」数据流转图.md)

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
- **场景 A**：在处理 [XXX] 代码逻辑时可以替代原有的 [YYY] 方法。
- **场景 B**：在进行 [ZZZ] 决策时，用来规避逻辑谬误。