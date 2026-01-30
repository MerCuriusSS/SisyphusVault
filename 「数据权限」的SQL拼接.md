---
tags:
  - Areas/开发/javaWeb
  - Areas/开发/基础原理
category: 技术
status: 加工
project: "[[../../02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: 数据权限划分
source:
---


## 💥 核心定义

 🔴 概念：通过**非硬编码**的方式将**不同权限**角色的**查询范围**SQL**拼接**到业务SQL后面。【在SQL执行前，通过MyBatis拦截器自动在WHERE子句中添加基于用户角色的权限过滤条件】
 
 🔴 核心目的：将「业务逻辑」与「权限规则」进行**解耦**，以**非硬编码**、非侵入的**配置**方式实现**降低思考负担**。
【大白话：手动拼接权限SQL会导致100个mapper要改动100次的屎山代码，动作重复效率低，不如“一次配置，自动拼接”】


## 🔪 解构

🔴 「组件」流程概览图：[「数据权限SQL拼接」流程概览图](excalidraw/「数据权限SQL拼接」流程概览图.md)

🔴 「数据流转」流程图：[「数据权限SQL拼接」数据流转图](excalidraw/「数据权限SQL拼接」数据流转图.md)

🔴 底层逻辑：

🔴 失效边界：

## ⛪ 最小化实践

🔴 注册拦截器
```java
public class MyBatisConfig {  
  
    /**  
     * 注册数据权限拦截器  
     *  
     * 方式1：通过代码注册（推荐）  
     */  
    @Bean  
    public String registerDataPermissionInterceptor(SqlSessionFactory sqlSessionFactory) {  
        // 注册简化版拦截器  
        sqlSessionFactory.getConfiguration()  
            .addInterceptor(new SimpleDataPermissionInterceptor());  
  
        // 或者注册JSQLParser版拦截器  
        // sqlSessionFactory.getConfiguration()  
        //     .addInterceptor(new JsqlParserDataPermissionInterceptor());  
        return "DataPermissionInterceptor registered";  
    }
}
```

🔴 注册拦截器