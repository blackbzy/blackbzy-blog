---
title: 开发_10_开发增强
description: 一些开发辅助
date: 2025-02-18
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
  path: tech/git.png
  alt: git
---

> 一些开发辅助，帮助提高效率
{: .prompt-info }

## 开发管理
- git
- maven
- ai辅助编码
- 单元测试：junit，mockito
- code review
- CI DI
- DDD领域驱动

### 技术文档怎么出，包括哪些内容
- **开发者文档（API/SDK）：** 面向程序员，侧重接口调用和代码逻辑。
- **用户手册（User Guide）：** 面向最终用户，侧重功能操作和业务流。
- **架构设计说明（Design Doc）：** 面向内部团队，侧重系统设计、数据模型和技术选型。
- **运维文档（Ops Doc）：** 面向运维人员，侧重部署、配置和故障排除。

1简洁描述解决什么问题：
2现状分析与目标 (Current Situation & Goals)
- 现有痛点： 具体的性能数据（如：当前单表 5000 万行，复杂查询耗时 5s+，Tomcat 在峰值时线程耗尽）。
- 优化目标： 量化指标（如：核心接口响应耗时降低至 200ms 以内，支持 3000 并发稳定运行）。
  3总体架构设计：
  4详细技术方案 (Detailed Design)：
- 数据库优化：
  - 索引变更： 列出新增/修改的索引名称、字段及原因。
  - SQL 优化： 对比优化前后的 SQL 语句及 `EXPLAIN` 执行计划。
  - 数据迁移策略： 如果涉及分表，说明存量数据如何平滑迁移，如何回滚。
- 中间件与 JVM 配置：
  - 参数清单： 列出具体的 `JAVA_OPTS`、Tomcat `server.xml` 的关键配置项。
  - 对比表： 调优前后的参数对照。
- 接口/逻辑改动：
  - 如果涉及代码重构，说明类图变化或核心逻辑伪代码。
    5性能压测与风险评估 (Testing & Risk)
- 回滚方案： 发生意外时，如何快速恢复到初始状态（具体操作步骤）。
  6安全与合规 (Security & Compliance)
- 数据脱敏： 接口返回是否涉及敏感字段？
- 日志审计： 关键操作是否留痕？

### 工作流（Workflow）
就是将一项业务任务拆解成多个有序的节点，并让这些节点按照预设的规则自动或半自动流转的过程。
理解工作流，只需抓住这三个词：
- 节点（Node/Task）： “谁来做？”或“做什么？”（比如：提交申请、系统自动发邮件）
- 路由（Route/Transition）： “接下来去哪？”（比如：审批通过走 A 路径，驳回走 B 路径）。
- 规则（Rule/Condition）： “凭什么走？”（比如：金额 > 5000 元需要总监审批，否则经理审批即可）。
  natasha+diagram.js
  Flowable
## flink相关：实时流数据管理
- flink sql状态管理
- check point
- 流计算，实时数据处理

## python

- 脚本
- 自动化
- 基本文件
- 数据库处理
- 第三方api调用

## linux命令
[[07-Linux]]
- shell
- unix环境开发

## ai
- LLm辅助编程：prompt编写
- coze dify 工作流
  - 底层原理
  - 知识库

代码辅助类
- GitHub Copilot
- Cursor
- ChatGPT
- 代码补全
- 生成接口/DTO/CRUD
- 重构代码（比如把同步改异步、优化结构）
  后端开发场景
- **快速生成 Spring Boot 模块骨架**
- **写接口文档**（Swagger/OpenAPI）
- **测试用例**
- 生成 SQL & 优化 explain
- 帮你分析慢查询
- 设计 Redis / MQ 方案
  调试 / 排错能力（很加分）
- WebSocket 收不到数据 → 让 AI 帮你分析可能原因（粘包、协议、编码问题）
- JVM 问题 → 分析 GC、线程阻塞
- 分布式问题 → 排查超时 / 重试 / 幂等
  AI + 工程结合（拉开差距）
- 用 AI 辅助写：
  - 接口幂等设计
  - 分布式事务方案（TCC / 最终一致性）
- 让 AI 帮你做：
  - 技术选型对比（比如 MQ / RPC）

主流代码辅助工具
1️⃣ IDE内嵌型（最常用）
GitHub Copilot
- 自动补全代码
- 生成函数
- 写测试代码
  Codeium
- 免费版很强
- 支持多语言、多IDE
  1️⃣ AI原生IDE
  Cursor
- 可以理解整个项目
- 一次修改多个文件
- 支持“对话式改代码”
  1️⃣ 云IDE / 轻量开发工具

AI Agent（自动写代码）
## Web 网络编程
基于 TCP/IP 协议，通过 Socket 进行进程间通信
网络分层模型
```
应用层   → HTTP / WebSocket
传输层   → TCP / UDP
网络层   → IP
数据链路层 → MAC
物理层
```

浏览器访问一个 URL，本质流程
```
1. DNS 解析 → 得到 IP
2. 建立 TCP 连接（三次握手）
3. 发送 HTTP 请求
4. 服务器处理请求
5. 返回 HTTP 响应
6. 断开连接（四次挥手）或复用连接（keep-alive）
```
实现方式
- BIO（Blocking IO）
- NIO（Non-blocking IO）——多路复用
  - Channel
  - Buffer
  - Selector（多路复用器）
- AIO（Asynchronous IO）——异步非阻塞：Java 实际用得少（成熟度问题）
  -  提交 IO 请求后，不用管
  - 操作系统完成后回调你
- Netty（工业级方案）：**对 NIO 的封装**
  - 高性能（零拷贝、池化）
  - 支持高并发
  - 内置协议（HTTP、WebSocket）

## WebSocket?
**RESTful API = 一问一答（请求-响应）**  
**WebSocket = 持久连接，随时双向通信**

|维度|RESTful API|WebSocket|
|---|---|---|
|通信方式|请求-响应|双向通信|
|连接|短连接|长连接|
|服务端主动推送|❌ 不行|✅ 可以|
|实时性|较低|很高|
|复杂度|简单|较复杂|
|适用场景|CRUD业务|实时场景|
WebSocket一开始**走HTTP握手**，然后升级协议，建立一个长连接，一直不关。

什么时候用 WebSocket？ 》》 “需不需要服务端主动推？”
- WebSocket **在频繁通信场景更高效**
- 因为：
  - 少了反复建立连接
  - 头部开销小
  - 可以推送

适合 WebSocket：
- 聊天系统（IM）
- 实时通知（消息推送）
- 股票行情（实时数据）
- 游戏（实时交互）
- 在线协作（文档同步）

常见问题：
1. 连接数太多
- 一个连接占资源
- 需要限流
1. 负载均衡问题
- **粘性会话（sticky session）**
3. 鉴权问题
- URL 带 token
- 或握手阶段校验

❌ WebSocket **不走注册中心**  
✅ 它走的是 **“连接在哪台机器，就在哪台机器处理”**

### WebSocket 怎么实现
核心流程：
1. 客户端发起 HTTP 请求
2. 带上升级头：

```http
Connection: Upgrade  
Upgrade: websocket  
Sec-WebSocket-Key: xxx
```

3. 服务端返回：

```http
101 Switching Protocols
```

之后连接就从 HTTP 变成 WebSocket（长连接）

Spring Boot》》
方式一：原生注解（最常用）  @ServerEndpoint
- `Session` = 一个连接
- 用 `Map` 维护所有在线用户
- 可以主动推送消息
  方式二：Spring 封装（更规范） WebSocketHandler / WebSocketConfigurer

前端怎么连
```js
const ws = new WebSocket("ws://localhost:8080/ws/123");

ws.onopen = () => {
  console.log("连接成功");
};

ws.onmessage = (event) => {
  console.log("收到消息:", event.data);
};

ws.send("hello");
```

### 核心实现点：
1️⃣ 连接管理 userId → Session
2️⃣ 消息推送
- 单推（发给某个人）
- 广播（发给所有人）
  3️⃣ 心跳机制（非常关键）
- 定时 ping / pong
- 或客户端定时发消息
  4️⃣ 断线重连
- 前端监听 `onclose`
- 自动重连

微服务怎么搞
方案一：Redis + Pub/Sub
- 每台机器维护自己的连接
- 发消息 → 发布到 Redis
- 所有节点订阅 → 转发给本机用户
  方案二：MQ（Kafka / RabbitMQ）
  更稳定，适合大系统
```txt
“我不知道你在哪台机器，那我就让所有机器都知道”
1️⃣ 用户A连接到机器1
node1 记录：userA → Session（本地）
2️⃣ 现在系统要给 userA 发消息：广播mq
3️⃣ 所有 WebSocket 节点都订阅 MQ
4️⃣ 每个节点做判断：
if (本机有 userA 的连接) {  
发送消息  
}
```

方案三：专门网关
- WebSocket Gateway
- 或基于 Netty 自己实现
```
1️⃣ 连接分发（负载均衡）
客户端 → 网关 → 某一台 WebSocket 节点
 一旦连上：
- 后续通信不再经过网关逻辑判断
- 而是**直连那台机器**
2️⃣ Sticky Session（很关键）
 WebSocket 不能乱跳机器
  
```

MQ（广播消息）
↓
所有 WebSocket 节点收到
↓
**每个节点自己判断**：
“这个用户在不在我这？”
↓
在 → 发
不在 → 忽略

---
故事未完:79
**Thoughts**:: justdoit.
