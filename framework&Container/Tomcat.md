# JavaEE面试问题总结

##### @Author LucI_PhAN

## 数据存储和消息队列

### Tomcat

**1. Tomcat的基础架构（Server、Service、Connector、Container）**  

Tomcat的顶层容器是Server代表整个服务器。一个Server可以包含至少一个Service，用于提供具体服务。  
Service包含两部分：Connectior：用于处理连接相关的事情，并提供Socket与Request和Response相关的转化。以及Container：用于封装和管理Servlet以及处理具体Request请求。

一个Tomcat中只有一个Server，一个Server可以包含多个Service，一个Service只有一个Container但可以存在多个Connectors，因为服务可以由多个连接（如同时提供Http和Https连接，也可以提供向相同协议不同端口的链接）。

多个Connector和一个Container形成了一个Service。Service对外提供服务，而整个Tomcat的生命周期由Server控制。我们开发中绝大部分进行配置的内容是属于Connector和Container的。  

请求流程：  
1. 请求发送到Tomcat之后，首先经过Service然后教给Connector
2. Connector接收请求并将接收的请求封装为Request和Response来具体处理，封装完成后再交由Container进行处理。
3. Container处理完请求之后再返回给Connector，由COnnector通过Socket将处理的结果返回给客户端。

Connector使用ProtocolHandler来处理请求的，不同的ProtocolHandler代表不同的连接类型。它包含了三个组件：  
--> Endpoint：用于处理底层Socket的网络连接，实现TCP/IP协议，它的抽象实现AbstractEndpoint里面定义了Acceptor和AsyncTimeout两个内部类和一个Handler接口。Acceptor用于监听请求，，AsyncTimeout用于检查异步Request的超时，Handler用于处理接收到的Socket，在内部调用Processor进行处理。  
--> Processor：用于将Endpoint接收到的Socket封装成Request，它实现Http协议。  
--> Adapter：用于将Request交给Container进行具体的处理。

Container内部包含四个子容器：  
--> Engine：引擎。用来管理多个站点。一个Service最多只能有一个Engine。  
--> Host：代表一个站点，虚拟主机，通过配置Host就可以添加站点。  
--> Context：代表一个应用程序，对应着平时开发的一套程序，或者一个WEB-INF目录以及下面的web.xml文件。Tomcat默认的配置下webapp下的每一个文件夹目录都是一个Context，ROOT目录存放主应用，其他目录存放子应用。而整个webapp就是一个Host站点。  
--> Wrapper：每一个Wrapper封装着一个Servlet

**2. Tomcat如何加载Servlet的**

请参见下面第三题最后部分（从FilterChain部分开始）。

**3. Pipeline-Valve机制**

Pipeline-value属于责任链模式。  
普通责任链模式指在一个请求处理的过程中有很多处理者依次对请求进行处理，每个处理者负责做自己相应的处理，处理完成之后将处理后的请求返回，再让下一个处理者继续处理。

但Pipeline-Value使用的责任链模式与普通责任链有不同。
1. 每个PipeLine都有特定的value。而且是在管道的最后一个执行。这个value叫做BaseValue，它是不可删除的。
2. 上层容器的管道的BaseValue中会调用下层容器的管道。

Connector接到请求后首先调用最顶层容器的Pipeline来处理（EnginePipeline）。在Engine的管道中依次执行EngineValue1，EngineValue2等等，最后执行StandardEngineValue，在这里会调用Host管道，再依次执行HostValue1，HostValue2，最后执行StandardHostValue，然后再依次调用Context和Wrapper的管道。

最后执行StandaradWrapperValue时，在其中创建FilterChain，并调用其doFilter方法来处理请求。这个FilterChain包含着我们配置的与请求相匹配的Filter和Servlet。其doFilter方法会依次调用所有的Filter的doFilter方法和Servlet的service方法，这样请求就得到了处理。这时将返回结果交还给Connector，Connector再通过Socket的方式将结果返回给客户端。
