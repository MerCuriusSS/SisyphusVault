
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
>[!warning] 拦截器的应用
> **概念厘清**——「拦截器」VS 「过滤器」
> **定位**：过滤器（filter）是servlet组件；拦截器（interceptor）是spring容器组件。
> 
> 过滤器是规则控制是「URL」级别，无法感知业务方法，拿不到Spring中的service；拦截器是「Method」级别，通过`handler`感知controller方法注解，并能与注入Spring Bean协同工作。
> 
> 
🔵 🟣 🟤 ⚫ ⚪ 


