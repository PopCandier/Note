### 	JAVA IO 演进之路



为Netty做准备。

**为什么要学Netty？**

分布式兴起后，变得横重要。

Spring5 底层用Netty

Spring Boot 不需要tomcat，因为内部实现了Web容器。

基于Netty



Zookeeper 也是用的Netty，协调中间件。

Dubbo 分布式服务框架，多协议支持 RPC



**Netty能帮我解决什么问题？**

Netty是封装IO操作的框架。

复杂的业务场景周年，没有说用一个单独的IO， API

就可以解决的，往往需要 IO+多线程来解决问题。



Mina类似的框架。被淘汰了。



**为什么要封装IO操作**

从java io的演讲过程

Input Output

选好参照物。



IO 模型，处理IO的一种方式

阻塞 ： acceet

非阻塞：主线程:轮询 selector



多路复用



阻塞和非阻塞 : IO 操作而言

同步和异步：时间而言



在处理数据的时候，

同一时间点做多个处理：异步

同一时间只能做一个操作： 同步



IO （BIO） Block IO 同步阻塞IO

NIO Non-Block IO 同步非阻塞IO(线程池)

AIO Async IO 异步非阻塞IO (事件驱动，回调)

目的：提升IO操作的性能。



但是操作比较复杂，所以IO框架孕育而生。



NIO

准备两个东西

轮询器 Selector

缓冲区 Buffer



应用场景

1. 1.4 以前都是BIO
2. 之后 NIO出现，IO性能得到大幅提升，对性能要求的
   Netty默认也是用NIO
3. 1.7 NIO2 （AIO） 支持回调，
   但是有一个毛病，因为回调是和操作系统直接调用
   操作系统的性能决定了IO的性能。
   加入线程实现异步

NIO的非阻塞，体现在，我可以不等待你处理完

，而是每个人都可以得到处理。只不过根据状态

会因为轮询，多处理几遍。 



#### Netty 与 IO发展

* 掌握NIO地核心组件 Buffer，Selector。Channel
* 何为多路复用
* Netty支持地功能和特性



Java NIO 都三套间



Buffer

底层就是byte[] 数组

position 当前数组地位置

limit 操作地范围值



flip方法调用地时候，表示要进行read操作，并且position

归零，表示下一次读取地时候，将会迭代到limit地位置。

get操作



read

flip

get

clear

0<=position<=limit<=capacity



slice() 分片

直接缓冲区，不copy到jvm内存中，而是直接操作操作系统地内容，

IO映射缓冲区，将不操作文件，而是直接改变文件在内存中

地位置。



**Selector**

Reactor模型，反应堆

给每个请求注册事件，可达不可达将会得到处理



**Channel**

操作对象地引用，真正传输地是buffer

矿车和矿道地比喻

channel 将会把请求和具体数据(文件等)建立关联关系



Netty就是一个同时支持多个协议地IO框架



服务治理

以前接口互相调用，之后会很复杂

现在必须通过注册中心，他将协调



分布式服务架构两大门派

* 阿里
  * Dubbo
  * Zookeeper
  * Nginx
* Spring 生态
  * Spring Cloud
  * Netflix Eureka
  * Spring Cloud Zuul



* 注册中心
  * Registry



![1560781859271](C:\Users\99405\AppData\Roaming\Typora\typora-user-images\1560781859271.png)

#### Netty高性能之路

Netty比传统IO性能提升了8倍

传统Rpc调用性差的三宗罪

* 阻塞而IO不阻塞弹性伸缩能力，高并发并导致宕机、
* Java序列化编码、解码的性能问题
* 传统IO模型过多占用CPU资源

IO模型、数据协议、线程模型



#### buffer的设计

零拷贝

内存池

* 池化 预先占位
* 非池化 用的时候再分配

高效的线程模型。Reactor

无锁化串行化pipeline

责任链，双向链表 inbound outbound

。。。

#### Netty核心组件之Channel启动

Netty服务端的启动入口

1. ServerBootstrap
2. 配置参数
3. bind()方法启动


### Reactor线程模型

单线程模型

多线程模型

主从线程模型



##### Netty消息推送

实时聊天通信：WebSocket协议来完成

Web容器：HTTP协议

IM聊天内容：发表情，送鲜花，发文字，自定义报文协议

RPC/Web容器/Socket/WebSocket、

终端：控制台、手机端、Web端

序列化:MsgPack,FastJson



#### 实战训练

对有需求的，对api熟悉，且有netty开发需求的人

#### Netty中的设计模式

**单例模式**

* 一个类在任何情况下只有一个对象，并提供一个访问点
* 



Netty 定位：

* 作为开源框架的底层框架（通信）TCP spring5
  * SpringBoot 内置的容器( Tomcat/Jerry)
  * Zookeper 数据交互
  * Dubbo 多协议 RPC的支持
* 直接做服务器(消息推送服务，游戏后台)



