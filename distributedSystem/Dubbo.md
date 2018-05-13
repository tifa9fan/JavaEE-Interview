# JavaEE面试问题总结

##### @Author LucI_PhAN

## 分布式

### 分布式其他

**1. 什么是Dubbo**

Dubbo是一个分布式服务框架，替代了webService中的wsdl，采用消费者与服务者在注册中心注册的方法来实现远程服务调用，是个远程呢个服务调用的分布式框架。  
Dubbo脱胎于阿里的电商系统。当网站流量很小的时候，ORM就足以满足需求。而当系统增长，第一步的扩容方法就是采用MVC来垂直架构项目。而当垂直应用越来越多，应用之间的交互将不可避免，我们将核心业务抽取出来作为独立的服务从而形成稳定的服务中心，这就是RPC分布式应用架构。而当服务进一步增长，我们就需要使用面向服务的SOA相关技术。  
而Dubbo提供高性能和透明化的RPC远程服务调用方案和SOA服务治理方案，他可以每天为两千多个服务提供大于30亿次的访问量支持，在国内得到了极其广泛的应用。

参考：  
[《Dubbo入门---搭建一个最简单的Demo框架》](https://blog.csdn.net/noaman_wgs/article/details/70214612)  
[《Dubbo是什么？能做什么？》](https://blog.csdn.net/houshaolin/article/details/76408399)

**2. 什么是RPC、如何实现RPC、RPC 的实现原理**

远程过程调用（Remote Procedure Call Protocol）是一种通过网络从远程计算机程序上请求服务器，而不需要了解底层网络技术的协议。  
它假定某些类似TCP和UDP的传输协议存在，为通信程序之间携带信息数据。他的本质还是底层的Socket通信。在OSI网络通信模型中，RPC跨越了传输层和应用层。它使得开发包括网络分布式多程序在内的应用程序更加容易。  

RPC采用客户机/服务器模式，请求程序是一个客户机，而服务提供程序就是一个服务器，首先，客户机调用进程发送一个有进程参数的调用信息到服务进程，然后等待应答信息。在服务器端，进程保持睡眠状态直到调用信息到达为止。当一个调用信息到达服务器获得进程参数，计算机国，发送答复信息，然后等待下一个调用信息。最后，客户端调用进程接收DAU信息，获得进程结果，然后调用执行继续进行。

RPC需要注意的是，服务的调用方和服务的提供方之间传输的数据需要进行序列化和反序列化操作，因为涉及到在网络上进行传输，任何类型的数据都需要转化为二进制，也就是序列化。序列化和反序列化的方式很多，可以使用Java本身内置的序列化方式，JSON，XML等。

参考：  
[《Dubbo入门---搭建一个最简单的Demo框架》](https://blog.csdn.net/noaman_wgs/article/details/70214612)   
[《基于TCP和HTTP协议的RPC简单实现》](https://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247483900&idx=1&sn=c5ca198a66a701f81c2ab118fe7a734a&chksm=e9c5f84ddeb2715bc574e467cd6537ef81f223453e0989ffd136976b48dcc2d961a75be596de&scene=21#wechat_redirect)

**3. Dubbo中的SPI是什么概念**

SPI即Service Provider Interface，是JDK内置的一种服务提供发现机制。它是一种动态替换发现的机制，为某个接口寻找服务，将装配的控制权转移到程序之外。  

Dubbo采用了微内核+插件体系，使得设计优雅扩展性也强大。我们定义了服务接口标准，让厂商去实现，JDK通过ServiceLoader类即可实现SPI机制的服务查找功能。

Dubbo使用的ExtensionLoader类似ServiceLoader类，其中含有一个静态属性：
ConcurrentMap, ExtensionLoader>EXTENSION_LOADERS = new ConcurrentHashMap, ExtensionLoader>();

用于缓存所有的扩展加载实例，这里加载Protocol.class，就以Protocol.class为key，创建的ExtensionLoader为value存储到上述EXTENSION_LOADERS中

这里没有进行任何的加载操作。

我们来看下，ExtensionLoader实例是如何来加载Protocol的实现类的：

1. 先解析Protocol上的Extension注解的name,存至String cachedDefaultName属性中，作为默认的实现

2. 到类路径下的加载 META-INF/services/com.alibaba.dubbo.rpc.Protocol文件

参考：  
[《跟我学Dubbo系列之Java SPI机制简介》](https://www.jianshu.com/p/46aa69643c97)  
[《dubbo之SPI解析》](https://blog.csdn.net/qiangcai/article/details/77750541)

**4. Dubbo的基本原理、执行流程**

Dubbo的整体架构如下：  
1. Provider：暴露服务的服务提供方  
服务提供者在启动时，需要向注册中心注册自己提供的服务。
2. Consumer：调用远程服务的服务消费方  
服务消费者在启动时，向注册中心订阅自己所需要的服务。
3. Registry：服务注册与发现的注册中心  
注册中心返回服务提供者列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
4. Monitor：统计服务的调用次数和调用时间的监控中心  
服务消费者和提供者在内存中累计调用次数和调用时间，将会定时发送一次统计数据到监控中心。
5. Container：服务运行容器  
负责启动，加载，运行服务提供者。

Dubbo采用全Spring配置方式，透明化接入应用，对应用没有任何API侵入，只需要Spring加载Dubbo的配置即可，Dubbo基于SPring的Schema扩展进行加载。

参考：  
[《Dubbo是什么？能做什么？》](https://blog.csdn.net/houshaolin/article/details/76408399)
