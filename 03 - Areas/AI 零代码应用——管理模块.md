---
tags:
  - Areas/Coder/javaWeb
category: 技术
status: 加工
project:
application:
source:
---
### CRUD模块

- 初始化实体类属性（主键、别名）
- 参数校验
	- 空指针校验（NPE）
	- 业务规则校验
- ORM 框架操作数据库
	- 新增：依实体新增
	- 修改：依实体修改（带主键）
	- 查询：依查询字段、查询条件查询
	- 删除：依主键删除
- Entity转VO
	- 直转
	- 脱敏

### 用户管理

- 注册
- 登录
- 登录状态
- 权限验证
- 注销



#### 注册

- 用户名重复验证
- 密码加密（加盐MD5）
- 插入数据库
- 返回用户id

```java
public long userRegister(String userAccount, String userPassword, String checkPassword) {  
    // 2. 检查是否重复  
    QueryWrapper queryWrapper = new QueryWrapper();  
    queryWrapper.eq("userAccount", userAccount);  
    long count = this.mapper.selectCountByQuery(queryWrapper);  
    if (count > 0) {  
        throw new BusinessException(ErrorCode.PARAMS_ERROR, "账号重复");  
    }  
    // 3. 加密  
    String encryptPassword = getEncryptPassword(userPassword);  
    // 4. 插入数据  
    User user = new User();  
    user.setUserAccount(userAccount);  
    user.setUserPassword(encryptPassword);  
    user.setUserName("无名氏");  
    user.setUserRole(UserRoleEnum.USER.getValue());  
    boolean saveResult = this.save(user);  
    if (!saveResult) {  
        throw new BusinessException(ErrorCode.SYSTEM_ERROR, "注册失败，数据库错误");  
    }  
    return user.getId();  
}
```

#### 登录

- 密码加密
- 用户+密码 验证
- 保存用户登录状态
	- 单机 session
	- redis session
- 返回脱敏用户信息

单机session 保存用户登录状态

```java

request.getSession().setAttribute(USER_LOGIN_STATE, user);
```

redis-session 保存用户登录状态

```xml
<!-- Redis客户端 lettuce（springboot默认） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- Spring Session Redis 核心 -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>

```

```yaml
spring:
  session:  
    store-type: redis  
	# 30天过期  
	timeout: 2592000
```

```java
// 2. 加密  
String encryptPassword = getEncryptPassword(userPassword);  
// 查询用户是否存在  
QueryWrapper queryWrapper = new QueryWrapper();  
queryWrapper.eq("userAccount", userAccount);  
queryWrapper.eq("userPassword", encryptPassword);  
User user = this.mapper.selectOneByQuery(queryWrapper);  
// 用户不存在  
if (user == null) {  
    throw new BusinessException(ErrorCode.PARAMS_ERROR, "用户不存在或密码错误");  
}  
// 3. 记录用户的登录态  
request.getSession().setAttribute(USER_LOGIN_STATE, user);  
// 4. 获得脱敏后的用户信息  
return this.getLoginUserVO(user);
```

#### 获取登录状态

- 获取 `session` 用户登录属性
- 值存在校验
- 数据库进一步核对身份（可选）

```java
public User getLoginUser(HttpServletRequest request) {  
        // 先判断是否已登录（getSession(false) 不会为新请求创建 session，防止未登录请求污染 Redis）  
        jakarta.servlet.http.HttpSession session = request.getSession(false);  
        if (session == null) {  
            throw new BusinessException(ErrorCode.NOT_LOGIN_ERROR);  
        }  
        Object userObj = session.getAttribute(USER_LOGIN_STATE);  
        User currentUser = (User) userObj;  
        if (currentUser == null || currentUser.getId() == null) {  
            throw new BusinessException(ErrorCode.NOT_LOGIN_ERROR);  
        }  
//        // 从数据库查询（追求性能的话可以注释，直接返回上述结果）  
//        long userId = currentUser.getId();  
//        currentUser = this.getById(userId);  
//        if (currentUser == null) {  
//            throw new BusinessException(ErrorCode.NOT_LOGIN_ERROR);  
//        }  
        return currentUser;  
    }
```

#### 权限验证

- 请求接口添加「权限注解」
- 切面类获取用户信息
- 权限值与用户身份比较
- 制定拦截/放行规则

```java
@AuthCheck(mustRole = UserConstant.ADMIN_ROLE)  
public BaseResponse<Page<UserVO>> listUserVOByPage(@RequestBody UserQueryRequest userQueryRequest){
	//.....
}
-------------------------------------------------------------------

@Around("@annotation(authCheck)")  
public Object doInterceptor(ProceedingJoinPoint joinPoint, AuthCheck authCheck) throws Throwable {  
    String mustRole = authCheck.mustRole();  
    RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();  
    HttpServletRequest request = ((ServletRequestAttributes) requestAttributes).getRequest();  
    // 当前登录用户  
    User loginUser = userService.getLoginUser(request);  
    UserRoleEnum mustRoleEnum = UserRoleEnum.getEnumByValue(mustRole);  
    // 不需要权限，放行  
    if (mustRoleEnum == null) {  
        return joinPoint.proceed();  
    }  
    // 以下为：必须有该权限才通过  
    // 获取当前用户具有的权限  
    UserRoleEnum userRoleEnum = UserRoleEnum.getEnumByValue(loginUser.getUserRole());  
    // 没有权限，拒绝  
    if (userRoleEnum == null) {  
        throw new BusinessException(ErrorCode.NO_AUTH_ERROR);  
    }  
    // 要求必须有管理员权限，但用户没有管理员权限，拒绝  
    if (UserRoleEnum.ADMIN.equals(mustRoleEnum) && !UserRoleEnum.ADMIN.equals(userRoleEnum)) {  
        throw new BusinessException(ErrorCode.NO_AUTH_ERROR);  
    }  
    // 通过权限校验，放行  
    return joinPoint.proceed();  
}
```

#### 注销

- 获取 session 用户登录属性
- 移除 session 用户登录属性

```java
public boolean userLogout(HttpServletRequest request) {  
    // 先判断是否已登录（getSession(false) 不会为新请求创建 session）  
    jakarta.servlet.http.HttpSession session = request.getSession(false);  
    if (session == null) {  
        throw new BusinessException(ErrorCode.OPERATION_ERROR, "未登录");  
    }  
    Object userObj = session.getAttribute(USER_LOGIN_STATE);  
    if (userObj == null) {  
        throw new BusinessException(ErrorCode.OPERATION_ERROR, "未登录");  
    }  
    // 移除登录态  
    session.removeAttribute(USER_LOGIN_STATE);  
    return true;  
}
```