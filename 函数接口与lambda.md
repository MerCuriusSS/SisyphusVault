---
tags:
  - Areas/Coder/基础原理
category: 技术
status: 加工
project:
application:
source:
---
>**笔记不是为了复述书本，而是为了**“存档当下的自己”。如果你的笔记里没有你的思考痕迹、痛苦经历和选择理由**，它就只是一份毫无生命力的说明书，自然无法在未来唤醒你的认知***


## 💥 核心结论
>核心定义是什么? 核心价值在哪里?

### 核心定义：函数接口是只有一个抽象方法的接口；lambda是一段可传递的「行为」，是函数接口的具体实现。

### 核心价值：能够传递「行为」（一段代码），并控制其「何时使用」，而非已经执行的「死结果」


## 🔪 我的见解
>为什么要记录它「我遇到了什么问题」？它能解决哪些痛点？

### 为什么要记录它？
- 项目中经常使用函数式接口（supplier），存在阅读障碍
- 一直弄不懂函数接口的核心价值，不知道使用场景，为什么用？

### 解决痛点
- 无法控制某段代码的执行顺序，传统方式只能传执行结果，相当于代码在方法中先执行
- 方法中存在大量重复代码，只有某段代码存在变化，变化的代码因无法抽象出来，只能改源码/生成新方法再重复编写新代码

## ⚡️ 我的重构
>它的底层逻辑是什么？（尝试用最简单的类比解释给外行听）它的结构是什么?



## 🚀 实践应用：

### 🟣 最佳实践：
当多个方法**大部分代码一模一样，只有一小段逻辑不同**时，把**变化的那部分逻辑**抽象成函数式接口，作为参数传入，实现**一个通用方法适配所有场景**，消除重复代码

## ⛪ 场景设想
- 传统重复代码：
```java
// 筛选成年人
public static List<User> filterAdult(List<User> users) {
    List<User> result = new ArrayList<>();
    for (User user : users) {
        // 只有这里不一样
        if (user.getAge() >= 18) { 
            result.add(user);
        }
    }
    return result;
}

// 筛选男性
public static List<User> filterMale(List<User> users) {
    List<User> result = new ArrayList<>();
    for (User user : users) {
        // 只有这里不一样
        if (user.getSex().equals("男")) {
            result.add(user);
        }
    }
    return result;
}

// 筛选余额>100
public static List<User> filterMoney(List<User> users) {
    List<User> result = new ArrayList<>();
    for (User user : users) {
        // 只有这里不一样
        if (user.getMoney() > 100) {
            result.add(user);
        }
    }
    return result;
}
```

- 函数接口式方法
```java
// 通用筛选方法
public static List<User> filter(
    List<User> users,
    Predicate<User> predicate  // 变化的逻辑，外部传入
) {
    // 以下全是不变的代码
    List<User> result = new ArrayList<>();
    for (User user : users) {
        // 使用传入的逻辑
        if (predicate.test(user)) { 
            result.add(user);
        }
    }
    return result;
}
//=====================调用时===========================

// 筛选成年
filter(users, user -> user.getAge() >= 18);

// 筛选男性
filter(users, user -> user.getSex().equals("男"));

// 筛选余额>100
filter(users, user -> user.getMoney() > 100);
```