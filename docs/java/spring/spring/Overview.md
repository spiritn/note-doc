[https://docs.spring.io/spring-framework/docs/current/reference/html/](https://docs.spring.io/spring-framework/docs/current/reference/html/)

# Spring Framework Documentation

|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [**Overview**](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview) | history, design philosophy, feedback, getting started.       |
| [**Core**](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core) | IoC Container, Events, Resources, i18n, Validation, Data Binding, Type Conversion, SpEL, AOP. |
| [Testing](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testing) | Mock Objects, TestContext Framework, Spring MVC Test, WebTestClient. |
| [Data Access](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#spring-data-tier) | Transactions, DAO Support, JDBC, R2DBC, O/R Mapping, XML Marshalling. |
| [Web Servlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#spring-web) | **Spring MVC**, WebSocket, SockJS, STOMP Messaging.          |
| [Web Reactive](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#spring-webflux) | Spring WebFlux, WebClient, WebSocket, RSocket.               |
| [Integration](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#spring-integration) | Remoting, JMS, JCA, JMX, Email, Tasks, Scheduling, Caching.  |
| [Languages](https://docs.spring.io/spring-framework/docs/current/reference/html/languages.html#languages) | Kotlin, Groovy, Dynamic Languages.                           |
| [Appendix](https://docs.spring.io/spring-framework/docs/current/reference/html/appendix.html#appendix) | Spring properties.                                           |
| [**Wiki**](https://github.com/spring-projects/spring-framework/wiki) | What’s New, Upgrade Notes, Supported Versions, and other cross-version information. |



> Spring makes it easy to create Java enterprise applications. It provides everything you need to embrace the Java language in an enterprise environment, with support for Groovy and Kotlin as alternative languages on the JVM, and with the flexibility to create many kinds of architectures depending on an application’s needs. As of Spring Framework 5.1, Spring requires JDK 8+ (Java SE 8+) and provides out-of-the-box support for JDK 11 LTS. Java SE 8 update 60 is suggested as the minimum patch release for Java 8, but it is generally recommended to use a recent patch release.
>
> Spring supports a wide range of application scenarios. In a large enterprise, applications often exist for a long time and have to run on a JDK and application server whose upgrade cycle is beyond developer control. Others may run as a single jar with the server embedded, possibly in a cloud environment. Yet others may be standalone applications (such as batch or integration workloads) that do not need a server.
>
> Spring is open source. It has a large and active community that provides continuous feedback based on a diverse range of real-world use cases. This has helped Spring to successfully evolve over a very long time.

spring让开发企业应用变得更简单。他提供了在企业环境中拥抱java语言所需的一切，支持Groovy和Kotlin作为JVM上的替代语言，并根据应用程序的需求灵活地创建多种架构。从Spring Framework 5.1开始，Spring需要JDK 8+（Java SE 8+），并提供对JDK 11 LTS的开箱支持。建议将Java SE 8更新60作为Java 8的最小补丁版本，但一般建议使用最近的补丁版本。

spring支持广泛的应用场景。在大型企业应用中，应用程序经常存在很长时间，而且必须运行在JDK和应用程序服务器上，其升级周期超出了开发人员的控制范围。其他的应用可能运行以嵌入服务器的形式，以单个jar运行，例如云服务器上。也有一些可能是独立的应用（如批处理或者集成工作负载），不需要服务器。

spring 是开源的。它拥有庞大而广泛的社区，根据各种不同的实际用例提供持续反馈。这帮助spring能在很长一段时间内保持成功。



# 1. What We Mean by "Spring"

'spring'在不同的语境下有不同的含义。它可以用来指spring框架项目本身，这是最开始的时候。随着时间的推移，其他的spring项目也建立在spring框架之上。大多数情况，当人们说“spring”时，意味着整个spring项目家族。本文档关注的是spring框架本身。

spring框架分为多个模块，应用程序可以选择自己需要的模块。最重要的是核心容器，包含配置模块和依赖注入。除此之外，spring框架还为不同的应用程序提供基础性的支持，包括消息传递，事务和持久化，以及web服务。也还包括基于servlet的spring mvc框架，以及Spring WebFlux的reactive web框架。



关于模块的说明：简单说就是jar支持JDK9。

# 2. History of Spring and the Spring Framework

spring诞生于2003年，是为了应对早期复杂的J2EE规范。虽然有些人认为java EE和spring是竞争关系，但事实上，spring是对java EE的补充。spring编程并不包含java EE平台规范；相反spring集成了java EE中的个别规范：

- Servlet API ([JSR 340](https://jcp.org/en/jsr/detail?id=340))
- WebSocket API ([JSR 356](https://www.jcp.org/en/jsr/detail?id=356))
- Concurrency Utilities ([JSR 236](https://www.jcp.org/en/jsr/detail?id=236))
- JSON Binding API ([JSR 367](https://jcp.org/en/jsr/detail?id=367))
- Bean Validation ([JSR 303](https://jcp.org/en/jsr/detail?id=303))
- JPA ([JSR 338](https://jcp.org/en/jsr/detail?id=338))
- JMS ([JSR 914](https://jcp.org/en/jsr/detail?id=914))
- as well as JTA/JCA setups for transaction coordination, if necessary.

spring框架还支持依赖注入 ([JSR 330](https://www.jcp.org/en/jsr/detail?id=330))和通用注解([JSR 250](https://jcp.org/en/jsr/detail?id=250)) 规范。开发者可以选择使用这些java EE规范来代替spring框架提供的。

从Spring Framework 5.0开始，Spring至少需要Java EE 7级别（如Servlet 3.1+、JPA 2.1+）--同时在运行时遇到Java EE 8级别（如Servlet 4.0、JSON Binding API）的较新API时，提供开箱即用的集成。这使得Spring与Tomcat 8和9、WebSphere 9和JBoss EAP 7等完全兼容。

随着时间的推移，Java EE在应用程序开发中的作用也在不断发展。在Java EE和Spring的早期，应用程序的创建是为了部署到应用服务器上。今天，在Spring Boot的帮助下，应用程序是以开发和云友好的方式创建的，嵌入了Servlet容器，改变起来也很简单。从Spring Framework 5开始，WebFlux应用甚至不直接使用Servlet API，可以在非Servlet容器的服务器（如Netty）上运行。

Spring不断创新，不断发展。在Spring框架之外，还有其他项目，如Spring Boot、Spring Security、Spring Data、Spring Cloud、Spring Batch等。需要记住的是，每个项目都有自己的源代码库、问题跟踪器和发布节奏。请参阅spring.io/projects了解Spring项目的完整列表。

# 3. Design Philosophy 设计哲学

当你了解一个框架时，不仅要知道他能做什么，还要知道它遵循什么原则。以下是spring的指导原则：

- 在每个层面提供选择。spring让你尽可能地推迟设计决策的时间。例如，你可以通过做配置来切换持久化依赖，而无需改变代码。对于其他基础框架和第三方的集成也是如此。
- 适应不同的观点。spring拥抱灵活性，不会去决定事情应该如何做。它以不同的视角提供广泛的应用需求
- 保持强大的向后兼容性。Spring的进化经过精心管理，迫使不同版本之间很少有中断性的变化。Spring支持一系列精心选择的JDK版本和第三方库，以方便维护依赖Spring的应用程序和库。
- 关心API设计。Spring团队花了大量的心思和时间来制作直观的API，并在许多版本和许多年内保持不变。
- 设置高质量的代码标准。spring强调有意义的、最新的和标准的Javadoc。它是极少数可以宣称代码结构干净，包之间没有循环依赖的项目之一。

# 4.Getting Started

如果您刚刚开始使用Spring，您可能希望通过Spring Boot来开始使用Spring框架。Spring Boot提供了一种快速（可选地）的方法来创建一个基于Spring的生产就绪的应用程序。它基于Spring框架，偏重于约定而非配置，旨在让您尽快启动并运行。

你可以使用start.spring.io来生成一个基本的项目，或者遵循 "["Getting Started" guides](https://spring.io/guides) "指南，比如 "[[Getting Started Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)](https://spring.io/guides/gs/rest-service/)"。除了更容易消化之外，这些指南还非常注重任务，而且大多数指南都是基于Spring Boot的。它们还涵盖了Spring组合中的其他项目，在解决某个问题时，你可能要考虑这些项目。