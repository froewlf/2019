#### 1、CAP定律

* Consisitency：【强】一致性，事务保障，ACID模型
* Availiablity：【高】可用性，冗余以避免单点，至少做到柔性可用（服务降级）
* Parition tolerance：【高】可扩展（分区容忍性），一般要求系统能够自动按需扩展

#### 2、什么是微服务？

* 微服务的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底的去耦合，每一个微服务提供单个也业务功能服务。

#### 3、什么是springboot和springcloud？

* Springboot 是一个快速整合第三方框架，关注的是微观具体快速方便的开发单个个体的服务，是对spring应用的简化，使用特定的方式进行配置（properties或yml）配置，无须war包在tomcat下运行，直接运行jar包。
* SpringCloud 关注的是宏观  关注全局微服务协调整体治理框架，将Springboot开发的一个个单体服务整合并管理起来，为各个微服务之间提供，配置管理、服务发现、路由、熔断等集成服务。

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

#### 4.5、和ZooKeeper的对比

* zk是CP模型
  * Leader挂了，服务就会存在很长时间的不可用（此时在进行leader的重新选举）
* Eureka是AP模型
  * 只要有一台EurekaServer还在，就能保证注册服务是可用的。

#### 5、Ribbon

* Ribbon是Netflix发布的开源项目，主要功能是提供客户端软件的负载均衡算法，将Netflix的中间层服务连在一起。Ribbon客户端组件提供一些列完善的配置项如连接超时，重试等。

* Ribbon和nginx的区别
  * Ribbon是从eureka注册中心服务器端上获取服务注册信息列表，缓存到本地，然后再本地实现轮训负载均衡策略。属于客户端负载均衡。
  * ngixe是客户端所有请求统一交给nginx，有nginx进行实现负载均衡请求转发，属于服务器端负载均衡。

#### 6、Feign

* Feign，它是基于NetflixFeign实现，整合了SpringCloudRibbon与SpringCloudHystrix，除了提供这两者的强大功能之外，它还是一个声明式WebService客户端。

#### 7、hystrix

* hystrix是一个用于处理分布式系统的**延迟**和**容错**的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等。Hystrix能够保证在一个依赖出问题的情况下，**不会导致整体服务失效，避免级联故障，以提高分布式系统的弹性**。

* **服务熔断**
  * 服务熔断是应对服务雪崩效应的一种微服务链路保护机制
  * 当**扇出**链路的某个微服务不可用或者相应时间太长时，会进行服务的降级，**进而熔断该节点微服务调用，快速返回“错误”的响应信息**。当检测到该节点微服务调用响应正常后恢复调用链路。
  * Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，默认是5秒内20次调用失败就会启动熔断机制。熔断机制的注解是@HystrixCommand

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