# RPC理论

## 什么是RPC

remote procedure call：远程过程调用。

> 过程是什么？
>
> 过程就是业务处理、计算任务，更直白理解就是程序。(像调用本地方法一样调用远程的过程)

RPC采用Client-Server结构，通过request-response消息模式实现。

## RPC和RMI有什么区别

remote method invocation：远程方法调用时OOP领域中RPC的一种具体实现。

> 我们熟悉的webservice、restfull接口调用时RPC吗？
>
> 都是RPC，仅消息的组织方式及消息协议不同。

## 远程过程调用和本地调用有何不同

- 速度相对慢
- 可靠性减弱

## RPC的流程

1.客户端处理过程中**调用**Client stub(就像调用本地方法一样)，传递参数。

2.Client stub将参数**编组**为消息，然后通过系统调用向服务端发送消息。

3.客户端本地操作系统将消息从客户端机器**发送**到服务端机器。

4.服务端操作系统将接收到的数据包**传递**给Server stub。

5.Server stub**解组**消息为参数。

6.Server stub**再调用**服务端的过程，过程执行结果以反方向的相同步骤响应给客户端。

## RPC流程中需要处理的问题

1.Client stub、Server stub的开发。

2.参数如何编组为消息，以及解组消息。

3.消息如何发送。

4.过程结果如何表示、异常情况如何处理。

5.如何实现安全的访问控制。

## RPC协议是什么

​	RPC调用过程中需要将参数编组为消息进行发送，接收方需要解组消息为参数，过程处理结果同样需要经编组、解组。消息由那些部分构成及消息的表示形式就构成了消息协议。

​	RPC调用过程中采用的消息协议称为RPC协议。

> RPC协议规定请求、响应消息的格式
>
> 在TCP上可选用或自定义消息协议来完成RPC消息交互。
>
> 我们可以选用通用的标准歇息(如：http、https)，也可以根据自身的需要定义自己的消息协议。

## 常见的RPC协议

XML-RPC、JSON-RPC、SOAP

## RPC框架是什么

> 封装好参数编组、消息解组、底层网络通信的RPC程序开发框架，带来的便捷是可以直接在其基础上只需要专注于过程代码编写。

Java领域：

- 传统webservice框架：Apache CXF、Apache Axis2、java自带的JAX-WS等。大多基于SOAP协议。
- 新兴的微服务框架：Dubbo、spring cloud、Apache Thrift等。

## 为什么要用RPC

> 服务化
>
> 可重用
>
> 系统间交互调用

## RPC核心概念术语

- Client、Server、calls、replies、service、programs、procedures、version、marshalling(编组)、unmarshalling(解组)。
- 一个网络服务由一个或多个远程程序集构成。
- 一个远程程序实现一个或多个远程过程。
- 过程、过程的参数、结果在程序协议说明书中定义说明。
- 为兼容程序协议变更、一个服务端可能支持多个版本的远程程序。



# 手写RPC框架

## 从使用者角度开始

​	用户使用RPC框架开发过程中需要做什么？

​	1.定义过程接口

​	2.服务端实现过程

​	3.客户端使用生成的stub代理对象

## 客户端设计

​	客户端生成过程接口的代理对象

​	设计客户端代理工厂，用JDK动态代理即可生成接口的代理对象。

![](http://prvyof0n9.bkt.clouddn.com/rpc1.png)

## 设计客户端-思考

1.在ClientStubInvocationHandler中需要完成哪些事情？

- 编组消息
- 发送网络请求

2.将请求内容编组为消息这件事由谁来做？

- 消息协议
- 网络层

3.消息协议是固定不变的么？它与什么有关？

​	看框架对协议的支持广度，如果支持多种协议，就是会灵活变化的，**它与具体的服务相关**，A服务提供者可能选用的事协议1，B服务提供者可能选用协议2。

4.某服务是用的什么消息协议？信息从哪里来？

​	从获取的服务信息中来，因此需要一个**服务信息发现者**。

​	**把发现者设计出来，要求：可以灵活支持多种发现机制**。

## 设计客户端-发现者

![](http://prvyof0n9.bkt.clouddn.com/rpc2.png)

## 设计客户端-协议层

5.我们想要做到支持多种协议，我们的类该如何设计？

​	面向接口、策略模式、组合

![](http://prvyof0n9.bkt.clouddn.com/rpc3.png)

> 问题：
>
> marshalling和unmarshalling方法该定义怎样的参数与返回值？
>
> 编组、解组的操作对象是请求、响应，请求、响应的内容是不同的。编组、解组两个方法是否满足？

6.定义框架标准的请求、响应类

![](http://prvyof0n9.bkt.clouddn.com/rpc4.png)

7.将协议层方法扩展为四个

![](http://prvyof0n9.bkt.clouddn.com/rpc5.png)

​	**消息协议独立为一层**(客户端服务端均需要)

## 设计客户端-网络层

8.网络层的工作是什么？

​	发送请求，获得响应。要发起网络请求，则需要知道服务地址。

![](http://prvyof0n9.bkt.clouddn.com/rpc6.png)

## 设计客户端-客户端完整类图

![](http://prvyof0n9.bkt.clouddn.com/rpc7.png)

## 实现客户端

​	见github

## 服务端设计

​	客户端的请求过来，服务端首先需要通过RPCServer接收请求。

![](C:\Users\yuanyulou\AppData\Roaming\Typora\typora-user-images\1560250310286.png)

## 设计服务端-思考

1.RPCServer接收到客户端请求后，还需要做哪些工作？

![](http://prvyof0n9.bkt.clouddn.com/rpc9.png)

> 网络层在RPCServer中提供多线程来处理请求，消息协议层复用客户端设计的。
>
> (设计一个请求处理类，来完成网络层以上的事情。)

​	RCPServer接收到请求后，将请求交给RequestHandler来处理，RequestHandler调用协议层来解组请求消息为Request对象，然后调用过程。

![](http://prvyof0n9.bkt.clouddn.com/rpc10.png)

2.RequestHandler如何得到过程对象？

​	通过过程注册，和暴露。

3.Request中有什么？

​	服务名、方法名、参数类型、参数值

4.是否需要一个过程注册模块？

​	需要，维护一个服务名和服务对象的关系。

![](http://prvyof0n9.bkt.clouddn.com/rpc11.png)

- 过程注册模块：让用户将他们的过程注册到RPC框架中来。
- 过程暴露模块：想对外发布(暴露)服务注册、暴露可以由同一个类实现。

## 设计服务端-完整类图

![](http://prvyof0n9.bkt.clouddn.com/rpc12.png)

## 实现服务端

1.RPCServer中实现网络层：Netty，使用RequestHandler。

2.ServiceRegister模块实现服务注册、发布。

3.RequestHandler中实现消息协议处理、过程调用。
