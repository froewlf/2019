#### 1、CAP定律

* Consisitency：【强】一致性，事务保障，ACID模型
* Availiablity：【高】可用性，冗余以避免单点，至少做到柔性可用（服务降级）
* Parition tolerance：【高】可扩展（分区容忍性），一般要求系统能够自动按需扩展

#### 2、什么是微服务？

* 微服务的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底的去耦合，每一个微服务提供单个也业务功能服务。

#### 3、什么是springboot和springcloud？

* Springboot 是一个快速整合第三方框架，关注的是微观具体快速方便的开发单个个体的服务，是对spring应用的简化，使用特定的方式进行配置（properties或yml）配置，无须war包在tomcat下运行，直接运行jar包。
* SpringCloud 关注的是宏观  关注全局微服务协调整体治理框架，将Springboot开发的一个个单体服务整合并管理起来，为各个微服务之间提供，配置管理、服务发现、路由、熔断等集成服务。
* SpringMVC 是一个基于spring的web开发框架。

#### 4、SpringCloudEureKa

##### 4.1、什么是Eureka？

* Eureka是Netflix的一个子模块，也是核心模块之一。Eureka是一个基于REST的服务，用与定位服务，以实现云端中间层服务发现和故障转移。服务注册与发现对于微服务架构来说是非常重要的，有了服务发现与注册，**只要使用服务的标识符，就可以访问到服务**，而不需要修改服务调用的配置文件。

##### 4.2、Eureka原理

* 系统中的其他服务，使用Eureka的客户端连接到Eureka Server并维持心跳连接。这样系统的维护人员就可以通过EurekaServer来监控系统中各个微服务是否正常运行。SpringCloud的一些其他模块（比如Zuul）就可以通过EurekaServer来发现系统中的其他微服务，并执行相关的逻辑。
* Eureka包含两个组件：Eureka Server 和Eureka Client
  * Eureka Server提供服务注册服务
  * 各个节点启动后，会在Eureka Server中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到。
  * EurekaClient是一个Java客户端，用于建辉EurekaServer的交互，客户端同时也具备一个内置的、使用轮询（round-robin）负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳（默认周期为30秒）。如果EurekaServer在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认为90秒）

##### 4.3、什么是Eureka的自我保护模式？

* 当EurekaServer节点在段时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式，一旦进入该模式，EurekaServer救护保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该EurekaServer节点会自动退出自我保护模式（它接收的心跳数重新恢复到阈值以上）。
* 可以使用eureka.server.enable-self-presevation = false禁用自我保护模式。

##### 4.4、集群配置

![img](https://img-blog.csdnimg.cn/20190529181808975.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29sZHNoYXVp,size_16,color_FFFFFF,t_70)

##### 4.5、和ZooKeeper的对比

* zk是CP模型
  * Leader挂了，服务就会存在很长时间的不可用（此时在进行leader的重新选举）
* Eureka是AP模型
  * 只要有一台EurekaServer还在，就能保证注册服务是可用的。

##### 4.6、EurekaServer是如何保证轻松抗住没秒数百次请求，每天千万级请求的呢？

* <https://blog.csdn.net/qq_42046105/article/details/83864668>
* 使用纯内存做注册表
* 注册表又做了多级缓存

#### 5、Ribbon

* Ribbon是Netflix发布的开源项目，主要功能是提供客户端软件的负载均衡算法，将Netflix的中间层服务连在一起。Ribbon客户端组件提供一些列完善的配置项如连接超时，重试等。

* Ribbon和nginx的区别
  * Ribbon是从eureka注册中心服务器端上获取服务注册信息列表，缓存到本地，然后再本地实现轮训负载均衡策略。属于客户端负载均衡。
  * ngixe是客户端所有请求统一交给nginx，有nginx进行实现负载均衡请求转发，属于服务器端负载均衡。

#### 6、Feign

* Feign，它是基于NetflixFeign实现，整合了SpringCloudRibbon与SpringCloudHystrix，除了提供这两者的强大功能之外，它还是一个声明式WebService客户端。
* Feign关键机制是使用了动态代理。

#### 7、hystrix

* hystrix是一个用于处理分布式系统的**延迟**和**容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等。Hystrix能够保证在一个依赖出问题的情况下，**不会导致整体服务失效，避免级联故障，以提高分布式系统的弹性**。

* **服务熔断**
  * 服务熔断是应对服务雪崩效应的一种微服务链路保护机制
  * 当**扇出**链路的某个微服务不可用或者相应时间太长时，会进行服务的降级，**进而熔断该节点微服务调用，快速返回“错误”的响应信息**。当检测到该节点微服务调用响应正常后恢复调用链路。
  * Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，默认是5秒内20次调用失败就会启动熔断机制。熔断机制的注解是@HystrixCommand

* 微服务架构如何保障双11狂欢下的99.99%高可用
  * <https://blog.csdn.net/qq_42046105/article/details/84000988>
  * 根据每秒的总请求数/该服务的部署台数，得到每台服务每秒会收到多少个请求
  * 然后根据每个请求的处理时间去计算hystrix线程池数量，然后确定熔断时间
  * 服务降级
    *  查询数据服务挂了，可以查本地缓存
    * 如果写入数据的服务挂了，可以先把这个写入操作记录日志到比如mysql里，或者写入MQ里，后面在慢慢恢复
    * 如果redis挂了，可以查mysql
    * 如果mysql挂了，可以吧操作日志记录到es里去，后面在慢慢恢复数据。

* 依赖隔离
  * Hystrix使用**舱壁模式**实现线程池的隔离，它会为每个依赖服务创建一个独立的线程池，这样就算某个依赖服务出现延迟过高的情况，也只是对该依赖服务的调用产生影响，而不会拖慢其他的依赖服务。
  * 优点
    * 应用自身得到完全保护，不会受不可控的依赖服务的影响。即便给依赖服务分配的线程池被填满，也不会影响应用自身的其余部分。
    * 可以有效降低接入新服务的风险。如果新服务接入后运行不稳定或存在问题，完全不会影响应用其他的请求。
    * 当依赖的服务从失效恢复正常后，它的线程池会被清理并且能够马上恢复健康的服务，相比之下，容器级别的清理恢复速度要慢的多
    * 当依赖的服务出现配置错误的时候，线程池也会快速反映出此问题（通过失败次数、延迟、超时、拒绝等指标的增加情况）。同时，我们可以在不影响应用功能的情况下通过实时的动态属性刷新来处理它
    * 当依赖的服务因实现机制调整等原因造成的其性能出现很大变化的时候，线程池的监控指标信息会反映出这样的变化。同时，我们也可以通过实现动态刷新自身应用对依赖服务的阈值进行调整以适应依赖方的变化
    * 除了上面通过线程池隔离服务发挥的优点之外，每个专有线程池都提供了内置的并发实现，可以利用它为同步的依赖服务构建异步访问。
  * Hystrix中除了可以使用线程池之外，还可以使用信号量来控制单个依赖服务的并发度，信号量的开销远比线程池的开销小，但是它不能设置超时和实现异步访问。所以，只有在依赖服务是足够可靠的情况下才能使用信号量。在HystrixCommand和HystrixObservableCommand中有两处支持信号量的使用。
    * 命令执行：如果将隔离策略参数execution.isolation.strategy 设置为SEMAPHORE，Hystrix会使用信号量代替线程池来控制依赖服务的发现。
    * 降级逻辑：当Hystrix尝试降级逻辑时，它会在调用线程中使用信号量。信号量默认值为10。

* 请求缓存
  * 在高并发的场景下，Hystrix中提供了请求缓存的功能，我们可以方便地开启和使用请求缓存来优化系统，达到减轻高并发的请求线程消耗、降低请求响应时间的效果。
  * 通过重载getCacheKey()方法开启请求缓存。
  * 开启请求缓存可以让我们实现的Hystrix命令具有下面几项好处
    * 减少重复的请求数，降低依赖服务的并发度
    * 在同一用户请求的上下文中，相同依赖服务的返回数据始终保持一致。
    * 请求缓存在run()和construct()执行之前生效，所以可以有效减少不必要的线程开销。
  * 清理失效缓存功能
    * 在使用请求缓存时，如果只是读操作，那么不需要考虑缓存内容是否正确的问题，但是如果请求命令中还有更新数据的写操作，那么缓存中的数据就需要我们在进行写操作时进行及时处理，以防止读操作的请求命令获取到失效的数据。
    * 通过HystrixRequestCache.clear()方法来进行缓冲的清理。
  * 使用注解方式增加缓存、移除缓存
    * 添加缓存
      * @CacheResult 标记在方法上
        * cacheKeyMethod 属性来指定具体的生成函数，来指定缓存key
      * @CacheKey 标记在参数上，指定缓存key
    * 移除缓存
    * @CacheRemove
      * commandKey属性是必须要指定的，它用来指明需要使用请求缓存的请求命令，因为只有通过该属性的配置，Hystrix才能找到正确的请求命令缓存位置。

* 请求合并

  * 微服务架构中的依赖通常通过远程调用实现，而远程调用中常见的问题就是通信消耗与连接数占用。在高并发的情况下，因通信次数的增加，总的通信时间消耗将会变得不那么理想。同时，因为依赖服务的线程池资源有限，将出现排队等待与响应延迟的情况。
  * Hystrix提供了HystrixCollapser来实现请求的合并，以减少通信消耗和线程数的占用。

* 4个级别属性配置

  * 全局默认值：如果没有设置下面三个级别的属性，那么这个属性就是默认值。由于该属性通过代码定义，所以对于这个级别，我们主要关注它在代码中定义的默认值即可。

  * 全局配置属性：通过在配置文件中定义全局属性值，在应用启动时或在与SpringCloudConfig 和SpringCloudBus实现的动态刷新配置功能配合下，可以实现对“全局默认值”的覆盖以及在运行期对“全局默认值”的动态调整。

  * 实例默认值：通过代码为实例定义的默认值。通过代码的方式为实例设置属性值来覆盖默认的全局配置。

  * 实例配置属性：通过配置文件来为指定的实例进行属性配置，以覆盖前面的三个默认值。可以使用config和bus实现的动态刷新配置功能实现对具体实例配置的动态调整。

  * execution配置

    * execution配置控制的是HystrixCommand.run()的执行。

    * execution.isolation.strategy：该属性用来设置HystrixCommand.run()执行的隔离策略，它有如下两个选择：

      * THREAD：通过线程池隔离的策略。它在独立的线程上执行，并且它的并发限制收线程池中线程数量的限制。

      * SEMAPHORE：通过信号量隔离的策略。它在调用线程上执行，并且它的并发限制受信号量计数的限制。

      * | 属性级别     | 默认值、配置方式、配置属性                                   |
        | ------------ | ------------------------------------------------------------ |
        | 全局默认值   | THREAD                                                       |
        | 全局配置属性 | hystrix.command.default.execution.isolation.stategy          |
        | 实例默认值   | 通过HystrixCommandProperties.Setter().withExecutionIsolationStrategy()(ExecutionIsolationStrategy.THREAD)设置，也可以通过@HystrixProperty（name="execution.isolation.strategy",value="THREAD"）注解设置。 |
        | 实例配置属性 | hystrix.command.HystrixCommandKey.exection.isolation.strategy |

    *  execution.isolation.thread.timeoutInMilliseconds：该属性用来配置HystrixCommand执行的超时时间，单位为毫秒。当HystrixCommand执行时间超过该配置值之后，Hystrix会将该执行命令标记为TIMEOUT并进入服务降级处理逻辑。

    * 

    * | 属性级别     | 默认值、配置方式、配置属性                                   |
      | ------------ | ------------------------------------------------------------ |
      | 全局默认值   | 1000毫秒                                                     |
      | 全局配置属性 | hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds |
      | 实例默认值   | 通过HystrixCommandProperties.Setter().withExecutionTimeoutInMillisecoends(int value)设置，也可以通过@HystrixProperty(name="executiom.isolation.thread.timeoutInMilliseconds", value = "2000")注解来设置 |
      | 实例配置属性 | hystrix.command.HystrixCommandKey.executiom.isolation.thread.timeoutInMilliseconds |

    * execution.timeout.enabled：该属性用来配置HystrixCommand.run()的执行是否启用超时时间。默认为true，如果设置为false，那么execution.isolation.thread,timeoutInMilliseconds属性的配置将不再起作用


#### 8、zuul

* Zuul包含了对请求的路由和过滤两个最主要的功能。
* 其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础
* 过滤功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础Zuul和Eureka整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也可以以后的访问微服务都是通过Zuul跳转后获得。
* 最终zuul还是会注册到Eureka服务中心
* 过滤器类型与请求生命周期
  * pre：这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。
  * routing：这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用Apache HTTPClient 或 Netflix Ribbon请求微服务。
  * post：这种过滤器在路由到微服务以后执行。这种过滤器可用来响应添加标准的HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端
  * error：在其他阶段发生错误时执行该过滤器。

#### 9、Feign和Ribbon的区别

* feign 封装了ribbon和hystrix
* feign自身是一个声明式的伪http客户端，写起来更加思路清晰和方便
* feign是一个采用基于接口的注解的编程方式，更加简单。

#### 10、SpringBoot注解方式加载配置文件
* @configuration ： 声明当前类做配置类使用
* @Bean ： 声明在方法上，将方法的返回值加入bean容器
* @value ： 属性值注入
* @PropertySource ： 指定外部待加载文件

#### 11、SpringBoot 启动流程

* SpringApplication模块初始化
* 实现具体的启动方案和加载上下文模块
* 加载自动化配置模块

* <https://www.cnblogs.com/trgl/p/7353782.html>

#### 12、微服务架构需要保证两点

* 首先hystrix资源隔离以及超时这块，必须设置合理的参数，避免高峰期，频繁的hystrix线程卡死
* 其次，针对个别的服务故障，要设置合理的降级策略，保证各个服务挂了，可以合理的降级，系统整体可用。

#### 13、RPC服务远程调用

* RPC的全称是Remote Procedure Call（远程过程调用）是一种进程间通信方式。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显示编码这个远程调用的细节。即程序员无论是调用本地还是远程，本质上编写的调用代码基本相同。

### 14、SpringMVC 执行流程

* DisPatcherServlet（前置控制器）拦截请求并解析url
* DisPatcherServlet调用HandlerMapping，HandleMapping调用HandleExecution根据url查找控制器，然后返回给DisPatcherServlet
* DisPatcherServlet再调用HandlerAdapter（处理器适配器），按照特定的规则去执行Handler（执行具体的Controller）。
* Controller执行后将具体的执行信息返回给HandlerAdapter，然后再返回给DispatcherServlet
* DisPatcherServlet调用视图解析器来解析HandlerAdapter传递的逻辑视图。
* DisPatcherServlet根据视图解析器解析的视图结果，调用具体的视图，最终视图呈现给用户。

### 15、