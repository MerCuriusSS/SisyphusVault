## 一、整合准备：

🔴 依赖引入（pom文件）与资源文件（yml）配置：提供环境准备
🔴 自动装配注入（Config）：
- Token「StpLogic」：定义Token类型（Session、纯JWT、session&jwt混合）
- 自定义权限接口「StpInterface」：定义角色、权限的存取
- SaTokenDao：Token如何存取（redis or redis+Caffeine本地缓存）
- 自定义异常：封装框架各类异常，统一异常管理
🔴 拦截器配置（WebMvcConfigurer）：
- SaInterceptor：拦截所有请求，匹配controller路径

## 二、流程应用：

🔴 阶段一：登录
- StpUtil.login(id)：密码校验后生成Token「**StpLogic**」
- StpUtil.getTokenSession().set(...)：将用户信息保存到Redis
- 调用**SaTokenDao**层：触发保存逻辑（写入Redis、同步写入redis&本地缓存）

🔴 阶段二：认证
- 拦截器「**SaInterceptor**」拦截请求：捕获请求头「header」并提取令牌（Authorization：Bearer XXXX）
- 拦截器调用「StpUtil.checkLogin() 」& 「**PlusSaTokenDao**」：检查登录是否有效、能否放行controller

🔴 阶段三：鉴权
- 调用「**StpInterface**」和「@SaCheckPermission」：权限列表与注解值作匹配

🔴 阶段四：权限异常处理
- 封装NotPermissionException、NotRoleException、NotLoginException：统一捕获输出异常信息「403」

## 三、底层工具与思想：

🔴 StpUtil：与redis交互的工具
- 功能：
	1.用户登录与注销：
	- `StpUtil.login(id)`：登录
	- `StpUtil.logout()`：登出
	- `StpUtil.kickout(id)`：踢下线
	2.状态校验与Token获取
	- `StpUtil.checkLogin()、StpUtil.isLogin()`：是否已经登录
	- `StpUtil.getTokenValue()`获取token值
	3.权限与角色比对
	- `StpUtil.hasPermission(permission)`：获取权限信息
	- `StpUtil.hasRole(role)`：角色信息
	4.业务层级的上下文存储
	- `StpUtil.getSession()、StpUtil.getTokenSession()`：用户信息上下文

🔴 Redis底层设计：

```markdown
satoken (Root Prefix)
├── login (Login Type)
│   ├── **token** (令牌->账号映射层: O(1) 鉴权)
│   │   └── <Token值>  -------------> Value: <LoginId> (账号ID)
│   │       
│   │
│   ├── **id** (账号->令牌映射层: O(1) 控权)
│   │   └── <LoginId> --------------> Value: <Token值> (或 Token 列表)
│   │   
│   │
│   ├── **session** (账号全局会话层: 共享状态)
│   │   └── <LoginId> -------------------> Value: {用户名, 角色集合等}
│   │      
│   │
│   ├── **token-session** (令牌专属会话层: 独立状态)
│   │   └── <Token值> -------------------> Value: { 登录地点, 临时上下文... }
│   │      
│   │
│   └── **last-activity** (活跃监测层: 自动续期)
│       └── <Token值> -------------------> Value: <最后访问时间戳>
│          
```

>***设计哲学：***
>1. token->id 与 id->token的**双重映射**：既保证「用户登录」，也保证「踢下线管理」
>2. 用户状态对象与token存储的value**拆分**：**多对一**思想，保证多端登录时无需拷贝多份用户数据，多端更新时用户信息时只改一份数据
>3. 全局用户状态信息与登录端用户信息的拆分：保证“同一账号”的**统一**、**多端**分开维护环境信息
>4. 记录最后活跃时间：每次请求更新活跃时间戳、TTL时间，保证“**过期/自动续签/统计在线时长**”
>	1. **过期=TTL**
>	2. **续签=当前时间戳-写入时时间戳 < TTL，更新TTL**
>	3. **统计在线时长=session记录的startTime+累加值； 累加值=(当前时间戳-写入时时间戳)&&<30分钟**

