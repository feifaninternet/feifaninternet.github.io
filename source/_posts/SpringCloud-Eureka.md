-------------------------
title: SpringCloud Eureka
tags: SpringCloud
categories: Spring
date: 2018-03-30 15:47:05
-------------------------

### Description
Eureka是Netflix开发的服务发现框架，springCould将它集成在自己的子项目spring-cloud-netflix中，实现springCould的服务发现功能。   
在一个完整的系统架构中，任何单点的服务都不能保证不会中断，因此我们需要服务发现机制，在某个节点中断后，其他的节点能够继续提供服务，从而保证整个系统是高可用的。这也是使用Eureka的原因。   
服务发现有两种模式: 一种是客户端发现模式，一种是服务端发现模式。Eureka采用的是客户端发现模式。

Eureka Server会提供服务注册服务，各个服务节点启动后，会在Eureka Server中进行注册，这样Eureka Server中就有了所有服务节点的信息，并且Eureka有监控页面，可以在页面中直观的看到所有注册的服务的情况。同时Eureka有心跳机制，当某个节点服务在规定时间内没有发送心跳信号时，Eureka会从服务注册表中把这个服务节点移除。Eureka还提供了客户端缓存的机制，即使所有的Eureka Server都挂掉，客户端仍可以利用缓存中的信息调用服务节点的服务。Eureka一般配合Ribbon进行使用，Ribbon提供了客户端负载均衡的功能，Ribbon利用从Eureka中读取到的服务信息，在调用服务节点提供的服务时，会合理的进行负载。   
Eureka通过心跳检测、健康检查、客户端缓存等机制，保证了系统具有高可用和灵活性。

### Eureka 程序构成
1. 是纯正的servlet应用，需构建成war包部署
2. 使用了Jersey框架实现自身的Restful HTTP接口
3. peer之间的同步与服务的注册全部通过 HTTP 协议实现
4. 定时任务(发送心跳/定时清理过期服务/节点同步等)通过JDK自带的Timer实现
5. 内存缓存使用Google的guava包实现

### 代码结构
eureka-core模块包含了功能的核心实现：
1. com.netflix.eureka.cluster -- 与peer节点复制(replication)相关的功能
2. com.netflix.eureka.lease -- 即"租约"，用来控制注册信息的生命周期(添加/清除/续约)
3. com.netflix.eureka.registry -- 存储，查询服务注册信息
4. com.netflix.eureka.resources -- Restful 风格中的"R",即资源。相当于SpringMVC中的Controller
5. com.netflix.eureka.transport -- 发送HTTP请求的客户端，如发送心跳
6. com.netflix.eureka.aws -- 与 amazon AWS 服务相关的类

eureka-client模块：   
Eureka客户端，微服务通过该客户端与Eureka进行通讯，屏蔽了通讯细节   

eureka-server模块：   
包含了servlet应用的基本配置，如web.xml。构建成功后在该模块下会生成可部署的war包。

eureka代码入口：   
由于是Servlet应用，所以Eureka需要通过servlet的相关监听器 ServletContextListener 嵌入到 Servlet 的生命周期中。EurekaBootStrap 类实现了该接口，在servlet标准的contextInitialized()方法中完成了初始化工作：

   ```
    @Override
        public void contextInitialized(ServletContextEvent event) {
            try {
                // 读取配置信息
                initEurekaEnvironment(); 
                // 初始化Eureka Client(用来与其它节点进行同步)
                // 初始化server
                initEurekaServerContext(); 
                ServletContext sc = event.getServletContext();
                sc.setAttribute(EurekaServerContext.class.getName(), serverContext);
            } catch (Throwable e) {
                logger.error("Cannot bootstrap eureka server :", e);
                throw new RuntimeException("Cannot bootstrap eureka server :", e);
            }
        }
   ```
   
Spring Cloud的EurekaServerBootstrap类没有实现servlet接口,在Spring容器初始化该组件时，Spring调用其生命周期方法start()从而触发了Eureka的启动。

### Eureka 和 Zookeeper
根据CAP理论，CAP不能同时满足，P(分区容错性)是分布式系统中必要保证的，因此我们只能在A和C之间进行权衡。Zookeeper保证的是CP，而Eureka保证的是AP。
#### Zookeeper 保证 CP
当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接down掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。但是zk会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30 ~ 120s, 且选举期间整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。

#### Eureka 保证 AP
Eureka看明白了这一点，因此在设计时就优先保证可用性。Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。Eureka的客户端在向某个Eureka注册或时如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：   
1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务 
2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用) 
3. 当网络稳定时，当前实例新的注册信息会被同步到其他节点中。

因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像Zookeeper那样使整个注册服务瘫痪。

### Eureka的一些概念
- Register：服务注册 

当Eureka客户端向Eureka Server注册时，它提供自身的元数据，比如IP地址、端口，运行状况指示符URL，主页等。

- Renew：服务续约

Eureka客户会每隔30秒发送一次心跳来续约。 通过续约来告知Eureka Server该Eureka客户仍然存在，没有出现问题。 正常情况下，如果Eureka Server在90秒没有收到Eureka客户的续约，它会将实例从其注册表中删除。 建议不要更改续约间隔。

- Fetch Registries：获取注册列表信息 

Eureka客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与Eureka客户端的缓存信息不同， Eureka客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，Eureka客户端则会重新获取整个注册表信息。 Eureka服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。Eureka客户端和Eureka 服务器可以使用JSON / XML格式进行通讯。在默认的情况下Eureka客户端使用压缩JSON格式来获取注册列表的信息。

- Cancel：服务下线 

Eureka客户端在程序关闭时向Eureka服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。

- Eviction 服务剔除 

在默认的情况下，当Eureka客户端连续90秒没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除，即服务剔除。