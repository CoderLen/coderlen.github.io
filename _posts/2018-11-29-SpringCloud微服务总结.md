<h2>SpringCloud微服务框架</h2>

<h4>微服务是什么？</h4>

微服务提倡将单一应用划分成一组小的服务，服务之间相互协调，相互配合，为用户提供最终的价值。目前开源的微服务框架有Dubbo，Spring Cloud。



<h4>微服务的优势有哪些？</h4>

1. 降低复杂度
2. 可独立部署
3. 容错
4. 扩展



<h4>微服务的核心部件有哪些？</h4>

微服务通常由服务注册，服务发现，路由网关，负载均衡，服务降级，服务熔断，分布式配置几个方面组成。



<h4>Spring Cloud和Dubbo的对比</h4>

- 总体架构

  ![](https://upload-images.jianshu.io/upload_images/9741289-8ce96414a9884bbc?imageMogr2/auto-orient/strip%7CimageView2/2/w/675/format/webp)

  Provider: 暴露服务的提供方，可以通过Jar或者容器的方式启动服务

  Consumer：调用远程服务的服务消费方；

  Register：服务注册中心和发现中心

  Monitor：统计服务和调用次数，调用时间监控中心，Dubbo的控制台页面中可以展示。

  Container：服务运行的容器

  ![](https://upload-images.jianshu.io/upload_images/9741289-8eb3a17f8c14ed2c?imageMogr2/auto-orient/strip%7CimageView2/2/w/679/format/webp)



​	Service Provider：服务提供方

​	Service Consumer：调用远程服务的消费方

​	Eureka Server：服务注册中心和服务发现中心

- 架构核心要素

  ![](https://upload-images.jianshu.io/upload_images/9741289-71bef2f3e9b5ad97?imageMogr2/auto-orient/strip%7CimageView2/2/w/679/format/webp)

  Dubbo只是实现了服务治理，而Spring Cloud子项目分别覆盖了微服务架构下的众多部件，而服务治理只是其中的一个方面。Dubbo提供了各种Filter，对于上述中的"无"的要素，可以通过扩展Filter来完善。例如：

   	1. 分布式配置：可以使用淘宝的diamond，百度的disconf，携程的Apollo等；
   	2. 服务跟踪：可以使用京东开源的Hydra，或者扩展Filter用Zippin来做服务跟踪；
   	3. 批量任务：可以使用当当开源的Elastic-job

  从核心要素来看，Spring Cloud更胜一筹，在开发的过程中只要整合Spring Cloud的子项目就可以顺利完成各种组件的融合，而Dubbo却需要实现各种Filter来做定制，开发成本以及技术难度略高。

- 通讯协议

  Dubbo：Dubbo使用RPC通讯协议，支持多种序列化方式

  Spring Cloud： 使用HTTP协议的REST API

- 性能对比

- 服务依赖方式

- 组件运行流程

- 



<h4>参考资料</h4>

1. <a href="https://www.jianshu.com/p/02f9854a1717">终极对决！Dubbo 和 Spring Cloud 微服务架构到底孰优孰劣？</a>
2. 