# JavaEE面试问题总结

##### @Author LucI_PhAN

## 数据存储和消息队列

### SSM/Servlet

**1. Servlet的生命周期**

Servlet的产生到消亡过程中，分为四个阶段，它们与三个生命周期函数有关：初始化方法init()，处理客户请求的方法service()，终止方法destroy()。

1.1 加载并实例化  
web容器负责加载Servlet。当web容器启动时或在第一次使用这个servlet时，容器会负责创建Servlet实例（这个选择默认为第一次访问时加载，但可以通过配置自动装载使其在启动时就自动载入）。但用户必须通过web.xml指定servlet的位置，也就是servlet所在的类名称。成功加载后，web容器会通过反射的方式对servlet进行实例化。

1.2 初始化 （init()方法执行且只执行一次）  
当srevlet被实例化后，servlet容器将调用每个servlet的init()方法来实例化每个实例。执行完init()方法后，servlet处于“已初始化”状态。也就是说一旦servlet被实例化，那么必将调用init()方法。

1.3 服务 （service()方法根据需求可能被执行多次）  
当执行service()方法时，servlet必定被初始化过。每个队servlet的请求由一个ServletRequest对象代表。servlet给客户端的响应由一个ServletResponse对象代表。对于到达客户机的请求，服务器创建特定于请求的一个请求对象和一个响应对象。service方法可以调用其他方法来处理请求。  
service()方法在服务器被访问时调用。servlet对象的生命周期中service方法可能被调用多次。由于web-server启动后，服务器中公开的部分自愿将处于网络中，当网络中不同客户端并发访问服务器中的统一资源，服务器将开设多个线程处理不同的请求。多线程同时处理同一对象时，可能会出现数据并发访问的错误。  

1.4 销毁 （destroy()方法执行且只执行一次）  
当服务器不再需要servlet实例或重新装入时，会调用destory()方法。使用这个方法，servlet可以释放掉所有在init()方法申请的资源。一个servlet实例一旦终止，就不允许再次被调用，只能等待卸载。  
servlet一旦终止，servlet实例即可被垃圾回收，处于卸载状态。而servlet容器被关闭也会使servlet被卸载。  
一个servlet实例只能初始化一次，但可以创建多个相同的servlet实例。如相同的servlet可以根据不同的配置参数连接不同的数据库时创建多个实例。

参考：  
[《Servlet运行原理以及生命周期》](https://www.cnblogs.com/fifiyong/p/6390805.html)

**2. 转发与重定向的区别**

从本质来说，转发是服务器行为，而重定向是客户端行为。

转发过程：  
客户浏览器发送http请求 -- web服务器接收此请求 -- 调用内部的方法在容器内部完成请求处理和转发动作 -- 将目标资源发送给客户。  
这里转发的路径限制为必须是同一个web容器下的url，不能转发到其他web路径上去。在客户浏览器路径栏中显示的仍然是其第一次访问的路径。转发行为是浏览器只做了一次访问请求，客户端是感觉不到服务器做了转发的。  

重定向过程：  
客户端发送http请求 -- web服务器接收后发送302状态码相应以及对应信封location给客户浏览器 -- 客户浏览器获得302响应码后，自动发送一个新的http请求，url是新的location地址 -- 服务器根据此请求寻找资源并发送给客户。  
这里重定向的location可以是任意的url。客户浏览器路径栏显示的是其重定向到的路径。重定向行为是浏览器做了至少两次的request请求。

通过本质分析，重定向是客户端完成的，会导致传输的信息被丢失，而请求转发可以被认为是对客户端透明的，客户不知道请求转发的过程，传输的信息也不会丢失。

参考：  
[《HTTP中的重定向和请求转发的区别》](https://blog.csdn.net/meiyalei/article/details/2129120)

**3. BeanFactory 和 ApplicationContext 有什么区别**

BeanFactory与ApplicationContext都是通过加载xml配置文件的方式加载bean。而后者是前者的扩展，ApplicationContext由BeanFactory派生而来，因此提供BeanFactory所有的功能。ApplicationContext以一种更面向框架的工作方式以及对上下文进行分层和实现继承。  
在绝大多数的“典型的”企业应用和系统，ApplicationContext都优先于BeanFactory。  

它们最大的不同就在于BeanFactory是延迟加载。如果一个bean当中存在属性没有加载，会在第一次调用getBean()方法的时候报错，而ApplicationContext会在读取xml文件后，如果配置文件没有错误，就会将所有的Bean加载到内存中。缺点就是在Bean较多的时候回占用很大的内存，程序启动较慢。但使得我们可以在容器启动时就发现Spring中存在的配置错误。

具体不同分析如下：  
1. 利用MessageSource的国际化  
BeanFactory没有扩展Spring中MessageResource接口。但ApplicationContext扩展了，因此具有消息处理的能力（i18N）。
2. 事件机制Event  
ApplicationContext的事件机制主要通过applicationEvent和ApplicationListener两个接口提供。当ApplicationContext中发布一个事件时，所有扩展了ApplicationListener的Bean都将接收到这个事件，从而进行相应的处理。
3. 底层资源的访问  
ApplicationContext扩展了ResourceLoader资源加载器接口。从而可以用来加载多个Resource。而BeanFactory是没有扩展ResourceLoader的。
4. 对web应用的支持  
与BeanFactory通常以编程的方式被创建不同的是，ApplicationContext能以声明的方式创建。

参考：  
[《Spring中ApplicationContext和beanfactory区别》](https://blog.csdn.net/hi_kevin/article/details/7325554)  
[《创建ApplicationContext与BeanFactory时的区别-Spring源码学习之容器的基本实现》](https://www.cnblogs.com/yuanmiemie/p/6812163.html)

**4. Spring Bean 的生命周期**

Spring框架中，一旦把一个Bean纳入Spring IOC容器之中，这个Bean的生命周期就会交由容器进行管理。一般当当管理角色的是BeanFactory或者ApplicationContext。  
以BeanFactory为例，说明Bean的生命周期：  
1. Bean的建立  
由BeanFactory读取Bean定义文件，并生成各个实例。
2. setter注入  
执行bean的属性依赖注入
3. BeanNameAware的setBeanName()  
如果实现了该接口，则执行此方法。
4. BeanFactoryAware的setBeanFactory()  
如果实现该接口，则执行此方法。
5. BeanPostProcessor的processBeforeInitialization()  
如果有关联的processor，则在Bean初始化之前都会执行这个实例的processBeforeInitialization()方法。
6. InitializingBean的afterPropertiesSet()  
如果实现了该接口，则执行此方法。
7. Bean定义文件中配置的init-method  
如果在bean中配置了自定义的init方法，则执行
8. BeanPostProcessors的processAfterInitialization()  
如果有关联的processor，则在Bean初始化之前都会执行这个实例的此方法。
9. DisposableBean的destroy()  
在容器关闭时，如果Bean实现了该接口，则执行此方法。
10. Bean定义文件中配置的destroy-method   
在容器关闭时，如果有在bean中配置自定义的destroy()方法，则会执行。

参考：  
[《Spring Bean的生命周期》](https://www.cnblogs.com/redcool/p/6397398.html)   
这里的流程可能不全，因为之前有手写过测试bean的流程代码，我记得是十几个步骤，之后如果能找到源码或者有时间重新测试，我会修改此题答案。

**5. Spring IOC 如何实现**

控制反转IOC inversion of control是一种设计思想。依赖注入DI dependency injection是实现IOC的一种方法。  
IOC是Spring的核心。所谓的控制反转，就是获得依赖对象的方式反转了。对于Spring来说，就是有Spring阿里负责空值对象的生命周期和对象间的关系。所有的类都在Spring容器中登记，告诉Spring自己是什么，自己需要什么。所有的类的创建，销毁都由Spring来控制。也就是说**控制对象生存周期的不再是引用它的对象，而是Spring**。对于某个具体的对象而言，以前是它控制其他对象，而现在是**所有对象都被Spring控制**，所以称为控制反转。

IOC地实现看起来高深莫测，实际上抽丝剥茧直达底层就会发现，其实IOC是建立在一些基础技术之上。IOC的实现建立在工厂模式，java反射机制和jdk的操作xml的DOM解析方式（当使用xml配置时）。甚至我们在某种程度上可以认为IOC就是一个存储了键值对的Map集合，里面存储的是beanId与实例的对应关系。  
首先Spring通过解析xml配置或者通过反射获取annotation注解来确立beanId与其对应的类的映射关系，然后通过工厂模式创建对应的实例。之后利用反射机制实行依赖注入。这就是IOC的灵魂。

参考：  
[《Spring如何实现IOC和AOP的，说出实现原理。》](https://www.cnblogs.com/dangjunhui/p/5473800.html)  
[《》spring ioc原理（看完后大家可以自己写一个spring）] (https://blog.csdn.net/a__yes/article/details/52201335)

**6. Spring中Bean的作用域，默认的是哪一个**

1. 单例Singleton（Spring默认作用域）  
Spring最初只有这个作用域。整个应用中只会创建一个实例，符合单例模式。
2. 原型Prototype  
每次调用都会创建一个新的实例。

在web使用的时候还有额外三个作用域。但必须在web.xml中注册一个RequestContextListener。目的是为了设置每次请求开始和结束都可以使Spring得到相应的事件。

3. 会话Session  
每个用户session可以产生一个新的bean，不同用户之间的bean互相不影响。
4. 请求Request  
为每个请求创建一个实例
5. globalSession  
作用和session雷士，只是使用portlet的时候使用。

参考：  
[《spring bean的四种常用作用域的测试》](http://gaddma.iteye.com/blog/2037038)

**7. 说说 Spring AOP、Spring AOP 实现原理**

**8. 动态代理（CGLib 与 JDK）、优缺点、性能对比、如何选择**

**9. Spring 事务实现方式、事务的传播机制、默认的事务类别**

**10. Spring 事务底层原理**
