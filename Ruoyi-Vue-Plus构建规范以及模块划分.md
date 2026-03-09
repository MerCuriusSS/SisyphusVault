---
tags:
  - Areas/Coder/javaWeb
category: 技术
status: 加工
project: "[[02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: 工业级项目构建规范
source:
---

#  🔴 maven模块管理
>[!important] 「POM文件类型」 VS 「jar文件类型」
>**POM类型`<packaging>pom</packaging>`：**
>核心在“管理”——父项目（统一版本管理&插件配置）/聚合子模块（module管理&统一构建）；除pom文件外不生成class文件；不能像jar类型被直接依赖！！！不能像jar类型被直接依赖！！！不能像jar类型被直接依赖！！！
>
>**JAR类型`<packaging>jar</packaging>`：**
>核心在“业务”——打包成jar，可运行、可依赖（Maven 默认类型，可省略）


# 🟡 模块划分设计理念
>[!important] 模块划分
>**主应用（admin）：**最后交付的应用、提供统一入口（扫描所有依赖包路径）、整合所有业务模块（保证功能可用）
>
>**功能模块（common）：**
>common-core：基层工具类。
>各类组件服务：序列化、redis缓存、satoken、数据库操作、短信、对象存储等。
>
>**业务模块（modules）**：代码生成、定时任务、演示、系统管理核心模块（用户、角色、菜单、部门等）
>
>**拓展服务（extended）**：监控、分布式定时任务


# 🟢 重点模块拆解 ：
## 一、功能模块（common）
>[!important] 自动装配的「starter」式思想
>**目标**：1.将通用模块拆解成可复用starter，实现按需加载（用则加载、不用则不加载），精简程序包；2.控制配置类加载顺序，避免容器加载混乱。
>
>**流程**：@SpringBootApplication先扫描所有jar包中的imports文件，筛选中标注有@AutoConfiguration的类，再通过@ConditionXX系列注解（依赖是否引入、Bean是否加载）判断是否加载，最终完成配置。
>
>**规则**：没有加@AutoConfiguration注解的类会被跳过，不报错，不影响启动；@AutoConfiguration默认优先级低于用户自定义配置类；使用@AutoConfigureBefore/After可强制控制加载顺序，优先级高于容器默认（无规则随机）顺序；


## 二、业务模块（system）
>[!warning] **概念厘清**：「拦截器」VS 「过滤器」
> **身份**：过滤器（filter）是**servlet**组件；拦截器（dispatchServlet-interceptor）是**spring容器**组件。
> 
> **范围**：过滤器（filter）是规则控制是「**URL**」级别，无法感知业务方法，拿不到Spring中的service【springboot项目中filter的自建权被移交到spring容器中，允许注入】；拦截器（interceptor）是「**Method**」级别，通过`handler`感知controller方法**注解**，并能与注入**Spring Bean**协同工作；控制更精细——涵盖`preHandle`（拦截）、`postHandle`（结果加工）、`afterCompletion`（清理）**三阶段。**
> 
> **应用场景**：过滤器（filter）控制“**通用规则**”（跨域、解密、XSS）；拦截器（interceptor）控制“**业务规则**”（权限、日志等）


>[!bug] 拦截器&过滤器应用——业务执行链
>
├─ ==***第一层：Servlet Filter 链（请求前置过滤）***==
│  ├─ 🔴 **CryptoFilter**（order=-2147483648，ApiDecryptAutoConfiguration.java:27）：请求/响应加密解密
│  ├─ 🔴 **XssFilter**（order=-2147483647，FilterConfig.java:28）：XSS攻击过滤，清理危险字符
│  └─ 🔴 **RepeatableFilter**（默认order=0，FilterConfig.java:36）：包装HttpServletRequest，支持重复读取请求体
│
├─ ==***进入 DispatcherServlet 分发请求***==
│
├─ ==***第二层：Spring MVC 拦截器链（请求前置校验）***==
│  ├─ 🟡 **PlusWebInvokeTimeInterceptor**（ResourcesConfig.java:31）：记录请求开始时间和参数，preHandle打印请求开始日志
│  └─ 🟡 **SaInterceptor**（SecurityConfig.java:51）：Sa-Token核心拦截，校验登录/ClientId/接口权限（@SaCheckPermission），权限不足终止请求
│
├─==*** 进入 SysUserController.add() 控制器方法***==
│
├─ ==***第三层：AOP 切面链（控制器方法执行前后增强）***==
│  ├─ 执行入口：SysUserController.add(@RequestBody SysUserBo user)
│  ├─🟢 **DataPermissionAdvice**（DataPermissionPointcutAdvisor.java）：Mapper层@DataPermission注解触发，ThreadLocal设置数据权限信息（Controller层通常不触发）
│  ├─🟢 **RepeatSubmitAspect**（@Before，RepeatSubmitAspect.java:41）：@RepeatSubmit注解触发，MD5生成唯一key，Redis防重复提交，已存在则抛异常
│  ├─🟢 **LogAspect**（@Before，LogAspect.java:51）：@Log注解触发，创建StopWatch计时并存入ThreadLocal
│  │
│  ├─ 执行 Controller 方法体
│  │  └─ deptService.checkDeptDataScope() → userService.insertUser() → 进入SysUserService.insertUser()
│  │
├─==*** 第四层：MyBatis-Plus 拦截器链（SQL执行时动态处理）***==
│  ├─ 执行入口：SysUserMapper.insert(user) 执行INSERT语句
│  ├─ 🔵 **TenantLineInnerInterceptor**（MybatisPlusConfig.java:42-46）：多租户插件，SQL自动拼接tenant_id（必须首位），INSERT赋值/查询类语句加WHERE条件
│  ├─ 🔵 **PlusDataPermissionInterceptor**（MybatisPlusConfig.java:48）：数据权限拦截，重写SQLWHERE条件（INSERT通常不触发）
│  ├─ 🔵 **PaginationInnerInterceptor**（MybatisPlusConfig.java:50）：分页插件，查询自动加LIMIT（仅查询触发）
│  └─ 🔵 **OptimisticLockerInnerInterceptor**（MybatisPlusConfig.java:52）：乐观锁插件，UPDATE校验version字段（仅更新触发）
│
├─ ==***执行原生 SQL，SQL执行成功返回结果***==
│
├─ ==***第五层：返回时的后置处理链（结果返回/资源清理/日志记录）***==
│  ├─ 🟢 **LogAspect**（@AfterReturning，LogAspect.java:63）：停止计时，构建OperLogEvent事件，异步发布并保存操作日志到数据库
│  ├─ 🟢 **RepeatSubmitAspect**（@AfterReturning，RepeatSubmitAspect.java:77）：根据返回码处理Redis key（成功保留/失败删除）
│  └─ 🟡 **PlusWebInvokeTimeInterceptor**（afterCompletion，PlusWebInvokeTimeInterceptor.java:104）：停止计时，输出请求结束及耗时日志
