---
tags:
  - Areas/开发/javaWeb
  - Areas/开发/基础原理
category: 基础原理
status: 加工
project: "[[../../02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: 通用规则、基础业务控制
source:
---
>[!question] 基本概念：
> 🔴 **身份**：是**servlet**组件；
> 
> 🔴 **范围**：规则控制范围是「**URL**」级别；
> 
> 🔴 **应用场景**：“通用规则”控制（跨域、解密、XSS）


>[!important] 控制权演变——「传统web应用」 VS 「Springboot应用」
>🔴 **servlet**是web应用**底座**——准备web运行的一切基础组件，包括过滤器，**spring**容器是用户创建实例的**代工厂**——准备一切非web组件的业务Bean（Service）。**springMVC**则是spring容器内**专精**处理**Web** 相关的Bean（Controller）
>
>🔴 **传统应用**中启动顺序是**servlet->spring容器**，因此spring容器**无法**干涉过滤器，filter由servlet创建。
>
>🔴 **springboot应用**中启动顺序是**spring容器->servlet**，spring容器控制范围增大，允许将filter**硬塞**到servlet中，能与spring容器的其他ServiceBean**协同**工作，更好完成“通用业务”建设。


>[!example] 基本原理
>🔴 ***Bean管理机制:***
>- 先创建连接servlet与spring容器之间的「**适配器**」：
>- 再创建具体功能的**过滤器**
>```java
>@Configuration
>public class SpringBootFilterConfig {
>
>    @Bean
>    public CustomFilter customFilter() {
>        return new CustomFilter();
>    }
>
>    // Spring Boot推荐的注册方式
>    @Bean
>    public FilterRegistrationBean<CustomFilter> customFilterRegistration(CustomFilter customFilter) {
>        FilterRegistrationBean<CustomFilter> registrationBean = new FilterRegistrationBean<>();
>        registrationBean.setFilter(customFilter);
>        registrationBean.addUrlPatterns("/*"); // 拦截URL
>        registrationBean.setName("customFilter"); // Filter名称
>        registrationBean.setOrder(FilterRegistrationBean.HIGHEST_PRECEDENCE + 1); // 执行顺序（数值越低，优先值越高）
>        return registrationBean;
>    }
>}
>```
>
>🔴 ***运行机制***
> - 本质：**递归调用**
> - doFilter之前：「**前序执行**」——处理未抵达controller**之前**的请求（日志记录、解密等）
>- doFilter()：「**放行动作**」——请求递交给后一个**过滤器/dispatchServlet**
>- doFilter之后：「**后序执行**」——执行响应**之后**的操作（日志记录、资源清理、加密等）
>
>```java
>    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
>            throws IOException, ServletException {
>        // 拦截逻辑
>        chain.doFilter(request, response);
>    }
>```
>

>[!success] 应用场景
>🔴 加解密：（CryptoFilter）
>- **核心目的**：对加密的请求进行**解密**/对可能的响应内容进行**加密**
>- **本质**：读取并加解密**流内容**。
>- **底层逻辑**：读取`Request.getInputStream()`;翻译成UTF-8格式的字符串后进行解密；
>
>🔴 XSS过滤：（XssFilter）
>- **核心目的**：预防XSS注入攻击。
>- **本质**：清除html标签`HtmlUtil.cleanHtmlTag`
>- **底层逻辑**：
>	- 全局限制（wrapper装饰器模式）：
>		- **表单类型**`application/x-www-form-urlencoded`：重写「Parameter」系列方法（getParameterValues、getParameterMap、getParameter），对每个参数值实现html标签清除。
>		- **请求体**`application/json`：拦截字节流并翻译成json，对内容实现html标签清除。
>	- 注解拦截方法级控制：注解`@XSS`、`@Validated`、`ConstraintValidator`实现对对象内容进行html标签清除
>
>🔴 重复读取请求体：（RepeatableFilter）
>- **核心目的**：允许过滤器/拦截器**重复**读取「**请求体**」，解决`inputstream`只能**读取一次**的困境
>- **本质**：输入流内容的拷贝复用。
>- **底层逻辑**：**临时**存储输入流的内容，并在每次`inputStream()`方法被调用时创建**新流**并**塞入**数据。


