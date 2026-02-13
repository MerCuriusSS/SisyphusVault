---
tags:
  - Areas/Coder/javaWeb
  - Areas/Coder/基础原理
category: 技术
status: 加工
project: "[[02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: 租户逻辑隔离
source:
---
>**笔记不是为了复述书本，而是为了**“存档当下的自己”。如果你的笔记里没有你的思考痕迹、痛苦经历和选择理由**，它就只是一份毫无生命力的说明书，自然无法在未来唤醒你的认知***


## 💥 核心结论
>核心定义是什么? 核心价值在哪里?

### 🟣 核心定义：以「**租户**」（组织/机构）为单位，使「数据、权限、配置、资源」使用相互**独立**，但整体**复用**同一套代码逻辑和部署实例。
### 🟣 核心价值：一套代码，多租户独立使用，隔离且复用。

## 🔪 我的见解
>什么要记录它？它解决了什么我以前解决不了的问题？

### 🟣 痛点：
#### 1. 人力不足，维护成本高昂：
- 多客户多部署实例，资源使用、监控成本上升。
-  一个BUG修复需要为所有客户系统都修复一次，人力不足。
#### 2. 定制化需求难管理：
- 不同用户用不同系统版本，新功能难同步上线。
- 不同定制化需求让代码分支泛滥，难管理。

### 🟣 解决方案：
#### 1. 租户id逻辑隔离，共用一套代码、一套部署实例，所有租户共用资源。
#### 2. 共用一套代码，版本统一、定制化逻辑通过配置实现，维护代码统一。

## ⚡️ 我的重构
>它的底层逻辑是什么？（尝试用最简单的类比解释给外行听）它的结构是什么?

### 🟣 底层逻辑：为大部分业务表、权限配置表统一添加`tenant_id`字段作为租户标识，在SQL执行前通过拦截器自动在WHERE 子句中添加基于租户ID的过滤条件。

>**添加tenant_id基准：该表数据是否「归属于特定租户」 + 是否「需要在租户间隔离」**

### 🟣 结构：

#### 1.核心组件
- 租户上下文（tenantContext）
- 租户拦截器（tenantInterceptor）& 租户处理器（tenantHandler）
- 用户登录状态上下文（SaToken）
- 缓存组件（Redis）
#### 2.「组件」流程概览图：[「多租户隔离」流程概览图](excalidraw/「多租户隔离」流程概览图.md)
#### 3.「数据流转」流程图：[「多租户隔离」数据流转图](excalidraw/「多租户隔离」数据流转图.md)

## 🚀 实践应用：

### 🟣 最小化实践&ruoyi原实现： [Saas多租户核心源码](Saas多租户核心源码.md)
### 🟣 「最小实践」与「ruoyi实现」比对

| 特性     | 最小化实践                     | ruoyi实现              | 核心逻辑                                                                                |
| ------ | ------------------------- | -------------------- | ----------------------------------------------------------------------------------- |
| 租户切换   | 依赖登录用户，切换用户会登录失效          | 不依赖登录态（定时任务、超级管理员切换） | 1）Redis专门保留动态租户ID<br>2）Redis全局（global）存Token，不跟租户绑定<br>3）查数据时优先用这个 ID，不用重新登录        |
| 业务数据缓存 | 需要手动添加「租户ID」前缀作租户隔离，以防串数据 | 自动添加「租户ID」前缀         | 1）框架自动给 Key 加 “租户 ID 前缀”，不同租户的 Key 不一样；<br>2）如果是全局数据（有global前缀，如config/login），就不加前缀 |
| 特定表共享  | 无差别对表进行「租户隔离」，不可区分        | 可对特定表实现「忽略租户ID」      | 查租户表、全局配置这些共享数据时，先开 “忽略租户开关”（ThreadLocal 存个 true），框架查数据库时就不加 tenant_id 条件，查完再关掉开关   |

### 🟣 实战演示
#### 1.Controller层
##### 普通接口（自动租户隔离）
```java
@RestController
@RequestMapping("/demo/product")
@RequiredArgsConstructor
public class ProductController extends BaseController {

    private final IProductService productService;

    // 查询列表（只返回当前租户数据）
    @SaCheckPermission("demo:product:list")
    @GetMapping("/list")
    public TableDataInfo<ProductVo> list(ProductBo bo, PageQuery pageQuery) {
        return productService.queryPageList(bo, pageQuery);
    }

    // 新增（自动归属当前租户）
    @SaCheckPermission("demo:product:add")
    @PostMapping()
    public R<Void> add(@Validated @RequestBody ProductBo bo) {
        return toAjax(productService.insert(bo));
    }
}
```

####
## ⛪ 场景设想

### 🟣 SaaS CRM 系统
- **业务流程**：企业 A（租户 001）和企业 B（租户 002）都用这套 CRM，销售只能看自己企业的客户数据，超管能看所有企业数据
- **描述**：
	- **销售登录后**，框架自动给 SQL 加 `tenant_id=001`，Redis 缓存 Key 加 `001:`，只能看到自己企业的客户；
	- **超管登录后**，通过 `TenantHelper.dynamic("002")` 切换到企业 B，无需重新登录，直接查看 B 的客户数据；
	- **系统字典**（如客户等级）是全局配置，Key 为 `global:dict:customer_level`，所有租户共享，无需重复维护。

### 🟣电商平台商家后台（定时任务场景）
- **业务流程**：每日凌晨 2 点，系统自动统计每个商家（租户）的订单数据，生成报表。
- **描述**：
	- 定时任务无登录用户，先通过 `ignore()` 开关查询所有启用的商家列表（sys_tenant 表）；
	- 遍历每个商家，调用 `dynamic(tenantId, () -> { 统计订单 })`，自动给 SQL / 缓存加对应租户前缀；
	- 执行完每个租户后，ThreadLocal 自动清理租户 ID，避免下一个租户串值；