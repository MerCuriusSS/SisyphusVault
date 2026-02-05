---
tags:
  - Areas/开发/基础原理
  - Areas/开发/javaWeb
category: 技术或思维
status: 加工
project: "[[../../02 - Projects/后台管理系统「Ruoyi-Vue-Plus」|后台管理系统「Ruoyi-Vue-Plus」]]"
application: SQL解析与拼接
source:
---
>**笔记不是为了复述书本，而是为了**“存档当下的自己”。如果你的笔记里没有你的思考痕迹、痛苦经历和选择理由**，它就只是一份毫无生命力的说明书，自然无法在未来唤醒你的认知***


## 💥 核心结论 (3秒原则)

🔴 概念：将字符串形式的SQL抽象成由一块块结构化的组件（select部分、where部分...）组成的抽象语法树（AST）

🔴 核心目的：拼接SQL不再因为定位问题导致语法错误；可实现复杂SQL查询（join、子查询、union等）

## 🔪 我的见解/重构

🔴 「SQL控制用户数据权限」痛点：

1. 朴素的SQL字符串拼接方式（where条件）定位麻烦，需要索引判断where是否存在？limit是否存在？order是否存在；**JSQLParser**能够直接提供where结构，节省大量条件判断；
2. 朴素的SQL字符串拼接方式难以处理复杂查询（join、子查询等）；**JSQLParser**则可以通过树的递归遍历实现复杂查询的拼接。

🔴 底层逻辑：
- 将完整SQL拆解一个个「词语」。如`SELECT name FROM user WHERE age > 18` 会被拆成`SELECT`、`name`、`FROM`、`user`、`WHERE`、`age`、`>`、`18`。
- 按照「SQL语法规则」搭成一棵树
```
查询(Select)
├─ 要查的东西: 名字(name)
├─ 从哪查: 用户表(user)
└─ 条件(Where)
   └─ 比较(>)
      ├─ 左边: 年龄(age)
      └─ 右边: 18
 ##################################################################
 
     
含有子查询:[SELECT name FROM user WHERE age > (select age from user1)]


查询(Select)
├─ 要查的东西: 名字(name)
├─ 从哪查: 用户表(user)
└─ 条件(Where)
   └─ 比较(>)
      ├─ 左边: 年龄(age)
      └─ 右边: 子查询(subSelect)
                ├─ 要查的东西: 名字(age)
                ├─ 从哪查: 用户表(user1)
                
 ##################################################################
 
 
 集合操作：[SELECT name FROM student UNION SELECT name FROM teacher;]
 
 
 集合操作(UNION)
├─ 查询1 (student表)
│   └─ 查name列
└─ 查询2 (teacher表)
    └─ 查name列

 ##################################################################
 
 
 JOIN 多表：[SELECT o.id, u.name FROM order o JOIN user u ON o.user_id = u.id WHERE o.status = 1;]
 
 
 查询
├─ 查列: o.id, u.name
├─ 主表: order (别名 o)
├─ 连接表: user (别名 u) ON o.user_id = u.id
└─ 条件: o.status = 1
```
- 再按照实际插入目标比如`where`在树中找到对应的位置并加入进去。
## ⛪ 场景设想

- **场景 A**：简单实现外层SQL查询
```java
public void simpleParser(){
	// 使用JSQLParser解析和修改SQL
	Statement statement = CCJSqlParserUtil.parse(originalSql);
	
	if (statement instanceof Select) {  
	    Select select = (Select) statement;  
	    processSelect(select, dataPermission, currentUser);  
	  
	    String newSql = select.toString();
	}
}

//解析Select内容
private void processSelect(Select select, DataPermission dataPermission, LoginUser user) {  
  
    if (select.getSelectBody() instanceof PlainSelect) {  
        PlainSelect plainSelect = (PlainSelect) select.getSelectBody();  
        Expression where = plainSelect.getWhere();  
  
        // 构建权限条件  
        Expression permissionCondition = buildPermissionCondition(dataPermission, user);  
        if (permissionCondition != null) {  
            if (where != null) {  
                // 已有WHERE条件，使用AND连接  
                AndExpression andExpression = new AndExpression(where, permissionCondition);  
                plainSelect.setWhere(andExpression);  
            } else {  
                // 没有WHERE条件，直接设置  
                plainSelect.setWhere(permissionCondition);  
            }  
        }  
    }  
}

```
- **场景 B：** 实现复杂SQL查询
```java

```