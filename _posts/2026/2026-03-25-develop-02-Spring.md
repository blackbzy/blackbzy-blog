---
title: 开发_02_Spring
description: Spring相关基础知识
date: 2026-03-25
categories:
  - develop
tags:
  - develop
author: blackbzy
update_date: false
pin: false
toc: true
comments: true
image:
  path: tech/spring.png
  alt: spring
---

> Spring是god
{: .prompt-info }


[spring官网](https://spring.io/quickstart)
## 1 Spring 核心
- IOC 原理
- AOP 原理
- Bean 生命周期 ：实 赋 初 销
- 注入Bean 的方式
- Bean 的作用域和实例化注解
- BeanFactory vs ApplicationContext
- Spring事务的传播行为和隔离级别
- 用到了哪些设计模式
  **AOP** = **Aspect-Oriented Programming（面向切面编程）**
  1️⃣ 横切关注点（Cross-Cutting Concerns）
- 事务管理（`@Transactional`）
- 日志打印
- 权限校验
- 缓存切面

### 1.1AOP 的实现方式
1️⃣ JDK 动态代理
- 代理接口
- 如果目标对象实现了接口，就用它
  2️⃣ CGLIB
- 代理类
- 目标类没接口时使用
- 创建子类覆盖方法
> ⚠️ Spring Boot 只是把 Spring AOP 封装了一层，但本质还是 Spring AOP。

1️⃣ `@Aspect` 注解
```java
@Aspect  
@Component  
public class LogAspect {  
    @Before("execution(* com.example..*Service.*(..))")  
    public void before() {  
        System.out.println("方法执行前");  
    }  
}
```
- 注解式切面
- 和 Spring AOP 核心原理一样
- `@Aspect` + `@Before/@After/...` 注解
- Spring Boot 自动扫描 `@Component` + `@EnableAspectJAutoProxy`

2️⃣ Spring AOP 原始接口（如 `org.aspectj.lang.ProceedingJoinPoint`）
```java
@Around("execution(* com.example..*Service.*(..))")  
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {  
    System.out.println("前置逻辑");  
    Object result = joinPoint.proceed(); // 调用原方法  
    System.out.println("后置逻辑");  
    return result;  
}
```

- 这里的 `ProceedingJoinPoint` 是 AOP 核心接口
- 提供访问目标方法、参数、返回值的能力
- 可以手动控制方法执行

>Spring AOP 是技术实现，@Aspect 是 Spring Boot 提供的注解方式来声明切面，使使用更简单，底层还是 Spring AOP。
## 2 Spring Boot
- 自动配置原理
- starter 原理
- @Conditional
- @EnableAutoConfiguration
- Mybatis的相关标签
## 3 Spring MVC
请求流程：

```
DispatcherServlet
HandlerMapping
HandlerAdapter
ViewResolver
```

## 事务失效场景
**你是否理解事务是怎么生效的（AOP代理）**。
1️⃣ **自调用**（最常见）
```java
public void A() {  
B(); // ❌ 事务失效  
}  
  
@Transactional  
public void B() {}
```
原因：
- A 调用 B 是 **this.B()**
- 没走代理
  正确做法：
- 通过代理调用（比如从 Spring 容器拿自己）

2️⃣ **方法不是 public**
```java
@Transactional  
private void B() {}
```
原因：
- Spring 默认基于 **动态代理**
- 只能拦截 public 方法

3️⃣ **异常被吃掉**：多层调用异常没抛出/异常被 try-catch 吃掉
```java
@Transactional  
public void test() {  
    try {  
        int i = 1 / 0;  
    } catch (Exception e) {  
        // 什么都不做  
    }  
}
```
原因：
- Spring 只在“异常抛出”时回滚

4️⃣ 抛出的不是运行时异常 ： Checked Exception 不回滚
```java
@Transactional  
public void test() throws Exception {  
    throw new Exception();  
}

```
原因：
- 默认只回滚：
  RuntimeException / Error
  解决：
```java
@Transactional(rollbackFor = Exception.class)
```

5️⃣ 数据库**不支持事务** / 引擎问题
比如 MySQL：
- MySQL 的 **MyISAM 引擎**
  不支持事务 ❌
  正确：
- 使用 InnoDB

6️⃣ 没被 Spring 管理
```java
new OrderService().createOrder();
```

原因：
- 没走 Spring 容器
- 没有代理对象

7️⃣ 事务传播行为导致“看起来失效”
```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
```
场景：
- 外层事务回滚
- 内层新事务提交
  现象：
  数据“部分提交”

8️⃣ **多线程导致事务失效**

```java
@Transactional  
public void test() {  
    new Thread(() -> {  
        // ❌ 没有事务  
    }).start();  
}
```
原因：
- 事务是绑定在当前线程的（ThreadLocal）

9️⃣ 只读事务写数据
```java
@Transactional(readOnly = true)
```
有些数据库会：
- 忽略写操作
- 或优化导致异常行为


## 事务使用
@Transactional 的 rollback 相关参数
```java
//指定哪些异常需要回滚
@Transactional(rollbackFor = Exception.class)
```
回滚的触发方式
1️⃣ 抛出异常（核心触发点）
2️⃣ 手动触发回滚
```java
import org.springframework.transaction.interceptor.TransactionAspectSupport;

@Transactional
public void test() {
    try {
        // 业务逻辑
    } catch (Exception e) {
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```
3️⃣ Error 也会触发回滚



---
故事未完:96
**Thoughts**:: justdoit.
