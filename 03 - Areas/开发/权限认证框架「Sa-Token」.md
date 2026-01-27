1.整合准备：
🔴 依赖引入（pom文件）与资源文件（yml）配置：提供环境准备
🔴 自动装配注入（Config）：
	- Token「StpLogic」：定义Token类型（Session、纯JWT、session&jwt混合）
	- 自定义权限接口「StpInterface」：定义角色、权限的存取
	- SaTokenDao：Token如何存取（redis or redis+Caffeine本地缓存）
	- 自定义异常：封装框架各类异常，统一异常管理

2.流程应用：
🔴 阶段一：登录
	- StpUtil.login(id)：密码校验后生成Token
	- StpUtil.getTokenSession().set(...)：将用户信息保存到Redis
	- 调用SaTokenDao层：触发保存逻辑（写入Redis、同步写入redis&本地缓存）
🔴 阶段二：认证
	- 拦截器捕获请求头「header」：提取令牌（Authorization：Bearer XXXX）
	- 拦截器调用「StpUtil.checkLogin() 」& 「PlusSaTokenDao」：检查登录是否有效、能否放行controller
🔴 阶段三：鉴权
	- 调用「StpInterface」和「@SaCheckPermission」：权限列表与注解值作匹配

🔴 阶段四：权限异常处理
	- 封装NotPermissionException、NotRoleException、NotLoginException：统一捕获输出异常信息「403」