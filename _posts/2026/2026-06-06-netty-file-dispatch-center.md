---
title: 【项目】一步步做一个基于 Spring Boot + Netty 的文件分发系统
description: 主要用于小型文件分发场景
date: 2026-06-06
categories:
  - develop
tags:
  - develop
  - project
author: blackbzy
update_date: false
pin: false
toc: true
comments: true
---

> 项目实现demo为[file-dispatch-center-demo](https://github.com/blackbzy/file-dispatch-center-demo)，本文中仅仅提供最基础demo做演示，复杂的改动需要参考完整的项目demo。
{: .prompt-info }

本文主要是基于最小netty框架，通过问题推导到后面的系统，所以阅读本文主要是思路借鉴，技术部分如果有更好的选择可以酌情替换，感谢理解(⌐■_■)。

需求场景 ： 大规模文件实时分发，需要中心向边缘节点同步文件到多个客户端，内网隔离环境，不能直接开放HTTP下载。

## 一、为什么用 Netty

> 最终选择用 Netty 做文件分发，而不是 HTTP 上传下载，是因为需要长连接、高频小包、ACK 控制、断线状态管理。

Netty的特点，高频通信为什么适合 Netty？：
1. 长连接能力强
2. 高并发性能更好
3. 对 TCP 细节控制更强

为什么没有使用 FTP？
1. 缺少实时性
2. 缺少状态控制
3. 缺少可靠ACK
4. 不适合大规模长连接

为什么没有使用 WebSocket？
1. WebSocket 本质也是 TCP，但需求场景不需要浏览器，非浏览器通信
2. 直接 TCP 更简单

为什么没有使用 MQ（Kafka/RocketMQ）？
1. 不适合大文件传输

为什么没有使用 gRPC？
1. 协议封装太重
2. 文件流控制不够灵活

为什么没有使用http？
1. http是请求驱动
2. 报文格式固定
3. 不需要浏览器访问

基于以上的判断，对于小规模的文件分发的中台，netty是合适的选择。
## 二、基础搭建

netty主要分nettyClient端和nettyServer端。
nettyClient作为客户端，会向nettyServer端注册，然后接受Server端推送的数据。
nettyServer端作为文件分发中心，nettyClient作为下游接收文件解析和记录方。
本次的系统设计不涉及对文件的解析落库，仅保证数据的接收和分发的文件中台稳定。

NettyServer

```java
@Component
public class NettyServer {

    @PostConstruct
    public void start() {

        new Thread(() -> {

            EventLoopGroup boss = new NioEventLoopGroup(1);
            EventLoopGroup worker = new NioEventLoopGroup();

            try {

                ServerBootstrap bootstrap = new ServerBootstrap();

                bootstrap.group(boss, worker)
                        .channel(NioServerSocketChannel.class)
                        .childHandler(new ChannelInitializer<SocketChannel>() {

                            @Override
                            protected void initChannel(SocketChannel ch) {

                                ChannelPipeline pipeline = ch.pipeline();

                                pipeline.addLast(new StringDecoder());
                                pipeline.addLast(new StringEncoder());

                                pipeline.addLast(new ServerHandler());
                            }
                        });

                ChannelFuture future = bootstrap.bind(9000).sync();

                System.out.println("Netty Server Started");

                future.channel().closeFuture().sync();

            } catch (Exception e) {
                e.printStackTrace();
            }

        }).start();
    }

}

```

ServerHandler

```java
public class ServerHandler extends SimpleChannelInboundHandler<String> {

    @Override
    public void channelActive(ChannelHandlerContext ctx) {

        System.out.println("Client Connected: "
                + ctx.channel().remoteAddress());
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) {

        System.out.println("Receive: " + msg);

        if (msg.startsWith("CLIENT_ID:")) {

            String clientId = msg.replace("CLIENT_ID:", "");

            SpringUtil.getBean(ClientChannelManager.class)
                    .add(clientId, ctx.channel());

            ctx.writeAndFlush("REGISTER_SUCCESS");
        }
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) {

        System.out.println("Client Disconnect");
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,
                                Throwable cause) {

        cause.printStackTrace();
        ctx.close();
    }
}

```

NettyClient

```java
@Component
public class NettyClient {

    @PostConstruct
    public void connect() {

        new Thread(() -> {

            EventLoopGroup group = new NioEventLoopGroup();

            try {

                Bootstrap bootstrap = new Bootstrap();

                bootstrap.group(group)
                        .channel(NioSocketChannel.class)
                        .handler(new ChannelInitializer<SocketChannel>() {

                            @Override
                            protected void initChannel(SocketChannel ch) {

                                ChannelPipeline pipeline = ch.pipeline();

                                pipeline.addLast(new StringDecoder());
                                pipeline.addLast(new StringEncoder());

                                pipeline.addLast(new ClientHandler());
                            }
                        });

                ChannelFuture future =
                        bootstrap.connect("127.0.0.1", 9000).sync();

                future.channel().closeFuture().sync();

            } catch (Exception e) {
                e.printStackTrace();
            }

        }).start();
    }

}

```

ClientHandler

```java
public class ClientHandler extends SimpleChannelInboundHandler<String> {

    @Override
    public void channelActive(ChannelHandlerContext ctx) {

        System.out.println("Connected Server");

        ctx.writeAndFlush("CLIENT_ID:client-001");
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx,
                                String msg) {

        System.out.println("Receive From Server: " + msg);

        if ("FILE_TASK".equals(msg)) {

            System.out.println("Start Handle File Task");

            ctx.writeAndFlush("ACK");
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,
                                Throwable cause) {

        cause.printStackTrace();
        ctx.close();
    }
}

```



### Netty TCP 粘包/半包
Netty TCP 粘包/半包是在tcp协议使用过程中首要的问题，这个并不能完全解决，但是了解原因可以更好的避免半包场景。

TCP 为什么会粘包 ，会半包  ？
- 本质在于： TCP 是面向字节流（Byte Stream）的协议，**它不知道你的业务消息边界**。
- 粘包是因为 TCP 会优化传输
- 一条完整消息被拆开了，这叫半包（拆包）
  - TCP缓冲区大小限制
  - MTU限制
  - 接收端读取时机刚好在发送完成结束前
    实践过程中使用先发长度，再读取实际内容解决。


```md
+----------+
| length=5 |
+----------+
| Hello    |
+----------+

```

为什么高频发送更明显 ？
- 其实也就是频率高了，半包触发的更容易了

单向发送能不能减少半包?
- 不能，单向发送和半包基本没有直接关系。

### 文件消息协议设计：
既然消息的半包处理已经有思路了，那么对于文件传输的本体需要做出定义，方便上下游进行数据解析和校验。详细的编解码内容过多，就不在本文粘出，请参考[demo](https://github.com/blackbzy/file-dispatch-center-demo)

```java
 /** 序列化版本号，用于保证序列化兼容性 */
    private static final long serialVersionUID = 1L;

    /**
     * 认证令牌，用于客户端与服务端之间的身份验证。
     * <p>仅在 {@link MessageType#AUTH_REQUEST} 消息中使用。</p>
     */
    private String validateFlag;

    /**
     * 消息类型，标识当前消息的业务类别。
     * <p>取值参见 {@link MessageType} 枚举定义。</p>
     */
    private MessageType messageType;

    /**
     * 消息携带的二进制数据。
     * <p>在 {@link MessageType#FILE_CHUNK} 消息中表示文件分片的字节内容。</p>
     */
    private byte[] data;

    /**
     * 传输类型，标识当前数据的传输类别。
     * <p>取值参见 {@link TransferCategory} 枚举定义，如 FILE_DATA、CONTROL_COMMAND 等。</p>
     */
    private String transferType;

    /**
     * 传输结束标识。
     * <p>当消息类型为 {@link MessageType#END} 时，该值为 {@code true}。</p>
     */
    private boolean endFlag;

    /**
     * 分片参数对象，包含分片总数、当前分片序号、分片大小及偏移量。
     * <p>仅在文件分片传输时使用，与 {@link #chunkIndex}、{@link #totalChunks} 字段保持同步。</p>
     */
    private ChunkParams chunkParams;

    /**
     * 文件元数据对象，包含文件名、文件大小、文件类型、MD5校验值等属性。
     * <p>与 {@link #fileName}、{@link #fileLastModified} 字段保持同步。</p>
     */
    private FileMetadata fileMetadata;

    /**
     * 当前分片序号（从0开始）。
     * <p>为向后兼容保留的字段，与 {@link #chunkParams} 中的 chunkIndex 保持同步。</p>
     */
    private int chunkIndex;

    /**
     * 分片总数。
     * <p>为向后兼容保留的字段，与 {@link #chunkParams} 中的 totalChunks 保持同步。</p>
     */
    private int totalChunks;

    /**
     * 文件最后修改时间戳（毫秒）。
     * <p>为向后兼容保留的字段，与 {@link #fileMetadata} 中的 lastModified 保持同步。</p>
     */
    private long fileLastModified;

    /**
     * 文件名称（含扩展名）。
     * <p>为向后兼容保留的字段，与 {@link #fileMetadata} 中的 fileName 保持同步。</p>
     */
    private String fileName;

    /**
     * 确认应答标识。
     * <p>用于唯一标识一次确认应答，可选字段。</p>
     */
    private String ackId;

    /**
     * 操作成功标识。
     * <p>在 {@link MessageType#AUTH_RESPONSE} 中表示认证是否成功。</p>
     */
    private boolean success;

    /**
     * 错误信息描述。
     * <p>当 {@link #success} 为 {@code false} 时，包含具体的错误原因。</p>
     */
    private String errorMessage;

    /**
     * 消息签名值。
     * <p>基于HMAC-SHA256算法生成，用于验证消息内容的完整性和防篡改。</p>
     */
    private String signature;

    /**
     * 消息生成时间戳（Unix时间戳，毫秒）。
     * <p>用于签名验证中的时间戳校验，防止重放攻击。</p>
     */
    private long timestamp;

    /**
     * 消息类型枚举定义。
     */
    public enum MessageType {
        /** 认证请求消息 */
        AUTH_REQUEST,
        /** 认证响应消息 */
        AUTH_RESPONSE,
        /** 文件分片数据消息 */
        FILE_CHUNK,
        /** 确认应答消息 */
        ACK,
        /** 传输结束信号消息 */
        END,
        /** 错误通知消息 */
        ERROR,
        /** 心跳检测消息 */
        HEARTBEAT,
        /** 文件元数据消息 */
        FILE_METADATA
    }

    /**
     * 传输类别枚举定义。
     */
    public enum TransferCategory {
        /** 普通数据 */
        NORMAL_DATA("NORMAL_DATA"),
        /** 文件数据 */
        FILE_DATA("FILE_DATA"),
        /** 控制指令 */
        CONTROL_COMMAND("CONTROL_COMMAND"),
        /** 元数据 */
        METADATA("METADATA");

        /** 枚举对应的字符串值 */
        private final String value;

        /**
         * 构造方法。
         *
         * @param value 枚举对应的字符串值
         */
        TransferCategory(String value) {
            this.value = value;
        }

        /**
         * 获取枚举对应的字符串值。
         *
         * @return 字符串值
         */
        public String getValue() {
            return value;
        }
    }

```

## 三、文件分发核心
既然整体的框架搭建已经有了大概，上下游的基础已经建立，且可以进行数据分发，接下来就需要针对文件的传输场景进行特殊优化处理。

### Netty 文件分片
首先针对存在大文件的场景，和多个文件同时发送的场景进行优化，这里面牵扯到了文件的分片和对于文件发送完成的确认机制，这里的ack确认也可以和后文的文件重试关联。

同一个 channel 可以 writeAndFlush 多次吗？
- 可以：TCP 可能把它们合并成一次发送（粘包）， 也可能拆成多次接收（半包）。

上千文件并发安全吗？
- Netty 本身是线程安全的，但是安全 ≠ 高性能
- 可能触发内存压力

为什么大文件不能直接发送？
- 多个文件并发容易 OOM。
- GC 压力巨大。
- 大文件不是不能直接发，而是不能一次性全部加载到内存再发。企业级文件传输通常采用“分块传输”或“零拷贝传输”。

分片大小如何选择？
- 1MB基本是个比较合理的起点。后续再根据带宽、文件大小和并发量调优。
  - 内存占用可控
  - 进度统计方便
  - 网络利用率不错
  - 实现简单



提供以上问题解决，整个流程如下：

```md
文件
 ↓
切片
 ↓
发送队列     ←  组织发送，重试次数+1
 ↓                ↑
ACK确认->未确认，且重试次数未满，进入时间轮重试
 ↓
完成

```

### Netty 高并发发送与背压控制

背压方式
1. Channel不可写控制
  1. 暂停发送（最常用），恢复后继续发送。
  2.  限流
2. WriteBufferWaterMark**高低水位线**机制
  1. 高水位
  2. 低水位
  3. OOM风险
3. 出口节奏限制：即通过水位+并发线程数控制

以下为本项目[demo](https://github.com/blackbzy/file-dispatch-center-demo)未实现部分，但是是性价比较高的补充内容：
1. 线程池控制隔离
  1. IO线程与业务线程隔离
2. EventLoop 不能阻塞
  1. 把耗时操作丢到业务线程池。
3. DirectBuffer
  1. 不在 JVM 堆里分配，而是在堆外内存（off-heap）分配的内存块，少一次 JVM → Native 拷贝
  2. Netty zero-copy 配合 FileRegion
  3. 更高网络吞吐
4. flush 时机：把 write 进 Netty 缓冲区的数据真正推到 socket 发送出去
  1. 要在“延迟 vs 吞吐”之间做权衡
  2. 本项目因为是文件，是每次都直接flush返回


### Netty 重传、时间轮与可靠性机制
对于整个消息的传输稳定性已经得到了控制，接下来需要对文件进行冗余的可靠性增强：

1. ACK机制
  1. 维护一个ack队列，每个文件进行消息返回确认
2. 重试机制
  1. 维护一个时间轮队列，放入需要重试文件
  2. 每次重试检查是否已ack确认，是否到达最大重试次数

为什么选择时间轮？
1. 它适合“海量定时任务 + 延迟不精确但要高性能”的场景。且netty自带。
2. Thread.sleep / TimerTask 线程/任务太多，调度成本高
3. ScheduledThreadPool任务堆积时队列很大

## 四、工程化

### Netty 心跳、断线重连与连接管理
1. 心跳周期：常见配置 10s ~ 60s，通过 IdleStateHandler 实现，本质是“定时检测连接是否活着 + 超时断开”。
2. 客户端重连：定时轮询是否可用，不可用触发重试连接
3. Channel管理：用 ChannelGroup / Map 统一维护连接集合，方便广播、清理和状态控制，本质是“连接的生命周期管理”。

### Spring Boot + Netty + Redis 整体控制
1. Redis存储发送状态
2. 去重缓存
3. Redisson控制分布式锁
4. 当天桶设计控制当天文件不重复发送


整体文件分发流程

```md
服务器发送文件 → 发送 END 信号 → 注册 AckRecord → 等待客户端确认
                                             ↓
                              收到确认 → 移除记录 → 创建去重记录
                                             ↓
                              超时未确认 → 触发重试 → 超过重试次数 → 标记失败

```


## 五、可扩展部分
仅做实现扩展参考，本系统设计不涉及以下扩展部分

```md
### 1. 断点续传机制
- 当前实现：文件传输中断后需要从头开始
- 建议：记录已传输的分片信息，支持断点续传
### 2. 传输队列管理
- 当前实现：没有专门的队列管理
- 建议：实现优先级队列、队列长度限制、队列状态查询
### 3. 带宽控制
- 当前实现：仅配置了缓冲区大小
- 建议：支持传输速率限制、流量整形
### 4. 并发传输控制
- 当前实现：固定线程池大小（4线程）
- 建议：支持动态调整并发数、资源使用限制
### 5. 文件校验与修复
- 当前实现：仅MD5校验
- 建议：支持CRC校验、文件修复机制
### 6. 传输优先级
- 当前实现：所有文件同等优先级
- 建议：支持文件优先级标记、紧急文件优先传输
### 7. 监控与告警
- 当前实现：基础日志记录
- 建议：指标监控（成功率、延迟、吞吐量）、告警通知（邮件/短信/钉钉）
### 8. 客户端管理
- 当前实现：仅记录连接状态
- 建议：客户端白名单、连接限制、客户端状态管理
### 9. 集群与高可用
- 当前实现：单节点部署
- 建议：支持集群部署、负载均衡、故障转移
### 10. 权限控制
- 当前实现：简单token验证
- 建议：基于角色的访问控制（RBAC）、API密钥管理
### 11. 传输统计分析
- 当前实现：基础统计信息
- 建议：传输趋势分析、热点文件识别、性能报表
### 12. 多协议支持
- 当前实现：仅Netty TCP
- 建议：支持HTTP/2、WebSocket、FTP等协议
### 13. 事务性保证
- 当前实现：无事务保证
- 建议：支持分布式事务、传输原子性保证
### 14. 灰度发布与回滚
- 当前实现：无版本管理
- 建议：支持灰度发布、版本回滚
### 15. 运维管理API
- 当前实现：基础状态查询
- 建议：节点管理、配置热更新、远程诊断

```


---
故事未完:157
**Thoughts**:: justdoit.
