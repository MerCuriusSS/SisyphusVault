---
tags:
  - Areas/Coder/javaWeb
category: 技术
status: 加工
project:
application:
source:
---
### 启动与自动配置

- `@SpringBootApplication`
- `@EnableAspectJAutoProxy`
- `@MapperScan`
- `@EnableCaching`

🔴`@SpringBootApplication` ：保证 Spring 容器正常运行

```
1.等同于 @SpringBootConfiguration+ @EnableAutoConfiguration + @ComponentScan
2.@ComponentScan 保证扫描范围为所在目录以及子包的的组件
3.@EnableAutoConfiguration 与Spring.factories，配合@ConditionalXX注解实现按需配置
```

🔴`@EnableAspectJAutoProxy`：保证 AOP 能正常运行

```
1.开启切面注解
2.“exposeProxy = true” 允许暴露代理对象——通过AopContext.currentProxy()获取代理对象
```

🔴`@MapperScan`：保证 dao 接口能正常运行

```
1.为Mybatis提供
2.扫描mapper接口并自动为此生成代理对象，无需添加@Mapper注解
```

🔴`@EnableCaching`： 保证 Redis 能通过注解使用

```
1.注册CacheInterceptor，使得@Cacheable生效
```

### IOC 容器与依赖注入

- `@Component`、`@Configuration`、`@Bean`
- `@Autowired`、`@Resource`
- `@Value`
- `@Lazy`
- `@PostConstruct` 、`@PreDestory`

🔴`@Component`、`@Configuration`、`@Bean`

```
1.@Component 默认为类创建一个单例对象
2.@Configuration 与 @Component 作用等效，同样能注册单例对象
3.@Configuration + @Bean 为方法返回值注册单例对象，单例名称等于方法名称。

使用场景：
1）@Component：自定义类 & 初始化逻辑简单
2）@Bean：第三方组件 or 实例构建逻辑复杂 or 需复杂配置
```


🔴`@Autowired`、`@Resource`

```
1.实现对象注入
2.autowired 默认按照类型注入，存在歧义时报错，按名字时搭配@Qualifier
3.resource 默认按照名称注入，可直接指定名称
```

🔴`@Value`

```
1.配置占位符 和 默认值

@Value("${xx.xx.xx}:你好")
```

🔴`@Lazy`

```
1.打破循环依赖；原理：注入代理而非真实Bean、真实Bean延迟到方法调用时执行
```


🔴`@PostConstruct` 

```
1.执行时机：对象创建 → 全部属性 / 依赖注入完成赋值之后->@PostConstruct->afterPropertiesSet()方法

```

### AOP切面编程

- 注解定义
- 切面类
- 方法增强

🔴 注解定义

```
1.定义注解面向目标 @Target(ElementType.METHOD,...)、执行时机 @Retention(RetentionPolicy.RUNTIME)
2.定义注解属性：无参方法，DEFAULT 定义默认值
```


🔴切面类

```
1.@Aspect 定义为切面类
2.@Component 被Spring管理，实现方法增强调用
```

🔴方法增强

```
1.切入点(pointCut)≠连接点(joinPoint)；切入点表示《入口&筛选「规则」》；连接点表示需要增强的「方法」
2.切入点的表现方式：
- 方法：execution(返回类型 包.类.方法(参数)) :execution(* com.sisyphus.zero-code.controller.*)
- 注解：@annotation(注解小写格式)
3.@Around、@Before、@After
4.表现方式：
********************************
@Around(@annotation(rateLimit))
public void method(JoinPoint joinPoint,RateLimit rateLimit){

}
*******************************
```


### Web MVC

- 控制器注解
	- `@RestController`
	- `@PathVariable`、`@RequestParam`、`@RequestBody`
- 统一响应与业务异常模型
	- `BaseResponse(code,data,message)`
	- `ResultUtils`
	- `BussinessException`&`ErrorCode`
- `@RestControllerAdvice`&`@ExceptionHandler`
- `WebMvcConfigurer`
- 访问静态资源
- 文件下载

🔴控制器注解

```
1.@RestController=@Controller+@RestquestBody
2.@PathVariable 提取uri中变量
3.@RequestParam 提取表单变量
4.@RequestBody json反序列化
```

🔴 统一响应与业务异常模型

```
1.结果返回封装为统一响应：响应码+响应消息+响应数据（T）
2.常用返回结果（成功/错误）
3.业务异常关联运行时异常
4.定义常用错误码（枚举类型）
```

🔴 统一捕获与处理异常

```
1.@RestControllerAdvice使用AOP统一管理所有异常
2.@ExceptionHandler 处理对应异常
```

🔴 `WebMvcConfigurer` 与CORS跨域

```
1.管理springMVC到controller之前的行为
2.功能范围：
- 拦截器（addInterceptors）
- 跨域（addCorsMappings）
- 静态资源处理（addResourceHandlers）
- ...
3.跨域坑点：allowCredentials(true) 与 allowedOrigins("*") 不能一起用，应该替换为allowedOriginPatterns("*")
```

🔴访问静态资源

```
1.原理：解析URL->找本地文件目录->文件转为SpringResource对象->根据文件类型设置响应头并返回；
2.相对路径(如src="js/main.js")会以URL基准拼接成完整请求路径（localhost:deploy/js/main.js），再重新走上述的流程
```

🔴文件下载

```
1.原理：读取文件->zip流实时打包成二进制->设置下载专用请求头返回给浏览器

下载请求头：
- Content-Type: application/zip
- Content-Disposition: attachment; filename="xxx.zip"
```

### SSE流式响应

🔴范式

```java
    @GetMapping(value = "/sse", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<String>> sse(String param) {
        // 1. 原始数据流
        Flux<String> dataFlux = streamService.getStream(param);

        // 2. 包装SSE标准事件
        Flux<ServerSentEvent<String>> eventFlux = dataFlux
                .map(data -> ServerSentEvent.builder(data).build());

        // 3. 追加结束事件 + 异常兜底
        return eventFlux
                .concatWith(Mono.just(ServerSentEvent.builder("").event("end").build()))
                .onErrorResume(e -> Mono.just(ServerSentEvent.builder(e.getMessage()).event("err").build()));
    }
```


### Actuator

```
1.核心目的：开放端口，让人可以通过URL访问应用的运行状态、指标等
```

```yaml
management:
  server:
    port: 9528                  # ⚠️ 独立于业务端口 9527
  endpoints:
    web:
      exposure:
        include: health,info,prometheus # 显示列出暴露的端口
  endpoint:
    health:
      show-details: always
```


### 配置文件属性绑定

- `@ConfigurationProperties` VS  `@Value`


```
1.@ConfigurationProperties 把同一前缀属性整体绑定到一个 POJO
2.@Value只支持绑定单个属性
```


### Profile 多环境

yaml文件引用属性值：${spring.name}

- `application.yml`：`spring.profiles.active: local`（默认 local）。
- `application-local.yml`：本地开发
- `application-prod.yml`：生产环境


### 日志：logback-spring.xml

```
1.SpringBoot 默认日志框架：Logback

2.logback.xml 缺陷：无法使用 Spring 的 `spring.profiles.active` 分环境，推荐改成 `logback-spring.xml`

3.优先级：XML > yml 
```