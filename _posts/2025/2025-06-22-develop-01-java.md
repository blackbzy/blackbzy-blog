---
title: 开发_01_java基础
description: 基础汇总
date: 2023-06-22
categories:
  - develop
tags:
  - develop
  - java
author: blackbzy
update_date: 2025-10-11
pin: false
toc: true
comments: true
image:
  path: /tech/Java.png
  alt: logo
---

> 基础概念和tips
{: .prompt-info }

[java官方文档](https://docs.oracle.com/en/java/javase/26/docs/api/index.html)
## 1 Java 基础语法
- String 为什么不可变
- StringBuilder / StringBuffer
- 封装、继承、多态的实现和特点
- Exception 和 Error 有什么区别
- equals vs ==
- hashCode 与 equals 的关系
- 什么是值传递
- final static 关键字含义和使用场景
- abstract vs interface
- overWrite 和 overRode
- 深拷贝 vs 浅拷贝
- io流相关知识
  - 序列化的应用场景和协议
  - BIO NIO AIO 分别代表什么
- 反射使用到的技术和应用
- SPI 机制实现方式，优点和缺点？
- 动态代理和静态代理的区别，还有使用场景
- java8和java21的新特性，使用场景
- happens-before
- java热部署
### tips：
- BigDecimal 比较
  - equals 比较：值 + 精度(scale)
  - compareTo比较值
- 字符串比较，常量前置
- Optional.ofNullable(null);
- 自动拆箱（隐蔽 NPE空指针）

```java
Integer a = null;  
int b = a; // NPE

```

- switch 穿透（忘写 break）

```java
switch (type) {
    case 1:
        doA();
    case 2:
        doB();
}

```

- SimpleDateFormat多线程不安全，应该使用DateTimeFormatter
- 浮点数比较：浮点数有精度问题
  - 一般业务：只要误差小于 0.000001，就认为两个数“相等”
  - 科学计算：`1e-9` 或更小
  - 金融（金额）：直接 **BigDecimal**（还是推荐直接BigDecimal）

```java
Math.abs(a - 0.3) < 1e-6

```

- JSON 反序列化精度丢失

```java
//不能用double，
double amount = json.getDouble("amount");
//而是BigDecimal
BigDecimal amount = json.getBigDecimal("amount");

```

- finally中return 覆盖原逻辑 return

```java
try {
    return 1;
} finally {
    return 2;
}

```

- 序列化版本号：反序列化会失败

```java
private static final long serialVersionUID = 1L;

```

- 枚举比较：枚举就是单例，不推荐equals（容易NPE空指针）

```java
if (status == Status.SUCCESS)

```

- ThreadLocal 内存泄漏：在线程池环境下 → 内存泄漏

```java
try {
    local.set("data");
} finally {
    local.remove();
}

```

- LocalDateTime 时区问题

```java
//没有时区
LocalDateTime.now()
//存 UTC
ZonedDateTime.now(ZoneId.of("UTC"))

```

- 继承 + 构造方法调用顺序

```java
class A {
    A() {
        print();
    }
}

class B extends A {
    int x = 10;

    void print() {
        System.out.println(x);
    }
}
//输出 0（不是10）
//父类构造先执行
//子类字段还没初始化

```

- 异常处理
  - 手动抛出异常， new 一个异常对象抛出。
  - 抛出的异常可读。
  - 避免重复记录日志。重复打印记录日志难以定位问题，使得问题更难以追踪和解决。

## 2 Java 集合
### List
- ArrayList 原理
- LinkedList 原理
- ArrayList 扩容机制
- fail-fast 机制
- HashSet LinkedHashSet TreeSet
- HashMap和ConcurrentHashMap比较
- 排序： Comparable 和 Comparator 的区别

tips：
- 用 isEmpty() 而非 size() == 0
- 集合foreach中不做add remove元素操作 ，该用itreater删除元素
  - Stream流 中同样不能 删除元素
- HashMap key 可变：
  - 作为 key 的对象必须不可变
  - 或重写JavaBean的 `hashCode + equals`
- List 转数组应该使用

```java
list.toArray(new String[0]);

```

- subList ：subList 是原 list 的视图

```java
List<String> sub = list.subList(0, 2);
list.clear();
sub.get(0); // 报错
//应该新建
new ArrayList<>(list.subList(0, 2))

```

- Arrays.asList 返回的是**固定大小列表**，不能再添加元素

```java
new ArrayList<>(Arrays.asList("a", "b"))

```

- 接口应当返回空集合，而不是`null`
### Map
- HashMap 原理和扩容机制
- HashMap 扩容
- HashMap 1.7 vs 1.8
- 为什么要红黑树
- Hash 冲突解决
- ConcurrentHashMap 的1.7和1.8实现
- segment vs CAS
- size() 为什么不准

## 3 Java 并发
- Thread的 run 和 start 方法
- Thread vs Runnable vs Callable
- Future
- ThreadLocal
  - threadLocal  内存泄漏
- 线程通信方法
- 锁：synchronizd、lock、ReenTrantLock
  - synchronizd锁升级
- 乐观锁，悲观锁
  - 公平锁，非公平锁
  - 可中断，不可中断
- violate
  -  内存可见性
- CAS 算法存在哪些问题
  -  ABA
  -  循环开销
  - 单个变量控制
- 线程池参数和调用逻辑、类型
- 线程数设置逻辑
- 并发工具 CountDownLatch 、Semaphare、CycleBarrier
- AQS 是什么
  - CLH锁队列

tips：
- try-with-resources优于自己在 finally 中释放资源
### JUC
- ThreadPoolExecutor
- 线程池参数
- 队列类型

```
corePoolSize
maximumPoolSize
workQueue
keepAliveTime
RejectedExecutionHandler

```


## 4 JVM
JVM 内存结构

```
堆
方法区
虚拟机栈
程序计数器
本地方法栈

```

GC

- CMS 垃圾回收方法
- G1 垃圾回收方法
- Full GC 触发条件
- 回收条件，晋升条件
- Minor GC
- 什么情况会触发频繁GC，怎么排查，解决

### JVM 调优
常见：
1. 命令行
  1. `jstat` 类信息、内存、垃圾收集。。
  2. `jmap` 堆转储快照
  3. `jstack` 当前时刻的线程快照
2. jconsole 分析死锁，内存监控
3. Visual VM
4. MAT（Memory Analyzer Tool）   堆内存离线分析工具

JVM 核心参数调优 (The Engine)：
金融系统的首要目标是**避免 Full GC 导致的系统停顿（Stop-the-world）**。

A. 内存空间锁定
- **`-Xms` / `-Xmx`**: 必须设为相等。
  > 理由：防止 JVM 在运行时**动态调整堆大小**导致性能抖动。例如：`-Xms8g -Xmx8g`。
- **`-XX:+AlwaysPreTouch`**:
  > 启动时即分配所有物理内存。这会稍微增加启动时间，但能避免运行时申请内存页带来的延迟（Page Fault）。
- **`-Djava.security.egd=file:/dev/./urandom`**:
  >  Linux 下 `SecureRandom` 默认使用 `/dev/random`，在加密/签名密集的金融场景下，如果熵池耗尽，线程会陷入长达数秒甚至分钟的阻塞。
- **`-Xss256k`**:
  > 减小单个线程栈大小（默认 1M）。如果并发连接极高，减小此值可以节省大量非堆内存，减少内存溢出风险。
- **`-XX:MetaspaceSize=256m` / `-XX:MaxMetaspaceSize=512m`**:
  > 固定元空间大小，防止因为类加载过多触发 Full GC。

B. 垃圾回收器 (GC) 选择
- G1 GC (主流选择): 适用于 4G~30G 左右的堆。
  - `-XX:+UseG1GC`
  - `-XX:MaxGCPauseMillis=200`: 设置最大停顿目标（默认 200ms），金融系统可尝试调至 100ms。
- ZGC (极低延迟需求): 适用于大内存且对停顿极其敏感（<10ms）的系统。
  - `-XX:+UseZGC` (Java 15+ 生产可用)

C. 故障诊断与日志
- **`-XX:+HeapDumpOnOutOfMemoryError`**: OOM 时自动生成快照。
- **`-XX:HeapDumpPath=/data/logs/`**: 指定快照路径。
- **`-Xlog:gc*:file=/data/logs/gc.log:time,level,tags`**: (Java 9+) 现代化的 GC 日志打印。


Tomcat 连接器调优 (The Gate)
Tomcat 的核心性能取决于 `server.xml` 中的 `<Connector>` 配置。
- **`protocol="org.apache.coyote.http11.Http11NioProtocol"`**: 确保使用非阻塞 I/O。
- **线程池配置**:
  - **`maxThreads="800"`**: 最大工作线程数。金融场景通常设为 500-1000。
  - **`minSpareThreads="100"`**: 始终保持存活的线程，应对突发流量。
- **队列与超时**:
  - **`acceptCount="1000"`**: 当所有线程忙碌时，允许在队列中排队的请求数。
  - **`connectionTimeout="20000"`**: 连接超时时间（毫秒）。



---
故事未完:173
**Thoughts**:: justdoit.
