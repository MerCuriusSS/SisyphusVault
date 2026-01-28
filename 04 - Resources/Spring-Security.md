# Spring Security

## Spring-Security完整过滤器链

### 子主题

## 登录认证过程

### 流程图

- 子主题

### 在SpringSecurity中的流程

- SpringSecurity的本质：过滤器链
- 基础流程图

	- 子主题

- 认证流程

	- 基于表单登录的实现（入门案例）

		- 子主题

			- 简单步骤理解

				- ①访问登录接口
②基于Filter生成AuthenticationManager调用认证方法【authenticate()】{不使用Filter时通过注入获取}
③基于UserDetailService接口查询用户{自定义}
④用户信息封装成UserDetails对象返回{自定义}
⑤校验密码，设置权限信息，返回已认证Authentication对象{自动}
⑥设置认证条件{自定义}

	- 基于自定义登录的实现

		- 登录实现

			- 子主题
			- ①自定义登录接口
②注入AuthenticationManager做好认证准备
③调用认证方法【authenticate()】
④基于UserDetailService实现数据库查询认证
⑤封装成UserDetail对象返回
⑥校验密码返回已认证Authentication对象
⑦生成JWT，保存到Redis
			- 代码

				- 自定义登录接口

					- 子主题

				- 注入authenticationManager认证配置

					- 子主题

				- 调用认证方法

					- 子主题

				- 基于UserDetailService实现数据库查询认证
封装成UserDetail对象返回

					- 子主题

				- 根据认证结果生成JWT，保存到Redis

					- 子主题

				- 设置认证放行条件

					- 子主题

		- 登录状态校验

			- 子主题
			- ①定义JWT认证过滤器
②获取JWT
③解析JWT的载荷值
④从redis中获取用户信息
⑤封装成已认证authentication对象
⑥存入SecurityContextHolder
⑦调整过滤器优先级
			- 代码

				- ①定义JWT认证过滤器
②获取JWT

					- 子主题

				- ③解析JWT的载荷值
④从redis中获取用户信息

					- 子主题

				- ⑤封装成已认证authentication对象
⑥存入SecurityContextHolder

					- 子主题

				- ⑦调整过滤器优先级

					- 子主题

		- 退出登录

			- 获取SecurityContextHolder中的值，删除对应Redis数据
			- 代码

				- 子主题
				- 子主题

	- 密码加密存储

		- 默认使用的PasswordEncoder要求数据库中的密码格式为{id}password，明文密码存储格式为{noop}password一般使用BCryptPasswordEncoder作为密码加密
		- 子主题

	- 认证配置

		- 子主题

- 授权流程

	- 权限作用：让不同的用户使用不同的功能
	- 子主题

		- 步骤理解

			- 简化理解

				- ①权限信息存入UserDetail并放入authentication对象（getAuthorities()）{自定义}
②放入holder中{自定义}
③交由FilterSecurityInterceptor执行鉴权{自动}

			- 详细步骤

				- ①设置资源所需权限{自定义}
②首次登录时，在认证过程中查询用户信息的同时查询权限信息，放入redis中{自定义}
③后续每次访问API前，构造已认证Authentication对象并在构造参数插入权限（redis中获取），放入Holder中{自定义}
④在FilterSecurityInterceptor中从Holder取出对象并获取权限信息{自动}
⑤访问资源

	- 权限实现

		- 使用注解限制访问权限

			- @PreAuthorize

				- @PreAuthorize("hasAuthority('xxx)")/@PreAuthorize("hasAnyAuthority('xxx)")

					- 只要权限列表中存在所设值，则允许访问

				- @PreAuthorize("hasRole('xxx)")/@PreAuthorize("hasAnyRole('xxx)")

					- 只要权限列表存在ROLE_所设值，则允许访问

				- 原理

					- 

				- 自定义权限校验方法

					- 子主题

		- 使用配置实现权限控制

			- 子主题

		- RBAC模型【Role-Based Access Control（基于角色的权限控制）】

			- 用户表（user）

				- 子主题

			- 角色表（role）

				- 子主题

			- 权限表（menu）

				- 子主题

			- 角色权限关联表（role_menu）【角色与权限是多对多关系】

				- 子主题

			- 用户角色关联表（user_role）【用户与角色是多对多关系】

				- 子主题

	- 代码

		- ①设置资源所需权限

			- 子主题

		- ②首次登录时，在认证过程中查询用户信息的同时查询权限信息，放入redis中

			- 子主题

		- ③后续每次访问API前，构造已认证Authentication对象并在构造参数插入权限（redis中获取），放入Holder中

			- 子主题

		- ④在FilterSecurityInterceptor中从Holder取出对象并获取权限信息

			- 自动

		- ⑤访问资源

			- 自动

### JWT与Token的一些思考

- ①JWT可以单独使用，与Redis搭配时更多是为了续签，但此时JWT本身的验证方式有点画蛇添足
②因此更多情况下，还是选择token+redis的方式【存储用户信息+续签】

### 基于表单登录（UserNamePasswordAuthenticationFilter）的处理器

- 认证成功处理器

	- 子主题
	- 子主题

- 认证失败处理器

	- 子主题
	- 子主题

- 注销成功处理器

	- 子主题
	- 子主题

## springSecurity异常处理机制

### 认证失败

- 被封装成AuthenticationException然后调用AuthenticationEntryPoint对象

### 授权失败

- 被封装成AccessDeniedException然后调用AccessDeniedHandler对象

### 自定义失败处理

- ①实现AuthenticationEntryPoint、AccessDeniedHandler

	- 子主题
	- 子主题

- ②注入到security配置中

	- 子主题

## springSecurity跨域问题

### 子主题

## springSecurity中的跨站请求伪造（csrf）问题

### ①springSecurity默认通过csrf_token实现csrf的防范
②csrf主要依赖cookie来进行请求伪造；前后端分离网站依赖的是token，天然防范csrf，所以可以禁用掉csrf

### 子主题

## 总结

### 登录流程

- ①前端携带用户名密码访问登录接口【用户名密码访问】
②数据库查询并比对用户名密码信息【数据库查询比对】
③比对成功则在redis存储用户信息，生成并返回token给前端【存储用户信息并生成token】


### 登录状态校验

- ④前端下次访问其他请求时携带token，根据token获取用户信息【token查询用户信息】
⑤访问目标资源，返回响应给前端。【访问目标资源】

### SpringSecurity作为权限框架，本质上就是一系列的过滤链；引入DelegatingFilterProxy，生成FilterChainProxy来代理执行过滤链

- 子主题

### 基本流程

- 子主题
- ①通过UserNamePasswordAuthenticationFilter实现了表单登录的认证和授权。
②访问受保护资源时，使用FilterSecurityInterceptor实现鉴权。
③操作当中出现的认证和授权异常由ExceptionTranslationFilter统一处理

### SpringSecurity的登录认证流程

- 表单登录认证流程图【基于内存】（务必要记牢）

	- 子主题
	- 描述

		- ①基于表单访问登录接口
②基于Filter生成AuthenticationManager并调用AuthenticationManager认证方法
③实现UserDetailService接口查询用户
④封装成UserDetails对象返回
⑤校验密码、设置权限信息，返回认证成功Authentication对象
⑥设置认证条件

	- 重要组件

		- AuthenticationManager

			- 由过滤器产生或自行注入，执行认证方法，返回已认证对象

		- UserDetailService

			- 实现查询用户具体操作

		- UserDetail

			- 封装用户信息

		- AuthenticationProvider

			- 实现用户密码校验、存放权限信息，封装成已认证Authentication对象

		- Authentication

			- ①作为准备认证的封装对象
②作为完成认证授权的封装对象

- 拓展延伸【基于Redis】

	- 自定义登录接口流程图

		- 子主题
		- 描述

			- ①自定义登录接口
②注入AuthenticationManager做好认证准备
③调用认证方法【authenticate()】
④基于UserDetailService实现数据库查询认证
⑤封装成UserDetail对象返回
⑥校验密码返回已认证Authentication对象
⑦生成JWT，保存到Redis
			- 子主题

	- 登录状态校验流程图

		- 描述

			- ①定义JWTFilter
②基于JWT获取用户信息
③封装已认证Authentication对象
④存入SecurityContextHolder
⑤调整过滤器优先级

	- 退出登录

		- 获取SecurityContextHolder中的值，删除对应Redis数据

### 授权流程

- 子主题
- 描述

	- 基本流程

		- ①认证查询时将权限信息存入UserDetails（getAuthorities()）并封装到Authentication对象
②放入holder中
③后续访问受保护资源时交由FilterSecurityInterceptor执行鉴权
④访问受权限约束资源

	- 实际流程

		- ①设置资源所需权限
②首次登录时，在认证过程中查询用户信息的同时查询权限信息，放入redis中
③后续每次访问API前，构造已认证Authentication对象并在构造参数插入权限（redis中获取），放入Holder中
④在FilterSecurityInterceptor中从Holder取出对象并获取权限信息
⑤访问资源

- UserDetail

	- 重写getAuthorities()方法得到用户权限信息

- 重要组件

	- SecurityHolder

		- 后续鉴权时需要借此获取Authentication对象

	- Authentication

		- 作为完成认证、授权的封装对象

	- FilterSecurityInterceptor

		- 结合@PreAuthorize注解、hasAuthority()等方法实现鉴权

- RBAC模型

	- 表的构造

		- 用户表
		- 角色表
		- 权限表
		- 用户_角色关联表
		- 角色_权限关联表

### 异常处理机制

- 原理

	- AuthenticationEntryPoint 捕获认证异常【AuthenticationException】
	- AccessDeniedHandler 捕获授权异常【AccessDeniedException】

- 自定义异常

	- 实现handler，构造返回结果
	- 加入Security配置

### springSecurity的跨域问题/CSRF问题

- springSecurity配置设置跨域和CSRF

*XMind - Trial Version*