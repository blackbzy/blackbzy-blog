---
title: java-review
description: 复盘
date: 2025-06-22
categories:
  - back-end
tags:
  - back-end
author: blackbzy
update_date: false
pin: false
toc: true
comments: true
image:
  path: /tech/Java.png
  alt: logo
---

> 准备结束,回忆路径
{: .prompt-info }

## 1.基础

1. 封装、继承、多态的实现和特点是什么？
2. 深拷贝和浅拷贝区别：内部对象为引用类型
3. Exception 和 Error 有什么区别
4. 值传递&引用传递区别？
5. 泛型和通配符的使用和关联？
6. BIO、NIO 和 AIO 的区别？
7. 动态代理和静态代理区别？
8. SPI 机制实现方式，优点和缺点？

###  基本类型和包装类型区别？

| 功能 | 基本类型          | 包装类型 |
| ---- | ------------- | ------- |
| 用途   | 一些常量和局部变量     | 对象属性，泛型 |
| 存储方式 | 局部变量存放栈中      | 堆中      |
| 占用空间 | 小             | 对象空间    |
| 默认值  | 有             | 无       |
| 比较方式 | == （建议equals） | equals  |

###  String、StringBuffer、StringBuilder 的区别？

尽量不使用+ ，会有额外对象开销。

| 特性 | String                  | StringBuffer | StringBuilder                                 |
| ----- | ----------------------- | ------------ | --------------------------------------------- |
| 可变性   | 不可变的                    | 可变           | 可变                                            |
| 线程安全性 | 线程安全                    | 线程安全         | 非线程安全                                         |
| 性能    | 每次改变会生成一个新的 `String` 对象 | 对象本身进行操作     | 对象本身进行操作，比`StringBuffer` 仅能获得 10%~15% 左右的性能提升 |

### 异常使用有哪些需要注意的地方？
- 手动抛出异常， new 一个异常对象抛出。
- 抛出的异常可读。
- 避免重复记录日志。重复打印记录日志难以定位问题，使得问题更难以追踪和解决。

### jvm相关  
1. 虚拟机内存分配：线程隔离  
2. 类加载过程
	1. 类加载器
3. java热部署  
4. GC内存分配和gc的不同收集器  
5. happens-before  
6. jvm调优  
7. ==  和 equals  
### 序列化
- 比较常用的序列化协议有 Hessian、Kryo、Protobuf、ProtoStuff
- JSON 和 XML 这种属于文本类序列化方式。虽然可读性比较好，但是性能较差。
- 对于不想进行序列化的变量
	- 可以使用 `transient` 关键字修饰。
	- `static` 变量


## 2.集合  
1. ArrayList和LinkedList比较  
2. HashMap和ConcurrentHashMap比较  
3. ArrayList 的扩容机制
4. HashMap 的扩容机制
5. 排序： Comparable 和 Comparator 的区别
6. ConcurrentHashMap 线程安全的具体实现方式(1.7和1.8的区别)


| 特性 | List                        | Set                           | Queue                                     | Map                           |
| ------ | --------------------------- | ----------------------------- | ----------------------------------------- | ----------------------------- |
| 顺序性    | 有序，按插入顺序                    | 无序                            | 有序                                        | 无序                            |
| 唯一性    | 可重复                         | 不可重复                          | 可重复                                       | key唯一，value可重复                |
| 接口实现   | ArrayList，LinkedList        | HashSet，LinkedHashSet，TreeSet | PriorityQueue，DelayQueue，ArrayDeque       | HashMap，LinkedHashMap，TreeMap |
| 线程安全实现 | CopyOnWriteArrayList，Vector | CopyOnWriteArraySet           | ConcurrentLinkedQueue，LinkedBlockingQueue | ConcurrentHashMap             |


  
## 3.多线程  
1. 线程  
2. 线程通信方法  
3. 锁：synchronizd、lock、ReenTrantLock  
	1. synchronizd锁升级  
	2. 乐观锁，悲观锁
	3. 公平锁，非公平锁
	4. 可中断，不可中断
4. violate  
	1. 内存可见性
5. CAS 算法存在哪些问题
	1. ABA
	2. 循环开销
	3. 单个变量控制
6. threadLocal  内存泄漏
7. 线程池参数和调用逻辑、类型  
	1. 线程数设置逻辑
8. 并发工具 CountDownLatch 、Semaphare、CycleBarrier  
9. AQS 是什么
	1. CLH锁队列
  
## 4.IO  
1. 概念：同步/异步 阻塞/非阻塞  
2. BIO NIO AIO  
3. 字节流 ，字符流，字节缓冲流，（字符缓冲流）
  
## 5.生产问题排查  
1. 参数  
2. GC  
3. OOM  
4. 线程占用

排查工具:
1. 命令行
	1. `jstat` 类信息、内存、垃圾收集。。
	2. `jmap` 堆转储快照
	3. `jstack` 当前时刻的线程快照
2. jconsole 分析死锁，内存监控
3. Visual VM
4. MAT（Memory Analyzer Tool）   堆内存离线分析工具

---
故事未完:173
**Thoughts**:: justdoit.
