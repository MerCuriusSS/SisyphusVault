---
title: 后台管理系统「Ruoyi-Vue-Plus」
status: 运行
deadline: 2026-01-18
tags: project
address: https://gitee.com/dromara/RuoYi-Vue-Plus
---
>[!example] 🏗️ Areas
> ```dataview
> LIST 
> FROM "03 - Areas"
> WHERE project=this.file.link
> ```


## 🚀 核心战略：AI 双模驱动下的“闭环学习”

在执行以下大纲时，请务必遵循：

- **归纳型 AI (GPT/Claude)：** 用于深度对齐源码逻辑。例如：_“请分析 Ruoyi-Vue-Plus 的数据权限拦截器元模型，总结其权限过滤的底层规则。”_ 1
    
- **演绎型 AI (NotebookLM/Midjourney)：** 用于将技术原理转化为面试演示文档或架构图 2。
    
- **行动逻辑：** 尝试阅读源码 -> 失败 -> 询问 AI 核心元模型 -> 动手复现关键代码（里程碑） -> 撰写技术复盘（知识库）
    

---


**核心目标**：从“业务实现者”向“体系构建者”转型，掌握分布式与多租户隔离的工程化标准。

- **行动方案**：
    1. **架构解构**：剖析admin、common、system模块化设计，理解依赖隔离的物理逻辑。
    2. **安全平替**：对比Sa-Token与Spring Security，实操多端登录与二级认证逻辑，摒弃臃肿的Filter链思维。
    3. **数据隔离**：通过MyBatis-Plus拦截器实现行级物理隔离，手动调试JSqlParser产生的SQL冲突问题。
    4. **多级缓存**：落地Caffeine+Redis二级缓存架构，利用Redis Pub/Sub实现本地缓存失效机制。
- **KPI**：
    - 独立完成一个基于Sa-Token的自定义权限策略扩展。与**AI 协同产出**一套由 AI 辅助生成的“Sa-Token 深度集成元模型手册”。
    - 在复杂JOIN查询下，成功解决多租户拦截器导致的语法错误,在复杂 JOIN 查询解决后，在公开平台发布一篇技术剖析博文，逻辑必须清晰到让外行也能看懂 70% 。
- **截止日期**：第7天。

---
模块层级结构分析

  1️⃣ 根父模块：ruoyi-vue-plus

  - 作用：顶层聚合 POM，统一管理所有子模块
  - 职责：
    - 版本统一管理（当前版本：5.5.2）
    - 依赖版本管理
    - Maven 插件配置
    - 编译规范配置（Java 16+）

  2️⃣ 第一层子模块（4个聚合模块）：

  | 模块          | 类型 | 说明                                    |
  |---------------|------|-----------------------------------------|
  | ruoyi-admin   | JAR  | 主应用入口模块，包含 Spring Boot 启动类 |
  | ruoyi-common  | POM  | 通用模块聚合器，包含 24 个子模块        |
  | ruoyi-modules | POM  | 业务模块聚合器，包含 5 个业务模块       |
  | ruoyi-extend  | POM  | 扩展服务聚合器，包含 2 个扩展服务       |

  3️⃣ 核心依赖链（ruoyi-common 的层级关系）：

  底层 → 顶层：
  common-core (最底层)
      ↓
  common-json
      ↓
  common-redis
      ↓
  common-satoken → common-mybatis
      ↓                 ↓
  common-security   common-tenant
      ↓                 ↓
  业务模块依赖 ← ← ← ← ← ←

  关键依赖说明：
  - common-core: 最底层，只依赖 Spring 框架和第三方工具类（Hutool、Lombok、MapStruct）
  - common-redis: 依赖 core，提供 Redis 缓存功能
  - common-satoken: 依赖 core 和 redis，提供 Sa-Token 认证
  - common-mybatis: 依赖 core 和 satoken，提供数据库操作
  - common-tenant: 依赖 mybatis 和 redis，提供多租户功能
  - common-security: 依赖 satoken，提供安全控制
  - common-web: 依赖 json，提供 Web 服务基础


  模块分类总结

  🔴 核心公共模块（最底层，被其他所有模块依赖）

  - ruoyi-common-core: 提供基础工具类、常量、异常处理、验证注解等

  🟡 基础服务模块（构建在核心模块之上）

  - ruoyi-common-json: JSON 序列化（Jackson）
  - ruoyi-common-redis: Redis 缓存服务
  - ruoyi-common-satoken: Sa-Token 认证框架
  - ruoyi-common-mybatis: MyBatis-Plus 数据库操作
  - ruoyi-common-web: Web 服务基础

  🟢 功能增强模块（依赖基础服务）

  - ruoyi-common-tenant: 多租户支持
  - ruoyi-common-security: 安全控制
  - ruoyi-common-log: 日志记录
  - ruoyi-common-excel: Excel 导入导出
  - ruoyi-common-oss: 对象存储
  - ruoyi-common-sms: 短信服务
  - ruoyi-common-mail: 邮件服务
  - 其他功能模块...

  🔵 业务模块（最终实现）

  - ruoyi-system: 系统管理核心模块（用户、角色、菜单、部门等）
  - ruoyi-job: 定时任务管理
  - ruoyi-generator: 代码生成器
  - ruoyi-demo: 演示模块
  - ruoyi-workflow: 工作流模块

  🟣 应用入口模块

  - ruoyi-admin: Spring Boot 应用启动入口，聚合所有业务模块和通用模块，最终打包为可执行 JAR

  这种模块化设计的优点是：
  1. 层次清晰：核心 → 基础 → 增强 → 业务
  2. 职责单一：每个模块只负责特定功能
  3. 低耦合：通过依赖管理避免循环依赖
  4. 易扩展：新模块可以按需引入依赖
  5. 易维护：修改某个功能只需更新对应的 common 模块

---

● 基于深入分析，我将为你详细解析 admin、common、system 三个模块的隔离逻辑和扩展机制。

  📊 一、显式依赖路径图

  1️⃣ Maven 依赖路径


  2️⃣ 依赖路径清单

  | 源模块                         | 目标模块           | 依赖类型 | 依赖方式        |
  |--------------------------------|--------------------|----------|-----------------|
  | admin                          | system             | 强依赖   | pom.xml         |
  | admin                          | job                | 强依赖   | pom.xml         |
  | admin                          | generator          | 强依赖   | pom.xml         |
  | admin                          | demo               | 强依赖   | pom.xml         |
  | admin                          | workflow           | 强依赖   | pom.xml         |
  | admin                          | common-doc         | 弱依赖   | Spring 自动注入 |
  | admin                          | common-social      | 弱依赖   | Spring 自动注入 |
  | admin                          | common-ratelimiter | 弱依赖   | Spring 自动注入 |
  | admin                          | common-mail        | 弱依赖   | Spring 自动注入 |
  | system                         | common-core        | 强依赖   | pom.xml         |
  | system                         | common-mybatis     | 强依赖   | pom.xml         |
  | system                         | common-web         | 强依赖   | pom.xml         |
  | system                         | common-security    | 强依赖   | pom.xml         |
  | system                         | common-tenant      | 强依赖   | pom.xml         |
  | system → common-* (其他 11 个) | 强依赖             | pom.xml  |                 |

  ---
  🔧 二、核心扩展机制分析

  1️⃣ Spring Boot 3.x 自动配置机制

  配置文件位置

  ruoyi-common-xxx/src/main/resources/META-INF/spring/
  └── org.springframework.boot.autoconfigure.AutoConfiguration.imports

  示例：common-core 的自动配置

  # ruoyi-common-core/.../AutoConfiguration.imports
  org.dromara.common.core.config.ApplicationConfig
  org.dromara.common.core.config.ThreadPoolConfig
  org.dromara.common.core.config.ValidatorConfig
  org.dromara.common.core.utils.SpringUtils

  示例：common-satoken 的自动配置

  # ruoyi-common-satoken/.../AutoConfiguration.imports
  org.dromara.common.satoken.config.SaTokenConfig

  核心配置类：SaTokenConfig.java
 ```java
 @AutoConfiguration
  @PropertySource(value = "classpath:common-satoken.yml", factory = YmlPropertySourceFactory.class)
  public class SaTokenConfig {

      /**
       * 权限接口实现(使用bean注入方便用户替换)
       */
      @Bean
      public StpInterface stpInterface() {
          return new SaPermissionImpl();  // ← 默认实现
      }

      /**
       * 自定义dao层存储
       */
      @Bean
      public SaTokenDao saTokenDao() {
          return new PlusSaTokenDao();  // ← 默认实现（绑定 Redis）
      }
  }
 ```
  

  🎯 扩展点：
  - 用户可以在自己的模块中创建 @Primary 标注的 Bean 来覆盖默认实现
  - 例如：实现 SaTokenDao 接口，使用内存存储而非 Redis

  ---
  2️⃣ 条件装配机制

  ① @ConditionalOnMissingBean - 防止重复定义
```java
// SpringDocConfig.java:46
  @Bean
  @ConditionalOnMissingBean(OpenAPI.class)  // ← 只有当容器中没有 OpenAPI Bean 时才创建
  public OpenAPI openApi(SpringDocProperties properties) {
      // ... 创建 OpenAPI Bean
  }
```
  

  🎯 扩展点：
  - 如果用户自定义了 OpenAPI Bean，系统将使用用户的实现
  - 无需修改 common 模块代码

  ② @ConditionalOnProperty - 配置驱动
```java
// TenantConfig.java:32
  @AutoConfiguration(after = {RedisConfig.class})
  @ConditionalOnProperty(value = "tenant.enable", havingValue = "true")
  public class TenantConfig {
      // 只有当配置文件中 tenant.enable=true 时才会启用
  }

  // EncryptorAutoConfiguration.java:24
  @ConditionalOnProperty(value = "mybatis-encryptor.enable", havingValue = "true")
  public class EncryptorAutoConfiguration {
      // 只有配置了加密功能才启用
  }

  ③ @ConditionalOnClass - 类路径检测

  // TenantConfig.java:35
  @ConditionalOnClass(TenantLineInnerInterceptor.class)
  @AutoConfiguration
  static class MybatisPlusConfiguration {
      // 只有当类路径中存在 MyBatis-Plus 时才生效
  }
```
  

  ---
  3️⃣ @Primary 覆盖机制

  示例：多租户覆盖默认 Bean
```java
  // TenantConfig.java:71-84
  /**
   * 多租户缓存管理器
   */
  @Primary  // ← 标记为主要 Bean，覆盖默认实现
  @Bean
  public CacheManager tenantCacheManager() {
      return new TenantSpringCacheManager();
  }

  /**
   * 多租户鉴权dao实现
   */
  @Primary  // ← 覆盖 SaTokenConfig 中的 saTokenDao()
  @Bean
  public SaTokenDao tenantSaTokenDao() {
      return new TenantSaTokenDao();  // 包装了原始的 PlusSaTokenDao，增加租户隔离
  }
```


  🎯 扩展点：
  - 多租户模块通过 @Primary 覆盖了默认的 SaTokenDao 和 CacheManager
  - 实现了自动的租户隔离，无需修改底层代码

  ---
  4️⃣ 接口抽象 + Bean 注入机制

  ① 敏感数据脱敏接口
```java
 // common-sensitive/src/.../SensitiveService.java
  public interface SensitiveService {
      /**
       * 是否脱敏
       * @param roleKey 角色标识
       * @param perms 权限标识
       * @return true-需要脱敏 false-不需要脱敏
       */
      boolean isSensitive(String[] roleKey, String[] perms);
  }

  system 模块实现：
  // ruoyi-system/.../SysSensitiveServiceImpl.java
  @Service
  public class SysSensitiveServiceImpl implements SensitiveService {
      @Override
      public boolean isSensitive(String[] roleKey, String[] perms) {
          // 超级管理员、租户管理员不脱敏
          if (TenantHelper.isEnable()) {
              return !LoginHelper.isSuperAdmin() && !LoginHelper.isTenantAdmin();
          }
          return !LoginHelper.isSuperAdmin();
      }
  }

  common 模块使用：
  // common 模块通过 Spring 注入 SensitiveService
  @Autowired(required = false)  // ← 允许不存在
  private SensitiveService sensitiveService;

  // 使用时判断
  public String desensitize(String data, String[] roleKey, String[] perms) {
      if (sensitiveService != null && sensitiveService.isSensitive(roleKey, perms)) {
          return SensitiveUtil.desensitize(data);
      }
      return data;
  }
```
 

  🎯 扩展点：
  - common 模块定义接口，不提供实现
  - system 模块提供默认实现
  - 第三方开发者可以：
    a. 实现自己的 SensitiveService
    b. 使用 @Primary 或 @Component 覆盖默认实现

  ② Excel 动态选项接口

```java
// common-excel/src/.../ExcelOptionsProvider.java
  public interface ExcelOptionsProvider {
      /**
       * 获取下拉选项数据
       */
      Set<String> getOptions();
  }

  使用方式：
  // Excel 导出注解
  @ExcelDynamicOptions(providerClass = MyOptionsProvider.class)
  private String status;

  // 用户实现
  @Component
  public class MyOptionsProvider implements ExcelOptionsProvider {
      @Override
      public Set<String> getOptions() {
          return Set.of("启用", "禁用");
      }
  }

  运行时动态加载：
  // ExcelDownHandler.java:125
  ExcelOptionsProvider provider = SpringUtils.getBean(dynamicOptions.providerClass());
  Set<String> options = provider.getOptions();
```
  

  ---
  5️⃣ 核心业务服务接口
```java
// common-core/src/.../service/OssService.java
  public interface OssService {
      String selectUrlByIds(String ossIds);
      List<OssDTO> selectByIds(String ossIds);
  }

  // common-core/src/.../service/UserService.java
  public interface UserService {
      String selectUserNameById(Long userId);
      String selectNicknameById(Long userId);
      List<UserDTO> selectListByIds(List<Long> userIds);
      // ... 更多方法
  }

  // common-core/src/.../service/PermissionService.java
  public interface PermissionService {
      Set<String> getRolePermission(Long userId);
      Set<String> getMenuPermission(Long userId);
  }
```
  

  🎯 扩展点：
  - common-core 定义业务服务接口
  - common 模块通过接口调用业务服务
  - system 模块提供实现（通过 @Service 注入）
  - 第三方开发者可以实现新的存储方式（如 MongoDB、Elasticsearch）

  ---
  💡 三、如何添加新的存储实现（实战案例）

  场景：在不修改 common 核心代码的前提下，增加一个基于 Elasticsearch 的用户查询实现

  方案 1：实现接口 + Bean 覆盖

  步骤 1：创建新模块

  ruoyi-store-elastic/         # 新模块
  ├── pom.xml
  └── src/main/java/
      └── org/dromara/store/elastic/
          ├── config/
          │   └── ElasticAutoConfiguration.java
          └── service/
              └── ElasticUserServiceImpl.java

  步骤 2：实现 UserService 接口
```java
package org.dromara.store.elastic.service;

  import org.dromara.common.core.domain.dto.UserDTO;
  import org.dromara.common.core.service.UserService;
  import org.springframework.stereotype.Service;

  import java.util.List;
  import java.util.Map;

  @Service  // ← 注册为 Spring Bean
  public class ElasticUserServiceImpl implements UserService {

      @Autowired
      private ElasticsearchRestTemplate esTemplate;

      @Override
      public String selectUserNameById(Long userId) {
          // 从 Elasticsearch 查询
          UserDoc doc = esTemplate.get(
              String.valueOf(userId),
              UserDoc.class,
              "users"
          );
          return doc != null ? doc.getUserName() : null;
      }

      @Override
      public String selectNicknameById(Long userId) {
          // Elasticsearch 实现
      }

      @Override
      public String selectNicknameByIds(String userIds) {
          // 批量查询
      }

      // ... 实现其他方法
  }
```
  

  步骤 3：创建自动配置类
```java
 package org.dromara.store.elastic.config;

  import org.springframework.boot.autoconfigure.AutoConfiguration;
  import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
  import org.springframework.data.elasticsearch.client.elc.ElasticsearchRestTemplate;

  @AutoConfiguration(after = UserServiceAutoConfiguration.class)
  @ConditionalOnClass(ElasticsearchRestTemplate.class)  // ← 只有引入 Elasticsearch 才生效
  public class ElasticAutoConfiguration {
      // 配置 Elasticsearch 相关 Bean
  }
```
 

  步骤 4：添加自动配置文件
```yaml
# ruoyi-store-elastic/src/main/resources/META-INF/spring/
  # org.springframework.boot.autoconfigure.AutoConfiguration.imports
  org.dromara.store.elastic.config.ElasticAutoConfiguration
```
  

  步骤 5：在 admin 模块引入依赖
  ```maven
   <!-- ruoyi-admin/pom.xml -->
  <dependency>
      <groupId>org.dromara</groupId>
      <artifactId>ruoyi-store-elastic</artifactId>
      <version>${revision}</version>
  </dependency>

  <!-- Elasticsearch 客户端 -->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
  </dependency>
  ```

 

  步骤 6：配置文件启用
```yaml
# application.yml
  spring:
    elasticsearch:
      uris: http://localhost:9200
```
  

  ✅ 结果：
  - common 核心代码完全不需要修改
  - 运行时 Spring 会自动注入 ElasticUserServiceImpl
  - 所有调用 UserService 的地方都会使用 Elasticsearch 实现

  ---
  方案 2：使用 @Primary 覆盖（适合替换默认实现）

  场景：将 Sa-Token 的 Redis 存储改为 内存存储

```java
package org.dromara.custom.satoken;

  import cn.dev33.satoken.dao.SaTokenDao;
  import org.dromara.common.satoken.core.dao.PlusSaTokenDao;
  import org.springframework.context.annotation.Primary;
  import org.springframework.stereotype.Component;

  import java.util.concurrent.ConcurrentHashMap;

  /**
   * 内存存储实现（开发环境使用）
   */
  @Primary  // ← 覆盖 PlusSaTokenDao
  @Component
  public class MemorySaTokenDao implements SaTokenDao {

      private final ConcurrentHashMap<String, String> cache = new ConcurrentHashMap<>();

      @Override
      public String get(String key) {
          return cache.get(key);
      }

      @Override
      public void set(String key, String value, long timeout) {
          cache.put(key, value);
      }

      @Override
      public void delete(String key) {
          cache.remove(key);
      }

      // ... 实现其他方法
  }
```

  

  ✅ 结果：
  - 无需 Redis 环境
  - 所有 Sa-Token 的会话数据存储在内存中
  - 适合开发测试环境

  ---
  方案 3：条件配置切换（适合多环境）

  步骤 1：定义配置属性
```java
@ConfigurationProperties(prefix = "custom.store")
  public class CustomStoreProperties {
      /**
       * 存储类型：redis/mysql/elasticsearch/memory
       */
      private String type = "redis";
  }
```
  

  步骤 2：条件配置类
```java
@AutoConfiguration
  @EnableConfigurationProperties(CustomStoreProperties.class)
  public class CustomStoreAutoConfiguration {

      @Bean
      @ConditionalOnProperty(name = "custom.store.type", havingValue = "elasticsearch")
      public UserService elasticsearchUserService() {
          return new ElasticUserServiceImpl();
      }

      @Bean
      @ConditionalOnProperty(name = "custom.store.type", havingValue = "mysql")
      public UserService mysqlUserService() {
          return new MysqlUserServiceImpl();
      }

      @Bean
      @ConditionalOnProperty(name = "custom.store.type", havingValue = "redis", matchIfMissing = true)
      @Primary  // 默认实现
      public UserService redisUserService() {
          return new RedisUserServiceImpl();
      }
  }
```
  

  步骤 3：配置文件切换
  # 开发环境：使用内存
```yaml

  custom:
    store:
      type: memory
```
  

  # 测试环境：使用 MySQL
```yaml
  spring:
    profiles: test
  custom:
    store:
      type: mysql
```


  # 生产环境：使用 Redis
 ```yaml
  spring:
    profiles: prod
  custom:
    store:
      type: redis
 ```
 

  ---
  🎯 四、扩展机制总结表

  | 扩展机制   | 核心注解/接口             | 适用场景     | 是否需要修改 common | 示例                    |
  |------------|---------------------------|--------------|---------------------|-------------------------|
  | 接口抽象   | interface UserService     | 业务服务扩展 | ❌ 不需要           | ElasticUserServiceImpl  |
  | Bean 覆盖  | @Primary                  | 替换默认实现 | ❌ 不需要           | 内存存储覆盖 Redis 存储 |
  | 条件装配   | @ConditionalOnMissingBean | 防止重复定义 | ❌ 不需要           | 自定义 OpenAPI          |
  | 配置驱动   | @ConditionalOnProperty    | 功能开关     | ❌ 不需要           | 多租户开关              |
  | 类路径检测 | @ConditionalOnClass       | 依赖检测     | ❌ 不需要           | 只有引入 ES 才启用      |
  | 自动配置   | AutoConfiguration.imports | 模块自动启用 | ❌ 不需要           | Spring Boot 3.x 机制    |
  | 动态接口   | ExcelOptionsProvider      | 运行时扩展   | ❌ 不需要           | Excel 动态选项          |
  | 工厂模式   | OssFactory.instance()     | 多实现切换   | ❌ 不需要           | 动态 OSS 客户端         |

  ---
  📌 五、关键设计原则

  1. 单向依赖原则

  common (核心) ← system (业务) ← admin (应用)
     ↑              ↑                ↑
   定义接口      实现接口          聚合模块

  2. 开闭原则

  - 对扩展开放：通过接口和条件装配支持扩展
  - 对修改封闭：无需修改 common 核心代码

  3. 依赖倒置原则

  - common 模块依赖接口抽象
  - system 模块依赖接口 + 提供实现
  - 两者都依赖抽象，不依赖具体实现

  4. 配置外部化

  - 通过 application.yml 控制功能开关
  - 通过 @ConditionalOnProperty 响应配置

  ---
  ✅ 总结

  RuoYi-Vue-Plus 通过以下机制实现了高度的灵活性和可扩展性：

  1. Spring Boot 3.x 自动配置：零配置自动启用模块
  2. 接口抽象：common 定义接口，system 提供实现
  3. 条件装配：智能判断是否启用功能
  4. Bean 覆盖：通过 @Primary 替换默认实现
  5. 工厂模式：运行时动态选择实现

  第三方开发者无需修改 common 核心代码，只需：
  6. 实现接口
  7. 创建 @Service 或 @Primary Bean
  8. 添加 AutoConfiguration.imports 文件
  9. 引入模块依赖

  这种设计保证了核心模块的稳定性，同时提供了强大的扩展能力！ 🚀


---
# 新增用户请求至 SysUserService.save() 完整执行链
## 完整执行顺序流程
### 第一层：Servlet Filter 链
1. **CryptoFilter**（order = HIGHEST_PRECEDENCE = -2147483648）
   位置：ApiDecryptAutoConfiguration.java:27
   功能：请求/响应加密解密
2. **XssFilter**（order = HIGHEST_PRECEDENCE + 1 = -2147483647）
   位置：FilterConfig.java:28
   功能：XSS 攻击过滤，清理危险字符
3. **RepeatableFilter**（默认 order = 0）
   位置：FilterConfig.java:36
   功能：包装 HttpServletRequest，支持重复读取请求体
   
→ 进入 DispatcherServlet

### 第二层：Spring MVC 拦截器链
4. **PlusWebInvokeTimeInterceptor**
   位置：ResourcesConfig.java:31
   功能：记录请求开始时间和参数，输出日志
   preHandle: 打印 "[PLUS]开始请求 => URL[POST /system/user]"
5. **SaInterceptor**（Sa-Token 核心拦截器）
   位置：SecurityConfig.java:51
   功能：
   - StpUtil.checkLogin() 检查登录状态
   - 验证 clientId 与 Token 是否匹配
   - 触发 @SaCheckPermission 注解的权限校验
   ⚠️ 校验 @SaCheckPermission("system:user:add")，权限不足直接抛出异常终止请求
   
→ 进入 SysUserController.add()

### 第三层：AOP 切面链
**执行入口**：SysUserController.add(@RequestBody SysUserBo user)
6. **DataPermissionAdvice**（数据权限切面）
   位置：DataPermissionPointcutAdvisor.java
   条件：Mapper 方法有 @DataPermission 注解时触发
   功能：在 ThreadLocal 中设置数据权限注解信息
   注意：Controller 层通常不触发此切面
7. **RepeatSubmitAspect** (@Before)
   位置：RepeatSubmitAspect.java:41
   触发条件：@RepeatSubmit() 注解
   功能：
   - 生成唯一 key: MD5(token + 请求参数)
   - Redis.setObjectIfAbsent() 防重复提交
   - key 已存在则抛出 ServiceException
8. **LogAspect** (@Before)
   位置：LogAspect.java:51
   触发条件：@Log(title = "用户管理", businessType = INSERT)
   功能：
   - 创建 StopWatch 开始计时
   - 存储到 ThreadLocal.KEY_CACHE
   
→ 执行 Controller 方法体
```
deptService.checkDeptDataScope(user.getdeptId())
        ↓
userService.insertUser(user)
        ↓
进入 SysUserService.insertUser()
```

### 第四层：MyBatis-Plus 拦截器链（执行 SQL 时）
**执行入口**：SysUserMapper.insert(user) → 执行 INSERT 语句
9. **TenantLineInnerInterceptor**（多租户插件）
   位置：MybatisPlusConfig.java:42-46
   功能：
   - 自动在 SQL 中插入 tenant_id 字段
   - INSERT 时: SET tenant_id = 当前租户ID
   - SELECT/UPDATE/DELETE: WHERE tenant_id = 当前租户ID
   ⚠️ 必须放在第一位（注释明确说明）
10. **PlusDataPermissionInterceptor**（数据权限拦截器）
    位置：MybatisPlusConfig.java:48
    功能：修改 WHERE 条件，添加数据权限过滤；beforeQuery/beforePrepare 解析并重写 SQL
    注意：INSERT 操作通常不触发此拦截器
11. **PaginationInnerInterceptor**（分页插件）
    位置：MybatisPlusConfig.java:50
    功能：查询时自动添加 LIMIT 语句；INSERT/UPDATE/DELETE 不触发
12. **OptimisticLockerInnerInterceptor**（乐观锁插件）
    位置：MybatisPlusConfig.java:52
    功能：UPDATE 时检查 version 字段；INSERT 不触发
    
→ 执行原生 SQL

### 第五层：返回时的处理
**执行入口**：SQL 执行成功，返回结果
13. **LogAspect** (@AfterReturning)
    位置：LogAspect.java:63
    功能：
    - stopWatch.stop() 计算耗时
    - 构建 OperLogEvent 事件
    - SpringUtils.context().publishEvent(operLog)
    - 异步保存操作日志到数据库
14. **RepeatSubmitAspect** (@AfterReturning)
    位置：RepeatSubmitAspect.java:77
    功能：
    - 检查返回值 R<Void>.getCode()
    - 成功 (code == 0): 保留 Redis key 防止重复
    - 失败: 删除 Redis key 允许重试
15. **PlusWebInvokeTimeInterceptor** (afterCompletion)
    位置：PlusWebInvokeTimeInterceptor.java:104
    功能：
    - stopWatch.stop()
    - 输出 "[PLUS]结束请求 => 耗时: XXX毫秒"
    
→ 返回 HTTP 响应

## 总结：执行顺序
Filter 链 → MVC 拦截器 → AOP 切面 → Controller → Service → MyBatis 拦截器 → SQL

### 关键点
1. Filter 最早执行，按 order 值从小到大排序
2. Sa-Token 权限校验在 MVC 拦截器阶段，早于 AOP 切面
3. @RepeatSubmit 防重复提交在 AOP 切面阶段触发
4. @Log 日志记录跨越 Controller 方法整个执行周期
5. MyBatis 拦截器仅在执行 SQL 时触发，非 SQL 操作不生效
6. 多租户拦截器必须放在 MyBatis 拦截器链第一位

## 关键代码位置速查
| 组件                          | 文件位置                         | 行号 | 关键代码                                                     |
|-------------------------------|----------------------------------|------|--------------------------------------------------------------|
| CryptoFilter                  | ApiDecryptAutoConfiguration.java | 27   | order = HIGHEST_PRECEDENCE                                    |
| XssFilter                     | FilterConfig.java                | 28   | order = HIGHEST_PRECEDENCE + 1                                |
| SaInterceptor                 | SecurityConfig.java              | 51   | registry.addInterceptor(new SaInterceptor(...))               |
| PlusWebInvokeTimeInterceptor  | ResourcesConfig.java             | 31   | registry.addInterceptor(new PlusWebInvokeTimeInterceptor())   |
| RepeatSubmitAspect            | RepeatSubmitAspect.java          | 41   | @Before("@annotation(repeatSubmit)")                          |
| LogAspect                     | LogAspect.java                   | 51   | @Before(value = "@annotation(controllerLog)")                 |
| TenantLineInnerInterceptor    | MybatisPlusConfig.java           | 42   | // 多租户插件 必须放到第一位                                  |
| PlusDataPermissionInterceptor | MybatisPlusConfig.java           | 48   | interceptor.addInnerInterceptor(dataPermissionInterceptor())  |
| SysUserController.add         | SysUserController.java           | 158  | @SaCheckPermission("system:user:add")                         |

## 需要特别注意的设计细节
1. Filter 无明确 @Order：通过 @FilterRegistration 注解的 order 属性控制，order 值越小优先级越高
2. Spring MVC 拦截器按注册顺序执行：ResourcesConfig 先注册性能拦截器，SecurityConfig 后注册 Sa-Token 拦截器
3. AOP 切面无显式 @Order：Spring 按切面类的类名或 Bean 注册顺序决定，可通过 @Order 注解手动调整
4. MyBatis 拦截器链必须严格控制顺序：多租户 → 数据权限 → 分页 → 乐观锁