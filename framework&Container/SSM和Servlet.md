# JavaEE面试问题总结

##### @Author LucI_PhAN

## 开源框架及容器

### SSM/Servlet

**1. Servlet的生命周期**

Servlet的产生到消亡过程中，分为四个阶段，它们与三个生命周期函数有关：初始化方法init()，处理客户请求的方法service()，终止方法destroy()。

1.1 加载并实例化  
web容器负责加载Servlet。当web容器启动时或在第一次使用这个servlet时，容器会负责创建Servlet实例（这个选择默认为第一次访问时加载，但可以通过配置自动装载使其在启动时就自动载入）。但用户必须通过web.xml指定servlet的位置，也就是servlet所在的类名称。成功加载后，web容器会通过反射的方式对servlet进行实例化。

1.2 初始化 （init()方法执行且只执行一次）  
当srevlet被实例化后，servlet容器将调用每个servlet的init()方法来实例化每个实例。执行完init()方法后，servlet处于“已初始化”状态。也就是说一旦servlet被实例化，那么必将调用init()方法。

1.3 服务 （service()方法根据需求可能被执行多次）  
当执行service()方法时，servlet必定被初始化过。每个对servlet的请求由一个ServletRequest对象代表。servlet给客户端的响应由一个ServletResponse对象代表。对于到达客户机的请求，服务器创建特定于请求的一个请求对象和一个响应对象。service方法可以调用其他方法来处理请求。  
service()方法在服务器被访问时调用。servlet对象的生命周期中service方法可能被调用多次。由于web-server启动后，服务器中公开的部分资源将处于网络中，当网络中不同客户端并发访问服务器中的统一资源，服务器将开设多个线程处理不同的请求。多线程同时处理同一对象时，可能会出现数据并发访问的错误。  

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
IOC是Spring的核心。所谓的控制反转，就是获得依赖对象的方式反转了。对于Spring来说，就是有Spring来负责控制对象的生命周期和对象间的关系。所有的类都在Spring容器中登记，告诉Spring自己是什么，自己需要什么。所有的类的创建，销毁都由Spring来控制。也就是说**控制对象生存周期的不再是引用它的对象，而是Spring**。对于某个具体的对象而言，以前是它控制其他对象，而现在是**所有对象都被Spring控制**，所以称为控制反转。

IOC的实现看起来高深莫测，实际上抽丝剥茧直达底层就会发现，其实IOC是建立在一些基础技术之上。IOC的实现建立在工厂模式，java反射机制和jdk的操作xml的DOM解析方式（当使用xml配置时）。甚至我们在某种程度上可以认为IOC就是一个存储了键值对的Map集合，里面存储的是beanId与实例的对应关系。  
首先Spring通过解析xml配置或者通过反射获取annotation注解来确立beanId与其对应的类的映射关系，然后通过工厂模式创建对应的实例。之后利用反射机制实行依赖注入。这就是IOC的灵魂。

参考：  
[《Spring如何实现IOC和AOP的，说出实现原理。》](https://www.cnblogs.com/dangjunhui/p/5473800.html)  
[《spring ioc原理（看完后大家可以自己写一个spring）》](https://blog.csdn.net/a__yes/article/details/52201335)

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
作用和session类似，只是使用portlet的时候使用。

参考：  
[《spring bean的四种常用作用域的测试》](http://gaddma.iteye.com/blog/2037038)

**7. 说说 Spring AOP、Spring AOP 实现原理**

AOP面向切面编程 Aspect Oriented Programming 是通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生泛型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提供程序的可用性，同时提高了开发的效率。  
OOP引入了封装，继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。当需要为分散的对象引入公共行为的时候，OOP则显得无力。OOP允许定义从上到下的关系，但不并适合定义从左到右的关系。部分例如日志功能的代码往往水平范散在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。这种散布在各处的无关代码被称为横切（cross-cutting）代码。在OOP设计中，它导致了大量代码的重复，不利于各个模块的重用。而且我们无法通过抽象父类的方式消除重复性横切代码，因为这些横切逻辑依附在业务类方法的流程中，他们不能转移到其他地方去。  
对此，出现了AOP技术来解决难题。AOP利用一种称为横切的技术，解剖开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为切面 Aspect。AOP将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并利于未来的可操作性和可维护性。AOP的核心思想就是讲应用程序中的商业逻辑同对其提供支持的通用服务进行分离。  

AOP的实现主要分为两大类，可以采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行。也可以选择采用静态织入的方式，引入特定的语法创建切面，从而使得编译器可以在编译期间织入有关切面的代码。  
Spring采用了第二代AOP技术，采用动态代理机制和字节码生成技术实现。而第一代AOP例如AspectJ则是采用编译器将横切逻辑织入目标对象。  

Spring的AOP实现由两种方法，一是通过JDK自带的动态代理来完成代理类的动态创建。二是通过开源项目CGLIB来生成动态代理类。二者区别如下：  
1. 动态代理的机制实现由lang.reflect.Proxy类和lang,reflect.InvocationHandler接口完成，代理类与被代理类共同继承了相同的接口，他们彼此间是兄弟关系。而CGLIB扩展对象的原理是对目标对象进行了继承，为其生成相应的子类，而子类可以通过重写来扩展父类的行为。因此只需要将横切逻辑的实现放到子类中，然后让系统使用扩展后的目标对象的子类，就可以达到与代理模式相同的效果了。被代理类与代理类是父子关系。
2. JDK的动态代理只能对实现了相应Interface的类使用。如果这个类没有实现任何的Interface，就无法使用动态代理对其产生相应的代理对象。而相较于动态代理，由于采用了继承来实现，因此CGLIB可以为没有实现任何接口的类进行扩展。但请注意CGLIB也有限制，就是无法对final方法进行重写。
3. 在默认情况下，如果Spring AOP发现目标实现了相应的Interface，则采用JDK动态代理为其生成代理对象实例。而如果目标对象没有实现任何的Interface，Spring AOP会尝试使用CGLIB动态字节码生成库，为目标对象生成代理对象。

参考：  
[《探析Spring AOP（二）：Spring AOP的实现机制
》](https://blog.csdn.net/jeffleo/article/details/61205623)  
[《OOP的完美点缀—AOP之SpringAOP实现原理》](https://www.cnblogs.com/chenjunping/p/6664454.html)

**8. 动态代理（CGLib 与 JDK）、优缺点、性能对比、如何选择**

参看上题最后部分讲解

**9. Spring 事务实现方式、事务的传播机制、默认的事务类别**

Spring事务的事务实现分为两类四种：  
1. 编程式事务管理：  
需要手动编写代码，在实际开发中很少使用
2. 声明式事务管理   
2.1 基于TransactionProxyFactoryBean的方式，需要为每个进行事务管理的类做相应配置  
2.2 基于AspectJ的XML方式，不需要改动类，在XML文件中配置好即可  
2.3 基于注解的方式，配置简单，只需要在业务层类中添加注解。

Spring在TransactionDefinition接口中规定了七种类型的事务传播行为。它们规定了事务方法和事务方法发生嵌套调用时事务如何进行传播。它用于协调已经有事务标识的方法之间发生调用时的事务上下文的规制（是否要有独立的事隔离级别和锁）
1. PROPAGATION_REQUIRED  
如果没有当前事务，就新建一个事务。如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。
2. PROPAGATION_SUPPORTS  
支持当前事务，如果当前没有事务，就以非事务方式执行。
3. PROPAGATION_MANDATORY  
使用当前的事务，如果当前没有事务，就抛出异常。当业务方法被设置为PROPAGATION_MANDATORY时，它不能被非事务的业务方法调用。所以PROPAGATION_MANDATORY的方法一般都是被其他业务方法间接调用的。
4. PROPAGATION_REQUIRES_NEW  
新建事务，这是一个新的，与外层事务无关的内部事务，该事务拥有自己的独立隔离级别和锁，不依赖于外部事物，独立提交和回滚。如果当前存在事务，就把当前事务挂起。内部事务结束时，外部事务才继续执行。
5. PROPAGATION_NOT_SUPPORTED  
以非事务方式执行操作。当前如果存在事务，就把当前事务挂起。  
此传播机制使外层业务方法的事务被挂起，当内部方法执行完成后，外层方法的事务重新运行，如果外层方法没有事务，直接运行，不需要做任何其他事。
6. PROPAGATION_NEVER  
以非事务方式执行。如果当前存在事务，则抛出异常。当业务方法被设置为此传播机制时，它不能被拥有事务的其他业务方法调用。
7. PROPAGATION_NESTED  
如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。它创建的新事务是依赖于外层事务的子事务。当外层事务提交或回滚时，子事务也会连带提交和回滚。

注意：当使用PROPAGATION_NESTED时，底层的数据源必须基于JDBC 3.0，并且实现者需要支持保存点事务机制。

这里我不清楚题目要求的默认事务类型指的是什么。具体而言，事务传播级别默认为REQUIRED，事务隔离级别采用底层数据库的默认隔离级别（MySQL为可重复读，Orcale，SQLServer默认为提交读），事物的超时秒数默认为-1，永不超时。是否为只读事务默认为false，表示可读写事务。

参考：  
[《Spring配置事务中@Transactional各个属性定义》](https://blog.csdn.net/mawming/article/details/52277431)  
[《Spring五个事务隔离级别和七个事务传播行为》](https://yq.aliyun.com/articles/48893)  
[《Spring事务传播机制》](https://www.cnblogs.com/softidea/p/5962612.html)

**10. Spring 事务底层原理**

10.1 编程式事务管理底层原理   
TransactionTemplate是编程式事务管理的入口。它提供了唯一的编程入口execute，接收用于封装业务逻辑的TransactionCallback接口的实例，返回用户自定义的事务操作结果。  
先判断transactionManager是否是接口CallbackPreferringPlatformTransactionManager的实例。如果是则直接委托给该接口的execute方法进行事务管理。否则就教给核心成员PlatformTransactionManager进行实物的创建，提交或回滚操作。  
CallbackPreferringPlatformTransactionManager接口扩展自PlatformTransactionManager，该接口相当于是把事物的创建，提交和回滚都封装了。用户只需要传入TransactionCallback接口实例即可，而不是像使用PlatformTransactionManager接口那样，还需要用户自己显式调用getTransaction,rollback或commit进行事务管理。

10.2 声明式事务管理底层原理  
声明式事务管理的核心就是利用AOP技术，将食物逻辑作为环绕增强MethodInterceptor动态织入目标业务方法中。其中核心类为TransactionInterceptor。它实现了MethodInterceptor接口，将食物管理的逻辑封装在环绕增强的实现中，而业务代码则抽象为MethodInvocation（该接口扩展自Joinpoint，实际是AOP中的连接点），使得事务管理代码与业务逻辑代码完全分离，可以对任意目标类进行无侵入性的事务织入。  
先根据MethodInvocation获取事务属性TransactionAttribute，根据其得到对应的PlatformTransactionManager，再根据其是否是CallbackPreferringPlatformTransactionManager的实例分别作不同的处理。

参考：  
[《Java互联网架构-深度解读底层原理Spring事务管理分析》](https://baijiahao.baidu.com/s?id=1586726317037183459&wfr=spider&for=pc)

**11. Spring事务失效（事务嵌套），JDK动态代理给Spring事务埋下的坑**

事务失效的具体情境请操作参考资料。  

JDK动态代理中，只有代理对象proxy直接调用的方法才真正走的是代理。而如果这个方法内部调用了其他方法，调用的其他方法无论嵌套了多少层都是不会走代理的。  
如果没有注意到这个特点，在spring的事务嵌套中，如果传播方式为PROPAGATION_REQUIRES_NEW，而子事务抛出异常，父事务未捕获则两个都会失败，如果捕获则两个一起成功，这就与逻辑不符了。

解决方法：  
1. 通过AopProxy上下文暴露代理对象。配置XML为<aop:aspectj-autoproxy expose-proxy="true"/>
2. 通过ApplicationContext上下文进行解决

[《面试必备技能：JDK动态代理给Spring事务埋下的坑》](https://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247484940&idx=1&sn=0a0a7198e96f57d610d3421b19573002&chksm=e9c5ffbddeb276ab64ff3b3efde003193902c69acda797fdc04124f6c2a786255d58817b5a5c&scene=21#wechat_redirect)

**12. 如何自定义注解实现功能**

1. 使用@interface自定义一个注解，请注意定义注解时不能继承其他的注解或接口  
2. 使用@Target注解表示该注解可以用于什么地方
3. 使用Retention注解表示需要在什么级别来保存该注解信息。默认为class：即注解在class文件中可用，但会被JVM丢弃，可选source（注解将被编译器丢弃）和Runtime（JVM将在运行期间保留注解。如果需要通过反射获取注解，必须为此级别）
4. 使用@Inherited来表示允许子类继承父类中的注解。
5. 在注解中可以定义属性，可以通过default关键字来设置该属性默认值。
6. 在运行中注解只能通过反射获得，通过反射isAnnotationPresent()或getAnnotations()等方法来获取注解，之后可以依据注解是否存在和属性的值来做进一步自定义处理。

**13. Spring MVC 运行流程**

在整个SpringMVC框架中，DispatcherServlet处于核心位置，负责协调和组织不同组件以完成请求处理并返回响应的工作。

1. 若一个请求匹配DispatcherServlet的请求映射路径，web容器将该请求转交给DispatcherServlet处理
2. DispatcheServlet接收到请求后，将根据请求信息及HandlerMapping的配置找到处理请求的处理器（Handler）。
3. DispatcherServlet根据HandlerMapping得到对应当前请求的Handler后，通过HandlerAdapter对Handler进行封装，再以同一的适配器接口调用Handler。
4. 处理器完成业务逻辑的处理后将返回一个ModelAndView给DispatcherServlet，ModelAndView包含了视图逻辑名和模型数据信息。
5. DispatcherServlet借助ViewResoler完成逻辑视图名到真实试图对象的解析
6. 得到真实视图对象View后，DispatcherServlet使用这个View对ModelAndView中的模型数据进行视图渲染。

参考 ：  
[《springMVC运行流程分析》](https://blog.csdn.net/u013628152/article/details/51440271)

**14. Spring MVC 启动流程**

1. COntextLoaderListener初始化，实例化IoC容器，并将此容器实例注册到ServletContext中。
2. DispatcheServlet初始化，建立自己的上下文，也注册到ServletContext中。DIspatchServlet会建立自己的上下文来持有SpringMVC特殊的Bean度夏凝，在建立这个自己持有的IoC容器的时候，会从ServletContext中得到根上下文作为DispatcherServlet上下文的parent上下文。有了这个根上下文再对自己持有的上下文进行初始化，之后把自己持有的这个上下文保存到ServletContext中，供以后检索和使用。

参考：  
[《Spring MVC的启动过程》](https://www.cnblogs.com/mingziday/p/4987058.html)  

**15. Spring 的单例实现原理**

Spring的单例模式既不是饿汉也不是懒汉，而是采用了第三种，单例注册表的方式来实现的。注册表的缓存是HashMap对象。

Spring在需要获取一个bean实例时，首先去一个存储了beanName和实例映射关系的HashMap中以同步代码块方式获取此beanName对应的实例。  
如果实例为null，则判断此bean是否被规定为单例模式，如果是，则在同步方法块中再次检测注册表中不存在此bean实例后，创建bean实例并像注册表注册，以后将可以通过注册表直接获取该单例。  
如果bean规定为非单例，则每次都创建一个bean实例。

参考：  
[《Spring的单例实现原理》](https://blog.csdn.net/u011305680/article/details/79717238)

**16. Spring 框架中用到了哪些设计模式**

此题请参见basic/设计模式.md中的同名题目。

**17. Spring 其他产品（Srping Boot、Spring Cloud、Spring Secuirity、Spring Data、Spring AMQP 等）**

17.1 Spring Boot  
Spring Boot是由Pivotal团队提供的框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。

17.2 Spring Cloud  
Spring Cloud是Pivotal提供的用于简化分布式系统构建的工具集。Spring Cloud引入了云平台连接器（Cloud Connector）和服务连接器（Service Connector）的概念。云平台连接器是一个接口，需要由云平台提供者进行实现，以便库中其他模块可以与该平台协同工作。

17.3 Spring data  
Spring Data是一个用于简化数据库访问，并支持云服务的开源框架。它的目标是让数据库的访问变得方便快捷，并支持map-reduce框架和云计算数据服务。此外，它还支持基于关系型数据库的数据服务，如Oracle RAC等。

17.4 Spring Security  
Spring Security基于Spring框架，提供了一套web应用安全性的完整解决方案。对于用户认证Authentication和用户授权Authorization两部分Spring Security都有很好的支持。

17.5 Spring AMQP  
SPring AMQP项目将Spring核心思想应用于基于AMQP的消息解决方案的开发上。它提供了template这个高度抽象来发送和接收信息。它同样提供了消息驱动的实体。这些实体存在于listener container容器中。

参考：  
[《说一说Spring家族》](https://www.jianshu.com/p/b3e4aaa83a7d)

**18. 有没有用到Spring Boot，Spring Boot的认识、原理**

SpringBoot可以帮助管理依赖和自动配置。它把一个个技术模块封装成一个个starter，当引入该模块依赖的时候就可以开箱即用。它能够做到开箱即用的原理其实是Spring 4.x提供的基于条件配置bean的能力。  
@SpringBootApplication这个注解允许应用直接执行。它是一个组合注解，包含了@Configuration，@EnableAutoConfiguration，@ComponentScan三个注解。其中@EnableAutoConfiguration注解提供了核心的功能。它通过@Import注解导入配置。  
EnableAutoConfigurationImportSelector使用SpringFactoriesLoader.loadFactoryNames方法来扫描具有META-INF/spring.factories文件的jar包。spring-boot-autoconfigure-x.x.x.x.jar中有spring.factories文件，文件中声明了有哪些要自动配置。

参考：    
[《Spring Boot 中文索引》](http://springboot.fun/)

**19. MyBatis的原理**
