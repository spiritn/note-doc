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

Spring 能够轻松创建 Java 企业级应用程序。它提供了在企业环境中使用 Java 所需的一切，支持 Groovy 和 Kotlin 作为 JVM 平台上的替代语言，并能根据应用程序需求灵活构建多种架构。从 Spring Framework 6.0 开始，Spring 需要 Java 17 及以上版本。

Spring 支持广泛的应用场景。在大型企业中，应用程序通常需要长期运行，且必须在开发者无法控制的 JDK 和应用服务器升级周期内正常工作。有些应用程序则以内嵌服务器的方式打包为单个 JAR 文件运行，常见于云环境。还有一些是无需服务器的独立应用程序（例如批处理或集成任务）。

# 1. What We Mean by "Spring"
"Spring" 在不同语境中有不同含义。它可以指 Spring Framework 项目本身（这是一切的起源），也可以指随着时间的推移构建在 Spring Framework 之上的其他 Spring 项目。大多数情况下，当人们提到 "Spring" 时，他们指的是整个 Spring 系列项目。本文档主要关注其基础部分：Spring Framework 本身。
Spring Framework 采用模块化设计，应用程序可以按需选择模块。其核心是容器模块，包含配置模型和依赖注入机制。除此之外，Spring Framework 还为不同的应用架构提供基础支持，包括消息传递、事务数据与持久化以及 Web 开发。它包含基于 Servlet 的 Spring MVC Web 框架，同时也提供了并行的响应式 Web 框架 Spring WebFlux。

# 2. History of Spring and the Spring Framework

Spring 诞生于 2003 年，旨在解决早期 J2EE 规范的复杂性。虽然有人认为 Java EE 及其现代继任者 Jakarta EE 与 Spring 存在竞争关系，但实际上它们是互补的。Spring 编程模型并未完全采用 Jakarta EE 平台规范，而是有选择地集成了传统 EE 体系中的部分规范：
- Servlet API ([JSR 340](https://jcp.org/en/jsr/detail?id=340))
- WebSocket API ([JSR 356](https://www.jcp.org/en/jsr/detail?id=356))
- Concurrency Utilities ([JSR 236](https://www.jcp.org/en/jsr/detail?id=236))
- JSON Binding API ([JSR 367](https://jcp.org/en/jsr/detail?id=367))
- Bean Validation ([JSR 303](https://jcp.org/en/jsr/detail?id=303))
- JPA ([JSR 338](https://jcp.org/en/jsr/detail?id=338))
- JMS ([JSR 914](https://jcp.org/en/jsr/detail?id=914))
- as well as JTA/JCA setups for transaction coordination, if necessary.
- Spring Framework 还支持依赖注入（JSR 330）和通用注解（JSR 250）规范，开发者可以选择使用这些规范替代 Spring Framework 提供的专属机制。这些规范最初基于通用的 javax 包。

从 Spring Framework 6.0 开始，Spring 已升级至 Jakarta EE 9 级别（例如 Servlet 5.0+、JPA 3.0+），基于 jakarta 命名空间而非传统的 javax 包。以 EE 9 为最低要求并已支持 EE 10，Spring 已准备好为 Jakarta EE API 的持续演进提供开箱即用的支持。Spring Framework 6.0 与 Tomcat 10.1、Jetty 11 等 Web 服务器以及 Hibernate ORM 6.1 完全兼容。
随着时间的推移，Java/Jakarta EE 在应用程序开发中的角色也在演变。在 J2EE 和 Spring 的早期阶段，应用程序需要部署到应用服务器中。如今，借助 Spring Boot，应用程序能够以符合 DevOps 和云原生理念的方式创建，内嵌 Servlet 容器且易于更换。从 Spring Framework 5 开始，WebFlux 应用程序甚至不直接使用 Servlet API，可以在非 Servlet 容器服务器（如 Netty）上运行

Spring 持续创新与发展。除了 Spring Framework，还有 Spring Boot、Spring Security、Spring Data、Spring Cloud、Spring Batch 等项目。需要注意的是，每个项目都有独立的源代码仓库、问题跟踪系统和发布节奏。完整的 Spring 项目列表可访问 spring.io/projects 查看。

# 3. Design Philosophy 设计哲学

学习一个框架时，不仅要了解它能做什么，还要理解其遵循的原则。以下是 Spring Framework 的核心指导原则：

- 提供多层次选择：Spring 让你尽可能推迟设计决策。例如，通过配置即可切换持久化提供者而无需修改代码，许多其他基础设施关注点与第三方 API 集成也是如此。
- 兼容多元视角：Spring 拥抱灵活性，对实现方式不设限，以多元视角支持广泛的应用需求。
- 保持强向后兼容性：Spring 的演进经过审慎管理，确保版本间破坏性变更最小。它精心选择支持的 JDK 版本和第三方库范围，以简化依赖 Spring 的应用程序和库的维护工作。
- 重视 API 设计：Spring 团队投入大量精力设计直观且能经受多版本、多年考验的 API。
- 坚持高标准代码质量：Spring Framework 高度重视编写有意义、与时俱进且准确的 javadoc，是少数能实现清晰代码结构且包间无循环依赖的项目之一。

--------------------

```text
Spring & Spring Boot 核心知识体系
│
├── **1. IoC容器 (控制反转与依赖注入) - Spring基石**
│   ├── 核心概念：BeanDefinition, ApplicationContext, BeanFactory
│   ├── Bean生命周期管理 BeanPostProcess
│   ├── **★ 循环依赖及其解决机制（三级缓存）**
│   ├── FactoryBean机制
│   ├── 条件装配 (@Conditional 及其派生注解)
│   ├── 环境与配置抽象：Environment, @Profile
│   ├── 资源访问：Resource, ResourceLoader
│
├── **2. AOP面向切面编程**
│   ├── 核心概念：切面(Aspect)、连接点(JoinPoint)、通知(Advice)、切点(Pointcut)
│   ├── 代理机制：**JDK动态代理 vs CGLIB代理**
│   ├── 通知类型：@Before, @After, @Around, @AfterReturning, @AfterThrowing
│
├── **3. 数据访问与事务管理**
│   ├── **声明式事务管理**：
│   │   ├── @Transactional的传播行为和隔离级别，失效场景！
│   └── 数据校验：Spring Validation, JSR 303/349
│
├── **4. Spring MVC 与 Web开发**
│   ├── 核心架构：DispatcherServlet, 处理器映射(HandlerMapping), 适配器(HandlerAdapter)
│   ├── 常用注解：@Controller, @RestController, @RequestMapping, @GetMapping等
│   ├── 参数绑定：@RequestParam, @PathVariable, @RequestBody, @ModelAttribute
│   ├── **★ 统一异常处理 (@ControllerAdvice, @ExceptionHandler)**
│   ├── **★ 跨域处理 (CORS)**
│   ├── 拦截器 (HandlerInterceptor)和过滤器 (Filter)
│
├── **5. Spring Boot 核心特性与自动配置**
│   ├── **核心理念**：约定优于配置、快速启动、独立运行
│   ├── **启动类与注解**：@SpringBootApplication (三大注解合成)
│   ├── **自动配置原理**：
│   │   ├── @EnableAutoConfiguration
│   │   ├── spring.factories 机制 (Spring Boot 2.x) / META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports (Spring Boot 3.x)
│   │   └── **★ 条件装配的深度应用** (如 @ConditionalOnClass, @ConditionalOnMissingBean)
│   ├── **★ 外部化配置与加载顺序**
│   │   ├── 配置文件：application.properties/yml
│   │   ├── 多环境配置：application-{profile}.yml
│   │   └── 加载优先级：命令行参数 > Java系统属性 > 配置文件...
│   ├── **Starter机制**：如何自定义Starter
│   ├── **内嵌Web容器**：Tomcat, Jetty, Undertow 及其配置
│
├── **6. 高级特性与扩展**
│   ├── **事件驱动模型**：ApplicationEvent, ApplicationListener, @EventListener
│   ├── **异步与调度**：@Async (异步执行), @Scheduled (定时任务)
│   ├── **扩展点**：
│   │   ├── BeanFactoryPostProcessor (修改Bean定义)
│   │   ├── BeanPostProcessor (修改Bean实例)
│   │   ├── ApplicationContextInitializer (上下文初始化)
│   │   └── **★ ImportSelector, ImportBeanDefinitionRegistrar**
│   └── **★ 配置元信息 (@ConfigurationProperties绑定原理)**
```

# Spring 体系知识图谱

## 1. IoC 容器核心机制

### 1.1 BeanDefinition 体系与解析机制

Spring IoC 容器的核心在于通过 BeanDefinition 管理 Bean 的元数据信息。BeanDefinition 是一个接口，位于 org.springframework.beans.factory.config 包中，它定义了 Bean 的完整配置信息，包括类名、作用域、构造函数参数、属性值、依赖关系等[(32)](https://juejin.cn/post/7473299165290381350)。

在 Spring 容器中，BeanDefinition 的解析是整个容器启动流程的关键环节。Spring 会根据配置文件（XML、注解、Java 配置）加载 Bean 的定义信息，将每个 Bean 的元数据存储在 BeanDefinition 中[(36)](https://blog.csdn.net/weixin_66592566/article/details/144920994)。不同类型的配置源对应不同的解析器：对于 XML 配置，使用 XmlBeanDefinitionReader 配合 BeanDefinitionParser 解析标签；对于注解配置，使用 AnnotationConfigApplicationContext 进行类路径扫描和注解解析[(33)](https://blog.csdn.net/qq_45000512/article/details/149825750)。

BeanDefinition 的核心属性包括：class 属性指定 Bean 的实现类；scope 属性定义 Bean 的作用域（如 singleton、prototype 等）；constructorArgumentValues 存储构造函数参数；propertyValues 保存属性值；dependsOn 指定依赖的 Bean；autowireMode 定义自动装配模式；lazyInit 控制是否延迟初始化。这些属性共同构成了 Bean 的完整配置信息，容器基于这些信息创建和管理 Bean 实例。

### 1.2 ApplicationContext 与 BeanFactory 关系

BeanFactory 是 Spring IoC 容器的最基础接口，它定义了容器的基本功能规范，提供了实例化对象和获取对象的基础能力[(2)](https://blog.csdn.net/mrluo735/article/details/135665427)。BeanFactory 有三个重要的子接口：ListableBeanFactory 提供了按类型列举 Bean 的功能；HierarchicalBeanFactory 支持父子容器的层级结构；AutowireCapableBeanFactory 提供了更高级的自动装配功能[(2)](https://blog.csdn.net/mrluo735/article/details/135665427)。

ApplicationContext 是 BeanFactory 的子接口，它在 BeanFactory 基础上添加了更多企业级功能[(1)](https://docs.spring.io/spring-framework/docs/5.3.18/reference/pdf/core.pdf)。ApplicationContext 提供的主要增强功能包括：与 Spring AOP 功能的无缝集成；支持国际化的消息资源处理；事件发布机制；Web 应用专用的 WebApplicationContext 等[(1)](https://docs.spring.io/spring-framework/docs/5.3.18/reference/pdf/core.pdf)。可以说，ApplicationContext 是 BeanFactory 的超集，在大多数应用场景中，开发者都应该优先使用 ApplicationContext 而非原始的 BeanFactory[(1)](https://docs.spring.io/spring-framework/docs/5.3.18/reference/pdf/core.pdf)。

在实际应用中，BeanFactory 通常用于需要完全控制 Bean 处理过程的特殊场景，而 ApplicationContext 则提供了更便捷的开发体验和丰富的企业级功能。两者的主要区别在于功能丰富程度和应用场景：BeanFactory 更轻量级，启动速度快，适合资源受限的环境；ApplicationContext 功能更全面，支持更多企业级特性，是 Spring 应用的首选容器[(1)](https://docs.spring.io/spring-framework/docs/5.3.18/reference/pdf/core.pdf)。

### 1.3 Bean 生命周期管理与扩展点

Spring Bean 的生命周期是一个从创建到销毁的完整流程，涉及实例化、依赖注入、初始化、代理生成、销毁等多个阶段[(79)](https://blog.csdn.net/m0_73077921/article/details/153744858)。理解 Bean 的生命周期对于正确使用 Spring 框架和实现自定义扩展至关重要。

Bean 的完整生命周期可以分为以下几个关键阶段：首先是实例化阶段，Spring 根据配置找到 Bean 对应的 Class 对象，通过构造函数创建 Bean 的实例；然后是属性注入阶段，通过 postProcessProperties 方法处理 @Autowired、@Resource 等标注的依赖；接下来是初始化阶段，执行 Aware 接口方法、BeanPostProcessor 前置处理、@PostConstruct 标注的方法和 init-method 配置的初始化方法；最后是销毁阶段，执行 @PreDestroy 标注的方法和 dispose-method 配置的销毁方法[(11)](https://blog.csdn.net/weixin_42148384/article/details/148770132)。

BeanPostProcessor 是 Spring 提供的最重要的生命周期扩展点之一。容器中所有 BeanPostProcessor 会被优先初始化（比普通 Bean 早），当创建一个普通 Bean 时，容器会先执行所有 BeanPostProcessor 的 postProcessBeforeInitialization 方法，然后执行 Bean 的初始化方法，最后执行所有 BeanPostProcessor 的 postProcessAfterInitialization 方法[(78)](http://m.toutiao.com/group/7592874726532383267/?upstream_biz=doubao)。这种机制允许开发者在 Bean 初始化前后插入自定义逻辑，是实现 AOP、依赖注入等核心功能的基础。

### 1.4 循环依赖解决方案（三级缓存机制）

循环依赖是指两个或多个 Bean 之间相互依赖，形成一个闭环的情况。Spring 通过三级缓存机制巧妙地解决了单例模式下的 Setter 循环依赖问题[(18)](https://blog.csdn.net/2403_88625492/article/details/153690120)。

三级缓存的核心设计包括：一级缓存 singletonObjects，存储完全初始化好的单例 Bean；二级缓存 earlySingletonObjects，存储提前暴露的原始 Bean（尚未完成初始化）；三级缓存 singletonFactories，存储 Bean 的 ObjectFactory，用于生成原始 Bean 的代理对象[(21)](https://blog.csdn.net/weixin_43521001/article/details/148685461)。

解决循环依赖的具体流程如下：当创建 BeanA 时，首先实例化 BeanA，然后将其 ObjectFactory 放入三级缓存 singletonFactories；在填充 BeanA 的属性时发现需要 BeanB，于是开始创建 BeanB；同样地，实例化 BeanB 后将其 ObjectFactory 放入三级缓存；在填充 BeanB 的属性时发现需要 BeanA，此时从三级缓存中获取 BeanA 的 ObjectFactory，调用 getObject () 方法获取 BeanA 的早期引用（可能是代理对象），并将其放入二级缓存 earlySingletonObjects；完成 BeanB 的创建并放入一级缓存后，继续完成 BeanA 的初始化过程，最终将 BeanA 从二级缓存移到一级缓存[(20)](https://blog.csdn.net/weixin_43598533/article/details/147937146)。

三级缓存机制的关键在于延迟代理对象的创建，确保在循环依赖场景下，Bean 能够获取到正确的代理对象引用。这种设计既保证了循环依赖的正确处理，又维护了 Spring 容器中 Bean 的单例特性[(19)](https://www.iesdouyin.com/share/video/7590624683045719348/?region=\&mid=7590624816974072618\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=EhXHQ71xHTfbhom9lBQaTXQImPAvVGDhGyNkYg9ybAg-\&share_version=280700\&ts=1769260986\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)。

### 1.5 FactoryBean 机制与应用场景

FactoryBean 是 Spring 框架提供的一种特殊 Bean 机制，它允许开发者自定义 Bean 的创建逻辑，而不是使用容器的默认反射机制[(39)](https://blog.csdn.net/Like0703/article/details/145719948)。FactoryBean 接口定义了三个核心方法：getObject () 用于返回目标对象实例；getObjectType () 声明目标对象的类型；isSingleton () 控制目标对象是否为单例。

FactoryBean 的核心价值在于将对象的创建逻辑从容器的默认机制中抽离出来，交给开发者通过编程方式灵活控制。普通 Bean 由容器直接通过构造器或工厂方法实例化，而 FactoryBean 则是先实例化 FactoryBean 本身，再通过调用其 getObject () 方法获取实际需要的 Bean，实现了 "工厂" 与 "产品" 的分离。

FactoryBean 在实际应用中有广泛的使用场景。在第三方组件集成方面，如 MyBatis 的 SqlSessionFactoryBean 封装了数据源配置、Mapper 扫描、插件注册等复杂逻辑；RedisTemplate 的创建也使用了 FactoryBean 机制[(53)](https://blog.csdn.net/ZuanShi1111/article/details/150416286)。在 AOP 代理生成方面，ProxyFactoryBean 通过 getObject () 方法动态生成代理对象，将切面逻辑织入目标 Bean[(54)](https://blog.csdn.net/2501_92540271/article/details/149776079)。在复杂对象初始化方面，当对象创建需要读取配置、连接数据库或执行复杂计算时，FactoryBean 可以将这些逻辑封装在 getObject () 方法中，避免 @Bean 方法的臃肿[(56)](https://blog.csdn.net/weixin_51786043/article/details/148202104)。

### 1.6 条件装配与 @Conditional 机制

@Conditional 是 Spring 4.0 引入的元注解，它是 Spring 框架中实现条件化配置的核心机制[(62)](https://juejin.cn/post/7483704056648237068)。@Conditional 的设计体现了 "开闭原则" 与 "策略模式" 的思想，允许开发者根据特定条件决定是否注册 Bean[(62)](https://juejin.cn/post/7483704056648237068)。

@Conditional 注解可以应用于类（通常是 @Configuration 类）或方法（通常是 @Bean 方法）上，它接收一个或多个实现了 Condition 接口的类作为参数[(63)](https://blog.csdn.net/qq_33181292/article/details/155532272)。Condition 接口只有一个 matches () 方法，Spring 在解析配置类时会调用这个方法，如果返回 true 则注册对应的 Bean，否则忽略[(61)](https://www.iesdouyin.com/share/video/7596160606789438783/?region=\&mid=7596160639836474122\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=zAnutlwYLj7WVc3EvV1ZMskZD4OoPtDV_Qqoa9SkbrQ-\&share_version=280700\&ts=1769261008\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)。

Spring 提供了丰富的 @Conditional 派生注解，简化了常见条件判断场景的使用。例如：@ConditionalOnClass 当指定类存在时生效；@ConditionalOnMissingClass 当指定类不存在时生效；@ConditionalOnBean 当容器中存在指定 Bean 时生效；@ConditionalOnMissingBean 当容器中不存在指定 Bean 时生效；@ConditionalOnProperty 当配置属性存在且匹配指定值时生效；@ConditionalOnResource 当指定资源存在时生效等[(59)](https://blog.csdn.net/yezi1238/article/details/149164390)。

条件装配的执行时机是在配置类解析阶段，由 Spring 的后置处理器 ConfigurationClassPostProcessor 触发。条件判断依赖 Spring 上下文的信息，包括环境变量、配置文件、类加载器等。这种机制使得 Spring 应用能够根据不同的运行环境和配置条件，灵活地注册不同的 Bean，极大地提升了应用的可配置性和可扩展性。

### 1.7 环境抽象与配置加载策略

Spring 的环境抽象机制通过 Environment 接口统一管理应用的配置信息，提供了强大而灵活的外部化配置能力。Environment 接口承载所有配置属性，是配置合并的统一入口[(65)](https://guosy.blog.csdn.net/article/details/152333868)。

在 Spring Boot 应用中，配置的加载遵循严格的优先级顺序，高优先级的配置会覆盖低优先级的配置。根据官方文档，配置源的加载顺序如下：通过 SpringApplication.setDefaultProperties (Map) 设置的默认属性；@PropertySource 注解指定的属性源；应用配置文件（application.properties 和 application.yaml）；RandomValuePropertySource；操作系统环境变量；Java 系统属性（System.getProperties ()）；JNDI 属性；ServletContext 初始化参数；ServletConfig 初始化参数；SPRING\_APPLICATION\_JSON 中的属性；命令行参数；测试相关的 properties 属性。

对于应用配置文件，Spring Boot 会自动从以下位置加载 application.properties 和 application.yaml 文件：类路径根目录；类路径下的 /config 包；当前目录；当前目录下的 config 子目录；config 子目录的直接子目录[(167)](https://docs.spring.io/spring-boot/3.4/reference/features/external-config.html)。这个列表按优先级排序，后面的位置会覆盖前面的配置。

Spring Boot 还支持 Profile 特定的配置文件，使用命名约定 application-{profile}.properties。例如，如果激活了 prod profile，那么 application.yaml 和 application-prod.yaml 都会被加载，其中 profile 特定的配置会覆盖非特定的配置。

### 1.8 资源访问抽象与实现策略

Spring 的 Resource 接口提供了统一的资源访问抽象，它是一个功能强大的接口，用于抽象对底层资源的访问[(71)](https://docs.spring.io/spring-framework/docs/4.0.4.RELEASE/spring-framework-reference/html/resources.html)。Resource 接口继承自 InputStreamSource，定义了资源访问的基本方法，包括 exists () 检查资源是否存在、isOpen () 判断资源是否已打开、getURL () 获取资源的 URL 等[(71)](https://docs.spring.io/spring-framework/docs/4.0.4.RELEASE/spring-framework-reference/html/resources.html)。

Resource 接口的设计目标是统一各种资源的访问方式。在传统的 Java 开发中，访问 classpath 资源需要使用 Class.getResourceAsStream ()，访问文件系统资源需要使用 new FileInputStream ()，访问网络资源需要使用 new URL ().openStream ()。Spring 的 Resource 接口将这些不同的资源访问方式统一起来，提供了一致的编程模型[(73)](https://juejin.cn/post/7564235452455157801)。

Spring 提供了多种 Resource 实现类，每种实现类代表一种特定的资源访问策略：ClassPathResource 用于访问类路径下的资源；FileSystemResource 用于访问文件系统资源；UrlResource 用于访问 URL 资源（如 HTTP、FTP 等）；ServletContextResource 用于访问 ServletContext 中的资源；InputStreamResource 用于包装输入流资源；ByteArrayResource 用于包装字节数组资源等[(76)](https://github.com/crisxuan/bestJavaer/blob/master/spring/spring-resource.md)。

ResourceLoader 接口是 Spring 资源加载的核心，ApplicationContext 实现了这个接口，提供了 getResource (String location) 方法用于获取 Resource 对象。ResourceLoader 使用特定的前缀来识别不同类型的资源，例如 classpath: 表示类路径资源，file: 表示文件系统资源，http: 表示网络资源等。如果没有指定前缀，ResourceLoader 会根据具体的 ApplicationContext 实现使用默认的资源类型。

## 2. AOP 面向切面编程

### 2.1 AOP 核心概念与术语体系

AOP（面向切面编程）是 Spring 框架的核心特性之一，它通过将横切关注点与业务逻辑分离，实现了代码的高度模块化和可维护性。AOP 的核心思想是将应用中的通用功能（如日志记录、事务管理、权限控制等）从业务逻辑中分离出来，形成独立的切面，然后在运行时将这些切面织入到目标对象中[(99)](https://juejin.cn/post/7590608345923272730)。

AOP 的核心术语包括：切面（Aspect）是一组相关通知的集合，通常对应一个横切关注点的实现；连接点（Join Point）是程序执行过程中的特定点，如方法调用、异常抛出等，在 Spring AOP 中主要指方法调用；切点（Pointcut）定义了哪些连接点需要被通知，通过切点表达式匹配特定的方法；通知（Advice）定义了在切点处执行的增强逻辑，包括前置通知、后置通知、环绕通知等；引入（Introduction）允许向现有类添加新的方法或属性；织入（Weaving）是将切面应用到目标对象并创建代理对象的过程[(99)](https://juejin.cn/post/7590608345923272730)。

在 Spring AOP 中，连接点始终代表方法执行，这是因为 Spring AOP 基于动态代理实现，只能对方法进行拦截。与 AspectJ 相比，Spring AOP 的功能相对有限，但它提供了与 Spring IoC 容器的无缝集成，并且在大多数企业应用场景中已经足够使用。

### 2.2 通知类型与执行顺序

Spring AOP 支持五种类型的通知，每种通知定义了在特定连接点执行的增强逻辑。理解这些通知类型及其执行顺序对于正确使用 AOP 至关重要。

\*\* 前置通知（@Before）\*\* 在目标方法执行之前执行，通常用于日志记录、权限校验等场景。前置通知不会影响目标方法的执行，除非在前置通知中抛出异常[(99)](https://juejin.cn/post/7590608345923272730)。

\*\* 后置通知（@After）\*\* 在目标方法执行之后执行，无论方法是否抛出异常，类似于 try-finally 块中的 finally 部分。后置通知通常用于资源清理、统计方法执行时间等操作[(99)](https://juejin.cn/post/7590608345923272730)。

\*\* 返回通知（@AfterReturning）\*\* 在目标方法正常返回后执行，只有在方法成功完成且没有抛出异常时才会触发。返回通知可以访问目标方法的返回值，常用于结果记录或缓存处理[(99)](https://juejin.cn/post/7590608345923272730)。

\*\* 异常通知（@AfterThrowing）\*\* 在目标方法抛出异常后执行，只有在方法抛出异常时才会触发。异常通知可以访问抛出的异常对象，常用于异常处理、错误日志记录等场景[(99)](https://juejin.cn/post/7590608345923272730)。

\*\* 环绕通知（@Around）\*\* 是功能最强大的通知类型，它完全包裹目标方法，可以控制目标方法是否执行、修改方法参数、替换返回值等。环绕通知需要显式调用 ProceedingJoinPoint 的 proceed () 方法来执行目标方法[(99)](https://juejin.cn/post/7590608345923272730)。

当一个方法被多个通知增强时，通知的执行顺序遵循特定规则。对于单个切面内的通知，如果使用 @Around 包裹其他通知，执行顺序类似于 try-catch-finally 结构：环绕通知的 before 部分→前置通知→目标方法→环绕通知获取返回值→返回通知→环绕通知的 after 部分（finally）→后置通知。如果目标方法抛出异常，则执行顺序为：环绕通知的 before 部分→前置通知→目标方法抛出异常→异常通知→环绕通知的 after 部分（finally）→后置通知[(99)](https://juejin.cn/post/7590608345923272730)。

### 2.3 JDK 动态代理与 CGLIB 代理机制

Spring AOP 的底层实现基于动态代理技术，主要支持两种代理方式：JDK 动态代理和 CGLIB 代理。理解这两种代理机制的原理和适用场景对于优化 AOP 性能和避免潜在问题至关重要。

**JDK 动态代理**是 Java 原生提供的代理机制，它基于反射原理在运行时动态生成代理类。JDK 动态代理要求目标对象必须实现至少一个接口，代理类会实现与目标对象相同的接口，并在代理方法中织入增强逻辑。JDK 动态代理的优势在于它是 Java 标准库的一部分，无需额外依赖，并且在现代 JVM 中通过反射优化后性能表现良好[(106)](https://blog.csdn.net/2301_80141552/article/details/150989549)。

**CGLIB 代理**通过字节码生成技术在运行时创建目标类的子类，重写非 final 方法来实现代理功能。CGLIB 代理的优势在于它不需要目标对象实现接口，可以代理任何非 final 的类和方法。在高频调用场景下，CGLIB 代理因避免了反射开销，性能通常优于 JDK 代理[(106)](https://blog.csdn.net/2301_80141552/article/details/150989549)。

Spring AOP 的代理选择策略如下：如果目标对象实现了接口，默认使用 JDK 动态代理；如果目标对象没有实现接口，或者强制设置 spring.aop.proxy-target-class=true，则使用 CGLIB 代理[(109)](https://www.iesdouyin.com/share/note/7534669206277705012/?region=\&mid=7168534822384732167\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&schema_type=37\&share_sign=vMTqJ5q21Mz4_pKVqIjBDb85doJ8VtIVG04Sb7q8xWQ-\&share_version=280700\&ts=1769261066\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)。这种策略确保了在大多数场景下都能选择合适的代理方式。

在实际应用中，选择代理方式需要考虑多个因素：目标对象是否有接口；是否需要代理 final 类或方法；性能要求；内存占用等。值得注意的是，CGLIB 代理会增加类加载的开销，并且在某些特殊场景下可能出现问题，如类的构造函数有特殊逻辑、使用了第三方字节码增强库等[(111)](https://juejin.cn/post/7561442148370137139)。

### 2.4 切点表达式与 AspectJ 集成

Spring AOP 使用 AspectJ 的切点表达式语言来定义切点，这种表达式语言提供了丰富的匹配模式，可以精确地定位需要增强的连接点。

切点表达式的基本语法结构为：execution (\[修饰符] 返回值类型 方法全限定名 (\[参数列表]) \[throws 异常])。其中，execution 是最常用的切点指示符，用于匹配方法执行的连接点[(112)](https://blog.csdn.net/weixin_30449239/article/details/99446198)。

**通配符的使用**是切点表达式的核心特性。\* 通配符匹配任意数量的字符（不包括包路径分隔符）；.. 通配符匹配任意数量的字符（包括包路径分隔符），在包名中表示多级包，在参数列表中表示任意参数；+ 通配符匹配指定类及其所有子类[(118)](https://juejin.cn/post/7509042958204420133)。

**常见的切点表达式示例**包括：匹配 service 包下所有类的所有方法：execution (\* com.xxx.service..*.*(..))；匹配 UserService 类的所有 public 方法：execution (public \* com.xxx.service.UserService.*(..))；匹配所有返回 String 类型的方法：execution (String com.xxx..*(..))；匹配带有 @Transactional 注解的方法：@annotation (org.springframework.transaction.annotation.Transactional)[(118)](https://juejin.cn/post/7509042958204420133)。

除了 execution 表达式，Spring 还支持其他类型的切点指示符：within 用于匹配特定类型内的连接点；target 用于匹配目标对象是特定类型的连接点；this 用于匹配代理对象是特定类型的连接点；args 用于匹配方法参数满足特定类型的连接点；@within 用于匹配类或接口上带有特定注解的连接点；@target 用于匹配目标对象带有特定注解的连接点等[(117)](https://www.javascriptcn.com/interview-spring/677f4e6d828d7b20c6314504.html)。

Spring AOP 与 AspectJ 的集成体现在切点表达式的使用上，但需要注意的是，Spring AOP 仅支持方法级别的连接点，而完整的 AspectJ 还支持构造函数、字段等其他连接点。Spring AOP 使用的是 AspectJ 切点表达式语言的一个子集，通过 aspectjweaver 库来解析和执行这些表达式[(112)](https://blog.csdn.net/weixin_30449239/article/details/99446198)。

## 3. 数据访问与事务管理

### 3.1 声明式事务管理原理

Spring 的声明式事务管理是企业级应用开发中的核心功能，它通过 @Transactional 注解和 AOP 机制实现了事务管理与业务逻辑的解耦。声明式事务的核心原理基于 AOP 动态代理和事务拦截器机制[(119)](https://blog.csdn.net/xiaolyuh123/article/details/156830720)。

**事务管理的核心架构**包括三个关键组件：PlatformTransactionManager 事务管理器，它是 Spring 事务管理的核心接口，定义了事务操作的基本方法（getTransaction、commit、rollback）；TransactionDefinition 事务定义，描述事务的属性（隔离级别、传播行为、超时时间、只读属性等）；TransactionStatus 事务状态，代表当前事务的运行状态，用于事务的控制和监控[(124)](https://blog.csdn.net/Prince140678/article/details/146466434)。

**@Transactional 注解的底层机制**通过 AOP 实现。Spring 会为被 @Transactional 标注的 Bean 创建动态代理对象，这个代理对象就是事务的增强器。当调用目标方法时，代理对象会拦截方法调用，根据 @Transactional 的配置信息创建事务、执行目标方法、处理事务的提交或回滚[(119)](https://blog.csdn.net/xiaolyuh123/article/details/156830720)。

事务管理的执行流程如下：首先，AOP 代理拦截目标方法调用；然后，TransactionInterceptor 根据 @Transactional 注解获取事务属性；接着，调用 PlatformTransactionManager 的 getTransaction () 方法获取事务；执行目标方法，如果执行过程中抛出异常，则调用 rollback () 方法回滚事务；如果执行成功，则调用 commit () 方法提交事务[(124)](https://blog.csdn.net/Prince140678/article/details/146466434)。

### 3.2 @Transactional 传播行为详解

@Transactional 注解的 propagation 属性定义了事务的传播行为，用于控制在嵌套方法调用场景下的事务处理策略。Spring 支持七种事务传播行为，每种行为都有其特定的应用场景。

\*\*REQUIRED（默认传播行为）\*\* 表示如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新事务。这是最常用的传播行为，适用于大多数业务场景。例如，方法 A 调用方法 B，若方法 A 已经在事务中，则方法 B 加入方法 A 的事务；若方法 A 没有事务，则为方法 B 创建新事务。

**REQUIRES\_NEW**表示总是创建一个新事务，如果当前存在事务，则将当前事务挂起。这种传播行为适用于需要完全隔离的事务场景，如日志记录、审计等。方法 A 调用方法 B 时，方法 B 会创建新事务，与方法 A 的事务相互独立，方法 B 的异常不会影响方法 A 的事务。

**NESTED**表示如果当前存在事务，则在当前事务中创建一个嵌套事务；如果当前没有事务，则行为与 REQUIRED 相同。嵌套事务可以部分回滚，父事务可以回滚到嵌套事务开始的保存点，而不影响其他部分的执行。这种传播行为适用于需要部分回滚的复杂业务场景。

**SUPPORTS**表示如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务方式执行。这种传播行为适用于事务可选的操作，如只读查询等。

**NOT\_SUPPORTED**表示以非事务方式执行，如果当前存在事务，则将当前事务挂起。这种传播行为适用于不需要事务的操作，如日志记录、缓存更新等。

**NEVER**表示以非事务方式执行，如果当前存在事务，则抛出异常。这种传播行为用于强制要求方法不运行在事务中。

**MANDATORY**表示必须在一个已存在的事务中执行，如果当前没有事务，则抛出异常。这种传播行为用于强制要求方法必须运行在事务中。

### 3.3 事务隔离级别与失效场景

事务隔离级别定义了多个事务之间的隔离程度，用于解决并发事务可能带来的脏读、不可重复读、幻读等问题。Spring 支持五种事务隔离级别，这些级别直接映射到数据库的隔离级别。

**ISOLATION\_DEFAULT**使用数据库默认的隔离级别，不同数据库的默认隔离级别可能不同。例如，MySQL 的默认隔离级别是 REPEATABLE\_READ，Oracle 的默认隔离级别是 READ\_COMMITTED[(127)](https://blog.csdn.net/lhmyy521125/article/details/150565066)。

**ISOLATION\_READ\_UNCOMMITTED**是最低的隔离级别，允许一个事务读取另一个未提交事务的数据。这种隔离级别可能导致脏读、不可重复读、幻读等问题，但性能最好[(127)](https://blog.csdn.net/lhmyy521125/article/details/150565066)。

**ISOLATION\_READ\_COMMITTED**确保一个事务只能读取已提交的数据。这种隔离级别可以避免脏读，但可能出现不可重复读和幻读问题，是大多数数据库的默认隔离级别[(127)](https://blog.csdn.net/lhmyy521125/article/details/150565066)。

**ISOLATION\_REPEATABLE\_READ**确保在同一个事务中多次读取同一数据时，得到相同的结果，即使其他事务修改了该数据。这种隔离级别可以避免脏读和不可重复读，但可能出现幻读问题，是 MySQL 的默认隔离级别[(127)](https://blog.csdn.net/lhmyy521125/article/details/150565066)。

**ISOLATION\_SERIALIZABLE**是最高的隔离级别，它通过强制事务串行执行来避免所有并发问题。这种隔离级别性能最差，但可以完全避免脏读、不可重复读和幻读问题[(127)](https://blog.csdn.net/lhmyy521125/article/details/150565066)。

**事务失效的常见场景**是实际开发中需要特别注意的问题。根据经验总结，事务失效主要包括以下几种情况：

方法不是 public 修饰符：Spring 的事务代理默认只对 public 方法生效，如果在非 public 方法上使用 @Transactional 注解，事务将失效[(126)](https://blog.csdn.net/z55947810/article/details/155201266)。

同类内部方法调用：当在同一个类中，一个方法调用另一个被 @Transactional 标注的方法时，事务会失效。这是因为 Spring 的事务代理是基于 AOP 实现的，内部方法调用不会经过代理对象，因此不会触发事务增强[(126)](https://blog.csdn.net/z55947810/article/details/155201266)。

异常被捕获：如果在被 @Transactional 标注的方法内部捕获了异常，并且没有重新抛出，事务将无法回滚。Spring 默认只对运行时异常（RuntimeException 及其子类）和 Error 进行回滚，对于受检异常需要通过 rollbackFor 属性显式配置[(126)](https://blog.csdn.net/z55947810/article/details/155201266)。

数据库引擎不支持事务：如果使用的数据库引擎不支持事务（如 MySQL 的 MyISAM 引擎），即使配置了 @Transactional，事务也不会生效。应该使用支持事务的引擎，如 InnoDB[(127)](https://blog.csdn.net/lhmyy521125/article/details/150565066)。

事务方法在构造函数或 @PostConstruct 中调用：由于 Spring 的 AOP 代理是在 Bean 初始化完成后创建的，在构造函数或 @PostConstruct 方法中调用事务方法时，代理对象还未创建，因此事务不会生效[(126)](https://blog.csdn.net/z55947810/article/details/155201266)。

### 3.4 数据校验与 Validation 机制

Spring 的数据校验机制基于 JSR-303 Bean Validation 规范，提供了强大而灵活的数据验证能力。数据校验是保证应用数据完整性和业务逻辑正确性的重要手段。

**JSR-303 内置校验注解**提供了基础的验证功能：@NotNull 用于验证对象是否为 null；@NotEmpty 用于验证字符串、集合、数组等是否为空；@NotBlank 用于验证字符串是否为空白（非 null 且长度大于 0）；@Size 用于验证字符串长度或集合、数组的大小；@Min 和 @Max 用于验证数字的最小值和最大值；@Email 用于验证邮箱格式；@Pattern 用于验证正则表达式匹配等[(134)](https://docs.spring.io/spring-framework/reference/6.0/core/validation/beanvalidation.html)。

**@Valid 和 @Validated 注解**用于触发数据校验。@Valid 通常用于方法参数、方法返回值、类属性等位置，用于验证对象及其嵌套对象；@Validated 是 Spring 提供的注解，主要用于类级别和方法级别，支持分组校验功能[(134)](https://docs.spring.io/spring-framework/reference/6.0/core/validation/beanvalidation.html)。

在 Spring MVC 应用中，数据校验通常与 @RequestBody、@RequestParam 等注解结合使用。例如：



```
@PostMapping("/user")

public ResponseEntity\<User> createUser(@Valid @RequestBody User user) {

&#x20;   // 处理用户创建逻辑

}
```

当数据校验失败时，Spring 会抛出 MethodArgumentNotValidException 异常，可以通过 @ControllerAdvice 进行全局异常处理，返回统一的错误响应格式[(133)](https://blog.csdn.net/python15397/article/details/153139481)。

**自定义校验注解**允许开发者实现特定业务规则的验证。创建自定义校验需要以下步骤：定义校验注解，使用 @Constraint 注解指定校验器；实现 ConstraintValidator 接口，编写具体的校验逻辑；在需要校验的字段上使用自定义注解[(135)](https://blog.csdn.net/gitblog_00102/article/details/151282299)。

例如，创建一个 @EnumType 注解用于校验枚举类型：



```
@Target({ METHOD, FIELD, ANNOTATION\_TYPE, CONSTRUCTOR, PARAMETER, TYPE\_USE })

@Retention(RUNTIME)

@Constraint(validatedBy = EnumTypeValidator.class)

public @interface EnumType {

&#x20;   String message() default "枚举类型不合法";

&#x20;   Class\<?>\[] groups() default {};

&#x20;   Class\<? extends Payload>\[] payload() default {};

&#x20;   Class\<? extends Enum\<?>> enumClass();

}

public class EnumTypeValidator implements ConstraintValidator\<EnumType, Object> {

&#x20;   private Class\<? extends Enum\<?>> enumClass;

&#x20;   @Override

&#x20;   public void initialize(EnumType constraintAnnotation) {

&#x20;       this.enumClass = constraintAnnotation.enumClass();

&#x20;   }

&#x20;   @Override

&#x20;   public boolean isValid(Object value, ConstraintValidatorContext context) {

&#x20;       if (value == null) {

&#x20;           return true;

&#x20;       }

&#x20;       try {

&#x20;           Enum\<?>\[] enumConstants = enumClass.getEnumConstants();

&#x20;           for (Enum\<?> enumConstant : enumConstants) {

&#x20;               if (enumConstant.name().equals(value.toString())) {

&#x20;                   return true;

&#x20;               }

&#x20;           }

&#x20;       } catch (Exception e) {

&#x20;           return false;

&#x20;       }

&#x20;       return false;

&#x20;   }

}
```

## 4. Spring MVC 与 Web 开发架构

### 4.1 DispatcherServlet 核心流程

DispatcherServlet 是 Spring MVC 的核心组件，它采用前端控制器模式，作为所有 Web 请求的统一入口。理解 DispatcherServlet 的处理流程对于掌握 Spring MVC 架构至关重要。

**DispatcherServlet 的初始化过程**在 Web 应用启动时完成。在传统的 Servlet 环境中，需要在 web.xml 中配置 DispatcherServlet：



```
\<servlet>

&#x20;   \<servlet-name>dispatcher\</servlet-name>

&#x20;   \<servlet-class>org.springframework.web.servlet.DispatcherServlet\</servlet-class>

&#x20;   \<init-param>

&#x20;       \<param-name>contextConfigLocation\</param-name>

&#x20;       \<param-value>/WEB-INF/spring-mvc-config.xml\</param-value>

&#x20;   \</init-param>

&#x20;   \<load-on-startup>1\</load-on-startup>

\</servlet>

\<servlet-mapping>

&#x20;   \<servlet-name>dispatcher\</servlet-name>

&#x20;   \<url-pattern>/\</url-pattern>

\</servlet-mapping>
```

在 Servlet 3.0 + 环境中，可以通过 Java 配置的方式注册 DispatcherServlet：



```
@Configuration

public class WebConfig implements WebApplicationInitializer {

&#x20;   @Override

&#x20;   public void onStartup(ServletContext servletContext) {

&#x20;       AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();

&#x20;       context.register(MvcConfig.class);

&#x20;      &#x20;

&#x20;       DispatcherServlet dispatcherServlet = new DispatcherServlet(context);

&#x20;       ServletRegistration.Dynamic registration = servletContext.addServlet("dispatcher", dispatcherServlet);

&#x20;       registration.setLoadOnStartup(1);

&#x20;       registration.addMapping("/");

&#x20;   }

}
```

**DispatcherServlet 的核心处理流程**包括以下步骤：



1. **请求接收与预处理**：DispatcherServlet 接收所有符合映射规则的 HTTP 请求，进行初步处理，包括设置请求属性、locale 解析、主题解析等。

2. **处理器映射（Handler Mapping）**：DispatcherServlet 调用 HandlerMapping 查找匹配的处理器（Handler）。HandlerMapping 根据请求 URL、HTTP 方法等信息确定对应的处理器。常用的 HandlerMapping 实现包括 RequestMappingHandlerMapping（基于 @RequestMapping 注解）、BeanNameUrlHandlerMapping（基于 Bean 名称）、SimpleUrlHandlerMapping（基于简单 URL 匹配）等[(142)](https://blog.csdn.net/weixin_73739493/article/details/151051268)。

3. **处理器适配器（Handler Adapter）**：找到处理器后，DispatcherServlet 需要调用合适的 HandlerAdapter 来执行处理器。HandlerAdapter 负责将 DispatcherServlet 的调用适配到具体的处理器类型。常用的 HandlerAdapter 包括 RequestMappingHandlerAdapter（处理 @Controller 注解的处理器）、SimpleControllerHandlerAdapter（处理实现 Controller 接口的处理器）等[(143)](https://www.iesdouyin.com/share/video/7514735876019064104/?region=\&mid=7514737312341805835\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=MgyZrG5JrxvekX5nteTVn3uwuLEj5Mo3zulFXgQwWYA-\&share_version=280700\&ts=1769261096\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)。

4. **处理器执行**：HandlerAdapter 执行处理器，包括数据绑定、数据校验、调用目标方法等步骤。处理器执行后返回 ModelAndView 对象，包含视图名称和模型数据。

5. **视图解析（View Resolver）**：DispatcherServlet 调用 ViewResolver 解析视图名称，得到具体的 View 对象。ViewResolver 根据逻辑视图名和 Locale 信息查找对应的物理视图。常用的 ViewResolver 包括 InternalResourceViewResolver（处理 JSP 视图）、ThymeleafViewResolver（处理 Thymeleaf 模板）等。

6. **视图渲染**：View 对象负责将模型数据渲染到响应中。对于传统的 MVC 应用，视图渲染通常指将模型数据填充到 JSP 或其他模板引擎中；对于 RESTful 应用，通常指将数据转换为 JSON 或 XML 格式的响应体。

7. **响应处理**：将渲染结果写入 HttpServletResponse，完成请求处理流程。

### 4.2 核心注解与参数绑定机制

Spring MVC 提供了丰富的注解支持，这些注解极大地简化了 Web 应用的开发。理解这些核心注解及其使用场景是掌握 Spring MVC 的基础。

**控制器相关注解**：



* @Controller：标识一个控制器类，用于处理 Web 请求。

* @RestController：是 @Controller 和 @ResponseBody 的组合注解，用于创建 RESTful 风格的控制器，自动将返回值转换为响应体。

* @RequestMapping：用于映射 URL 路径，可以标注在类或方法上，支持 URL 模板、请求方法限定等功能。

* @GetMapping、@PostMapping、@PutMapping、@DeleteMapping、@PatchMapping：是 @RequestMapping 的快捷方式，分别对应 HTTP 的 GET、POST、PUT、DELETE、PATCH 方法[(147)](https://blog.csdn.net/2401_83675559/article/details/156147464)。

**请求参数绑定注解**：



* @PathVariable：用于绑定 URL 路径中的占位符参数，例如：



```
@GetMapping("/user/{id}")

public User getUser(@PathVariable Long id) {

&#x20;   // 根据id查询用户

}
```



* @RequestParam：用于绑定查询参数或表单参数，支持 defaultValue、required 等属性：



```
@GetMapping("/users")

public List\<User> getUsers(@RequestParam(defaultValue = "0") int page,&#x20;

&#x20;                          @RequestParam(defaultValue = "10") int size) {

&#x20;   // 分页查询用户

}
```



* @RequestBody：用于绑定请求体中的数据，通常与 @Valid 结合使用进行数据校验：



```
@PostMapping("/user")

public User createUser(@Valid @RequestBody User user) {

&#x20;   // 创建用户

}
```



* @RequestHeader：用于绑定请求头信息。

* @CookieValue：用于绑定 Cookie 值。

**参数绑定机制**的底层原理是通过 DataBinder 和 WebDataBinder 来实现的。Spring MVC 支持多种类型的参数绑定：



* 基本类型和 String 类型的自动转换

* 对象类型的属性绑定

* 集合和数组类型的绑定

* 自定义类型转换器的支持

数据绑定的执行流程包括：提取请求参数、类型转换、数据校验、绑定到目标对象等步骤。当绑定过程中发生错误时，错误信息会存储在 BindingResult 对象中，可以通过方法参数或 Model 对象获取这些错误信息[(133)](https://blog.csdn.net/python15397/article/details/153139481)。

### 4.3 统一异常处理与跨域支持

在 Web 应用开发中，统一的异常处理和跨域支持是保证应用稳定性和可扩展性的重要功能。

**全局异常处理机制**通过 @ControllerAdvice 和 @ExceptionHandler 注解实现。@ControllerAdvice 用于标识全局异常处理类，@ExceptionHandler 用于指定处理的异常类型。例如：



```
@ControllerAdvice

public class GlobalExceptionHandler {

&#x20;   @ExceptionHandler(MethodArgumentNotValidException.class)

&#x20;   public ResponseEntity\<ErrorResponse> handleValidationException(MethodArgumentNotValidException ex) {

&#x20;       List\<String> errors = ex.getBindingResult().getFieldErrors().stream()

&#x20;               .map(FieldError::getDefaultMessage)

&#x20;               .collect(Collectors.toList());

&#x20;       ErrorResponse errorResponse = new ErrorResponse("VALIDATION\_ERROR", errors);

&#x20;       return ResponseEntity.badRequest().body(errorResponse);

&#x20;   }

&#x20;   @ExceptionHandler(UserNotFoundException.class)

&#x20;   public ResponseEntity\<ErrorResponse> handleUserNotFoundException(UserNotFoundException ex) {

&#x20;       ErrorResponse errorResponse = new ErrorResponse("USER\_NOT\_FOUND", Collections.singletonList(ex.getMessage()));

&#x20;       return ResponseEntity.notFound().body(errorResponse);

&#x20;   }

&#x20;   @ExceptionHandler(Exception.class)

&#x20;   public ResponseEntity\<ErrorResponse> handleAllExceptions(Exception ex) {

&#x20;       ErrorResponse errorResponse = new ErrorResponse("INTERNAL\_SERVER\_ERROR", Collections.singletonList("服务器内部错误"));

&#x20;       return ResponseEntity.status(500).body(errorResponse);

&#x20;   }

}
```

这种全局异常处理机制可以统一处理 Controller 层抛出的所有异常，返回标准格式的错误响应，避免了在每个 Controller 中编写大量的 try-catch 代码[(149)](https://blog.csdn.net/m0_46091798/article/details/157258114)。

**跨域资源共享（CORS）支持**是现代 Web 应用开发中的常见需求。Spring MVC 提供了多种方式支持跨域请求：



1. **使用 @CrossOrigin 注解**：可以在 Controller 类或方法上使用 @CrossOrigin 注解，例如：



```
@RestController

@RequestMapping("/api")

@CrossOrigin(origins = "http://localhost:3000", maxAge = 3600)

public class UserController {

&#x20;  &#x20;

&#x20;   @GetMapping("/users")

&#x20;   @CrossOrigin(origins = "\*") // 方法级别的配置会覆盖类级别的配置

&#x20;   public List\<User> getUsers() {

&#x20;       // 获取用户列表

&#x20;   }

}
```



1. **通过 WebMvcConfigurer 配置**：实现 WebMvcConfigurer 接口的 addCorsMappings 方法：



```
@Configuration

public class WebConfig implements WebMvcConfigurer {

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public void addCorsMappings(CorsRegistry registry) {

&#x20;       registry.addMapping("/api/\*\*")

&#x20;               .allowedOrigins("http://localhost:3000", "https://example.com")

&#x20;               .allowedMethods("GET", "POST", "PUT", "DELETE")

&#x20;               .allowedHeaders("\*")

&#x20;               .allowCredentials(true)

&#x20;               .maxAge(3600);

&#x20;   }

}
```



1. **使用 Filter**：可以注册 CorsFilter 来处理跨域请求：



```
@Bean

public CorsFilter corsFilter() {

&#x20;   UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();

&#x20;   CorsConfiguration config = new CorsConfiguration();

&#x20;   config.setAllowedOrigins(Collections.singletonList("\*"));

&#x20;   config.setAllowedMethods(Collections.singletonList("\*"));

&#x20;   config.setAllowedHeaders(Collections.singletonList("\*"));

&#x20;   source.registerCorsConfiguration("/\*\*", config);

&#x20;   return new CorsFilter(source);

}
```

跨域配置需要注意的要点包括：allowedOrigins 指定允许的源，可以使用 "\*" 表示所有源；allowedMethods 指定允许的 HTTP 方法；allowedHeaders 指定允许的请求头；allowCredentials 指定是否允许携带认证信息（如 Cookie）；maxAge 指定预检请求的缓存时间[(147)](https://blog.csdn.net/2401_83675559/article/details/156147464)。

### 4.4 拦截器与过滤器机制对比

在 Spring Web 应用中，拦截器（Interceptor）和过滤器（Filter）都提供了对请求进行预处理和后处理的能力，但它们在实现机制、作用范围、生命周期等方面存在显著差异。

**拦截器（Interceptor）机制**：

拦截器是 Spring MVC 框架的一部分，基于 AOP 思想实现，需要依赖 Spring 环境。拦截器通过实现 HandlerInterceptor 接口或继承 HandlerInterceptorAdapter 抽象类来定义，主要有三个方法：



* preHandle (HttpServletRequest request, HttpServletResponse response, Object handler)：在处理器执行前调用，可以用于权限校验、日志记录等，返回 false 将中断请求处理流程。

* postHandle (HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)：在处理器执行后、视图渲染前调用，可以修改 ModelAndView。

* afterCompletion (HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)：在整个请求完成后（包括视图渲染）调用，通常用于资源清理、统计耗时等。

拦截器通过 WebMvcConfigurer 的 addInterceptors 方法注册：



```
@Configuration

public class WebConfig implements WebMvcConfigurer {

&#x20;  &#x20;

&#x20;   @Autowired

&#x20;   private LoginInterceptor loginInterceptor;

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public void addInterceptors(InterceptorRegistry registry) {

&#x20;       registry.addInterceptor(loginInterceptor)

&#x20;               .addPathPatterns("/api/\*\*")

&#x20;               .excludePathPatterns("/api/login", "/api/register");

&#x20;   }

}
```

**过滤器（Filter）机制**：

过滤器是 Servlet 规范的一部分，独立于任何 Web 框架，在 Servlet 容器级别工作。过滤器通过实现 javax.servlet.Filter 接口来定义，主要有三个方法：



* init (FilterConfig filterConfig)：在过滤器初始化时调用，用于读取配置参数。

* doFilter (ServletRequest request, ServletResponse response, FilterChain chain)：核心方法，用于处理请求和响应。

* destroy ()：在过滤器销毁时调用，用于资源清理[(153)](https://blog.csdn.net/darling_cats/article/details/142108333)。

过滤器通过 @WebFilter 注解或在 web.xml 中配置：



```
@WebFilter(filterName = "characterEncodingFilter", urlPatterns = "/\*")

public class CharacterEncodingFilter implements Filter {

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)&#x20;

&#x20;           throws IOException, ServletException {

&#x20;       request.setCharacterEncoding("UTF-8");

&#x20;       response.setCharacterEncoding("UTF-8");

&#x20;       chain.doFilter(request, response);

&#x20;   }

}
```

**拦截器与过滤器的主要区别**：



1. **作用范围**：

* 过滤器作用于整个 Web 应用，包括静态资源（HTML、CSS、JS 等）、Servlet、JSP 等所有资源。

* 拦截器只作用于 Spring MVC 框架处理的请求，无法拦截静态资源和非 Spring MVC 的请求[(153)](https://blog.csdn.net/darling_cats/article/details/142108333)。

1. **执行时机**：

* 过滤器在请求进入 Servlet 容器后、进入 Servlet 之前执行。

* 拦截器在请求进入 Servlet 之后、执行 Controller 方法之前执行[(156)](https://www.iesdouyin.com/share/video/7482798033780886823/?region=\&mid=7468608251074824208\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=ZOx4tNvC9kQDeImzqXWkDCeuFHJrBuhBzb3Tlyxeau4-\&share_version=280700\&ts=1769261096\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)。

1. **实现机制**：

* 过滤器基于函数回调机制实现。

* 拦截器基于 Spring 的 AOP 机制和反射实现[(156)](https://www.iesdouyin.com/share/video/7482798033780886823/?region=\&mid=7468608251074824208\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=ZOx4tNvC9kQDeImzqXWkDCeuFHJrBuhBzb3Tlyxeau4-\&share_version=280700\&ts=1769261096\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)。

1. **依赖关系**：

* 过滤器独立于 Spring 框架，可以在任何 Servlet 容器中使用。

* 拦截器必须依赖 Spring 环境，只能在 Spring 应用中使用[(153)](https://blog.csdn.net/darling_cats/article/details/142108333)。

1. **功能特点**：

* 过滤器更适合处理与业务逻辑无关的通用功能，如字符编码转换、压缩解压缩、请求日志记录等。

* 拦截器更适合处理与业务逻辑相关的功能，如权限校验、用户认证、数据统计等[(153)](https://blog.csdn.net/darling_cats/article/details/142108333)。

在实际应用中，通常结合使用拦截器和过滤器：使用过滤器处理底层的通用功能，如字符编码、跨域处理等；使用拦截器处理业务相关的功能，如权限控制、日志记录等。这种组合能够充分发挥两者的优势，实现高效、灵活的 Web 应用开发。

## 5. Spring Boot 核心特性

### 5.1 自动配置原理与机制

Spring Boot 的自动配置是其核心特性之一，它通过 "约定优于配置" 的理念，极大地简化了 Spring 应用的配置工作。自动配置的核心机制基于 @EnableAutoConfiguration 注解、SpringFactoriesLoader 和 @Conditional 条件注解[(160)](https://blog.csdn.net/weixin_44289947/article/details/152934791)。

**自动配置的核心流程**如下：



1. **@SpringBootApplication 注解**是 Spring Boot 应用的入口，它实际上是一个组合注解，包含了 @SpringBootConfiguration、@EnableAutoConfiguration 和 @ComponentScan 三个注解。其中，@EnableAutoConfiguration 负责启用自动配置功能[(163)](https://www.iesdouyin.com/share/video/7521646695231261995/?region=\&mid=7521646742656437055\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=aEttfIuhoXE0nfBfbatqGrVGlTrR6k35rN94kVHqqwo-\&share_version=280700\&ts=1769261127\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)。

2. **SpringFactoriesLoader 机制**是自动配置的关键。Spring Boot 在 classpath 下的 META-INF/spring.factories 文件中定义了所有的自动配置类。例如，spring-boot-autoconfigure 模块中的 spring.factories 文件包含了大量的自动配置类，如 DataSourceAutoConfiguration、WebMvcAutoConfiguration 等[(160)](https://blog.csdn.net/weixin_44289947/article/details/152934791)。

3. **条件化配置**通过 @Conditional 系列注解实现。自动配置类使用各种条件注解来判断是否应该生效，例如：

* @ConditionalOnClass：当类路径下存在指定类时生效

* @ConditionalOnMissingBean：当容器中不存在指定 Bean 时生效

* @ConditionalOnProperty：当指定的配置属性存在且匹配时生效

* @ConditionalOnWebApplication：当应用是 Web 应用时生效

* @ConditionalOnNotWebApplication：当应用不是 Web 应用时生效[(160)](https://blog.csdn.net/weixin_44289947/article/details/152934791)

1. **自动配置类的执行顺序**通过 @AutoConfigureOrder 和 @AutoConfigureAfter 注解控制。这些注解确保自动配置类按照正确的顺序执行，避免依赖问题[(162)](https://blog.csdn.net/yzyyishi/article/details/150590104)。

自动配置的工作原理可以概括为：Spring Boot 启动时，@EnableAutoConfiguration 注解会触发 SpringFactoriesLoader 加载所有的自动配置类；然后，Spring 会根据条件注解判断每个自动配置类是否应该生效；对于生效的自动配置类，会注册相应的 Bean 定义到容器中；开发者可以通过自定义配置来覆盖自动配置的默认值，实现 "约定与配置" 的平衡[(166)](https://blog.51cto.com/u_16213627/14365236)。

### 5.2 Starter 机制与依赖管理

Spring Boot 的 Starter 机制是其依赖管理的核心创新，它通过提供精心设计的依赖模块，让开发者能够快速添加特定功能到应用中。Starter 本质上是一个 Maven 或 Gradle 的依赖描述符，它声明了应用需要的一组相关依赖。

**Starter 命名约定**：Spring Boot 官方提供的 Starter 遵循统一的命名规范：spring-boot-starter-\*，其中星号部分表示特定的功能。例如：



* spring-boot-starter-web：Web 应用开发（包括 Tomcat 和 Spring MVC）

* spring-boot-starter-data-jpa：JPA 数据访问（包括 Hibernate 和 Spring Data JPA）

* spring-boot-starter-test：测试支持（包括 JUnit、Spring Test 等）

* spring-boot-starter-security：安全认证支持

* spring-boot-starter-mail：邮件发送支持[(161)](https://developer.aliyun.com:443/article/1689530)

**自定义 Starter 的实现**通常包含以下步骤：



1. 创建一个 Maven 或 Gradle 项目，作为 Starter 的基础。

2. 在 pom.xml 或 build.gradle 中声明 Starter 的依赖。例如，创建一个 spring-boot-starter-mybatis：



```
\<dependencies>

&#x20;   \<dependency>

&#x20;       \<groupId>org.springframework.boot\</groupId>

&#x20;       \<artifactId>spring-boot-autoconfigure\</artifactId>

&#x20;   \</dependency>

&#x20;   \<dependency>

&#x20;       \<groupId>org.mybatis.spring.boot\</groupId>

&#x20;       \<artifactId>mybatis-spring-boot-starter\</artifactId>

&#x20;       \<version>2.0.6\</version>

&#x20;   \</dependency>

\</dependencies>
```



1. 创建自动配置类，使用 @Configuration 和 @Conditional 注解实现条件化配置。

2. 在 META-INF/spring.factories 文件中声明自动配置类：



```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\\

com.example.autoconfigure.MybatisAutoConfiguration
```



1. 提供必要的属性配置类，使用 @ConfigurationProperties 注解绑定配置属性。

Starter 机制的优势在于：简化了依赖管理，开发者无需了解每个功能所需的所有依赖；提供了一致的项目结构和配置方式；支持条件化配置，避免不必要的依赖冲突；便于团队内部或开源社区分享通用功能模块[(161)](https://developer.aliyun.com:443/article/1689530)。

### 5.3 外部化配置加载顺序

Spring Boot 的外部化配置机制允许将应用配置与代码分离，支持多种配置源和灵活的加载顺序。理解配置加载顺序对于正确管理应用配置至关重要。

根据官方文档，Spring Boot 的配置源加载顺序（从高到低）如下：



1. **命令行参数**：通过命令行指定的参数（如 --spring.profiles.active=prod）具有最高优先级。

2. **SPRING\_APPLICATION\_JSON**：可以通过环境变量或系统属性指定 inline JSON 格式的配置。

3. **ServletConfig 初始化参数**：ServletContext 中的初始化参数。

4. **ServletContext 初始化参数**：ServletContext 的上下文参数。

5. **JNDI 属性**：来自 java:comp/env 的 JNDI 属性。

6. **Java 系统属性（System.getProperties ()）**：可以通过 - D 参数指定的系统属性。

7. **操作系统环境变量**：系统环境变量。

8. **RandomValuePropertySource**：用于生成随机值的特殊属性源（如 \${random.int}）。

9. **应用配置文件**：这是最常用的配置源，包括：

* application.properties 和 application.yaml（所有 profile）

* application-{profile}.properties 和 application-{profile}.yaml（特定 profile）

1. **@PropertySource 注解指定的属性源**：通过 @PropertySource 注解加载的属性文件。

2. **默认属性**：通过 SpringApplication.setDefaultProperties (Map) 设置的默认属性[(170)](https://blog.csdn.net/ma_nong33/article/details/140558777)。

对于应用配置文件，Spring Boot 会按以下顺序搜索和加载（优先级从高到低）：



1. 当前目录下的 config 子目录

2. 当前目录

3. 类路径下的 config 包

4. 类路径根目录

例如，对于 application.properties 文件，Spring Boot 会依次检查以下位置：



* file:./config/application.properties

* file:./application.properties

* classpath:/config/application.properties

* classpath:/application.properties

当多个位置存在同名配置时，后面加载的配置会覆盖前面的配置。这种设计允许开发者在不同环境中灵活地覆盖默认配置[(167)](https://docs.spring.io/spring-boot/3.4/reference/features/external-config.html)。

**Profile 特定配置**的加载遵循类似规则。如果激活了 prod profile，Spring Boot 会加载：



* application.properties（或 yaml）

* application-prod.properties（或 yaml）

其中，application-prod.properties 中的配置会覆盖 application.properties 中的同名配置。

### 5.4 内嵌 Web 容器架构

Spring Boot 的内嵌 Web 容器是其 "可执行 Jar" 特性的核心，它允许将 Web 应用打包成一个独立的可执行文件，无需外部 Servlet 容器即可运行。这种设计极大地简化了应用的部署和运维。

Spring Boot 默认支持以下内嵌 Web 容器：



1. **Tomcat**：默认的 Web 容器，适合大多数 Java Web 应用场景。Tomcat 是最成熟、最广泛使用的 Servlet 容器，提供了稳定可靠的 Web 服务支持。

2. **Jetty**：轻量级的 Web 容器，支持 WebSocket 和嵌入式 JDBC，在某些高并发场景下表现优异。Jetty 的启动速度快，内存占用少，适合资源受限的环境[(175)](https://blog.csdn.net/zp357252539/article/details/147181807)。

3. **Undertow**：由 Red Hat 开发的高性能 Web 容器，基于 NIO2 异步 IO 模型，支持 HTTP/2 和 WebSocket。Undertow 在处理大量并发连接时具有出色的性能表现，特别适合微服务和云原生应用[(175)](https://blog.csdn.net/zp357252539/article/details/147181807)。

4. **Reactor Netty**：用于 Spring WebFlux 响应式 Web 应用的反应式 Web 服务器，基于 Netty 构建，提供了非阻塞的高性能 Web 服务能力[(174)](https://www.spring-doc.cn/spring-boot/3.3.1/how-to_webserver.html)。

**容器切换机制**：在 Spring Boot 应用中，可以通过修改依赖来切换 Web 容器。例如，从默认的 Tomcat 切换到 Undertow：



```
\<!-- 排除默认的Tomcat依赖 -->

\<dependency>

&#x20;   \<groupId>org.springframework.boot\</groupId>

&#x20;   \<artifactId>spring-boot-starter-web\</artifactId>

&#x20;   \<exclusions>

&#x20;       \<exclusion>

&#x20;           \<groupId>org.springframework.boot\</groupId>

&#x20;           \<artifactId>spring-boot-starter-tomcat\</artifactId>

&#x20;       \</exclusion>

&#x20;   \</exclusions>

\</dependency>

\<!-- 添加Undertow依赖 -->

\<dependency>

&#x20;   \<groupId>org.springframework.boot\</groupId>

&#x20;   \<artifactId>spring-boot-starter-undertow\</artifactId>

\</dependency>
```

**容器配置**：Spring Boot 提供了丰富的配置选项来定制内嵌 Web 容器的行为。常见的配置包括：



* 服务器端口配置：server.port=8080

* 上下文路径配置：server.servlet.context-path=/app

* 会话配置：server.servlet.session.timeout=30m

* Tomcat 特定配置：server.tomcat.max-threads=200

* Undertow 特定配置：server.undertow.io-threads=8

这些配置可以在 application.properties 或 application.yaml 文件中设置，也可以通过环境变量或命令行参数指定[(173)](https://docs.spring.io/spring-boot/3.3/how-to/webserver.html)。

**嵌入式容器的优势**：



* 简化部署：应用和容器打包成一个文件，无需单独安装和配置 Servlet 容器

* 统一环境：开发、测试、生产环境保持一致，避免 "环境不一致" 问题

* 快速启动：内嵌容器的启动速度通常比独立容器更快

* 灵活配置：可以通过配置文件灵活调整容器参数

* 支持多种容器：根据应用需求选择最适合的 Web 容器[(178)](https://blog.csdn.net/haohaizi_liu/article/details/154063534)。

## 6. 高级特性与扩展机制

### 6.1 事件驱动模型与 ApplicationEvent

Spring 的事件驱动模型基于观察者设计模式，提供了一种松耦合的组件间通信机制。这种机制允许应用中的不同组件通过事件进行通信，而无需直接依赖，极大地提高了应用的可扩展性和可维护性。

**事件驱动模型的核心组件**：



1. **ApplicationEvent（事件对象）**：所有应用事件的基类，是一个抽象类，定义了事件的基本属性，如事件源（source）、事件发生时间（timestamp）等。开发者可以通过继承 ApplicationEvent 类来创建自定义事件[(180)](https://blog.csdn.net/zuiyuelong/article/details/150165155)。



```
public class UserRegisteredEvent extends ApplicationEvent {

&#x20;   private final User user;

&#x20;  &#x20;

&#x20;   public UserRegisteredEvent(User user) {

&#x20;       super(user);

&#x20;       this.user = user;

&#x20;   }

&#x20;  &#x20;

&#x20;   public User getUser() {

&#x20;       return user;

&#x20;   }

}
```



1. **ApplicationListener（事件监听器）**：定义了事件处理的接口，只有一个 onApplicationEvent 方法。监听器通过实现 ApplicationListener 接口来处理特定类型的事件[(180)](https://blog.csdn.net/zuiyuelong/article/details/150165155)。



```
@Component

public class UserRegistrationListener implements ApplicationListener\<UserRegisteredEvent> {

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public void onApplicationEvent(UserRegisteredEvent event) {

&#x20;       User user = event.getUser();

&#x20;       // 处理用户注册事件，如发送欢迎邮件、记录日志等

&#x20;       System.out.println("用户注册事件：" + user.getUsername());

&#x20;   }

}
```



1. **ApplicationEventPublisher（事件发布器）**：用于发布事件的接口。ApplicationContext 实现了这个接口，因此可以在 Spring Bean 中直接注入 ApplicationEventPublisher 来发布事件[(97)](https://juejin.cn/post/7569072953901334579)。



```
@Service

public class UserService {

&#x20;  &#x20;

&#x20;   @Autowired

&#x20;   private ApplicationEventPublisher eventPublisher;

&#x20;  &#x20;

&#x20;   public void registerUser(User user) {

&#x20;       // 执行用户注册逻辑

&#x20;       // 发布用户注册事件

&#x20;       eventPublisher.publishEvent(new UserRegisteredEvent(user));

&#x20;   }

}
```

**事件发布与监听的执行流程**：



1. 应用代码调用 ApplicationEventPublisher 的 publishEvent 方法发布事件

2. Spring 通过 ApplicationEventMulticaster 查找所有注册的监听器

3. 筛选出能够处理该事件类型的监听器（包括父类和接口）

4. 依次调用这些监听器的 onApplicationEvent 方法

5. 如果需要异步处理，可以通过 @Async 注解实现异步事件监听[(96)](https://blog.csdn.net/weixin_46619605/article/details/146566012)

**@EventListener 注解**是 Spring 4.2 引入的简化事件监听的注解，它可以直接标注在方法上，无需实现 ApplicationListener 接口：



```
@Component

public class UserEventProcessor {

&#x20;  &#x20;

&#x20;   @EventListener

&#x20;   public void handleUserRegisteredEvent(UserRegisteredEvent event) {

&#x20;       // 处理用户注册事件

&#x20;   }

&#x20;  &#x20;

&#x20;   @EventListener(condition = "#event.user.age > 18")

&#x20;   public void handleAdultUserRegisteredEvent(UserRegisteredEvent event) {

&#x20;       // 处理成年用户注册事件

&#x20;   }

}
```

**事件的继承机制**允许一个监听器处理多种相关事件。例如，如果定义了一个基类事件 BaseEvent 和多个子类事件（UserEvent、OrderEvent 等），那么监听 BaseEvent 的监听器也会收到所有子类事件[(180)](https://blog.csdn.net/zuiyuelong/article/details/150165155)。

### 6.2 异步处理与调度机制

Spring 提供了强大的异步处理和调度机制，支持 @Async 注解实现方法的异步执行和 @Scheduled 注解实现定时任务调度。这些机制对于提高应用性能和实现定时任务至关重要。

**@Async 异步处理机制**：



1. **启用异步支持**：在配置类上添加 @EnableAsync 注解来启用异步处理功能：



```
@Configuration

@EnableAsync

public class AsyncConfig {

}
```



1. **使用 @Async 注解**：在需要异步执行的方法上添加 @Async 注解：



```
@Service

public class AsyncService {

&#x20;  &#x20;

&#x20;   @Async

&#x20;   public CompletableFuture\<String> asyncMethod() {

&#x20;       // 执行耗时操作

&#x20;       return CompletableFuture.completedFuture("异步执行完成");

&#x20;   }

}
```



1. **自定义线程池配置**：Spring Boot 提供了自动配置的线程池，可以通过 application.properties 进行配置：



```
spring.task.execution.pool.core-size=8

spring.task.execution.pool.max-size=16

spring.task.execution.pool.queue-capacity=100

spring.task.execution.pool.keep-alive=60s
```

也可以通过实现 AsyncConfigurer 接口来自定义线程池：



```
@Configuration

@EnableAsync

public class AsyncConfig implements AsyncConfigurer {

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public Executor getAsyncExecutor() {

&#x20;       ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();

&#x20;       executor.setCorePoolSize(8);

&#x20;       executor.setMaxPoolSize(16);

&#x20;       executor.setQueueCapacity(100);

&#x20;       executor.setThreadNamePrefix("async-thread-");

&#x20;       executor.initialize();

&#x20;       return executor;

&#x20;   }

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {

&#x20;       return new SimpleAsyncUncaughtExceptionHandler();

&#x20;   }

}
```

**@Scheduled 定时任务机制**：



1. **启用调度支持**：在配置类上添加 @EnableScheduling 注解：



```
@Configuration

@EnableScheduling

public class SchedulingConfig {

}
```



1. **使用 @Scheduled 注解**：支持多种调度方式：



```
@Service

public class ScheduledService {

&#x20;  &#x20;

&#x20;   @Scheduled(fixedRate = 5000)

&#x20;   public void fixedRateTask() {

&#x20;       System.out.println("Fixed rate task executed at: " + new Date());

&#x20;   }

&#x20;  &#x20;

&#x20;   @Scheduled(fixedDelay = 3000, initialDelay = 1000)

&#x20;   public void fixedDelayTask() {

&#x20;       System.out.println("Fixed delay task executed at: " + new Date());

&#x20;   }

&#x20;  &#x20;

&#x20;   @Scheduled(cron = "0 0/1 \* \* \* ?")

&#x20;   public void cronTask() {

&#x20;       System.out.println("Cron task executed at: " + new Date());

&#x20;   }

}
```



* fixedRate：按固定速率执行，上一次开始执行时间点之后每隔指定时间执行一次

* fixedDelay：按固定延迟执行，上一次执行完成时间点之后每隔指定时间执行一次

* cron：使用 Cron 表达式定义执行时间

* initialDelay：首次执行延迟时间

1. **调度线程池配置**：可以通过 application.properties 配置调度线程池：



```
spring.task.scheduling.pool.size=10

spring.task.scheduling.thread-name-prefix=scheduling-
```

**高级特性**：



1. **@SchedulerLock 分布式锁**：当应用部署在多个节点时，可以使用 @SchedulerLock 注解来确保同一个定时任务在集群中只执行一次：



```
@Scheduled(cron = "0 0 0 \* \* ?")

@SchedulerLock(name = "dailyCleanup", lockAtMostFor = "PT5M")

public void dailyCleanup() {

&#x20;   // 执行每日清理任务

}
```



1. **动态调度**：可以通过实现 SchedulingConfigurer 接口来动态配置调度任务：



```
@Configuration

@EnableScheduling

public class DynamicSchedulingConfig implements SchedulingConfigurer {

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {

&#x20;       taskRegistrar.addTriggerTask(

&#x20;           () -> System.out.println("Dynamic task executed"),

&#x20;           triggerContext -> {

&#x20;               // 动态计算下次执行时间

&#x20;               return new CronTrigger("0 0/5 \* \* \* ?").nextExecutionTime(triggerContext);

&#x20;           }

&#x20;       );

&#x20;   }

}
```

### 6.3 扩展点机制与最佳实践

Spring 框架提供了丰富的扩展点，允许开发者在不修改框架核心代码的情况下，定制和增强框架的功能。理解和掌握这些扩展点对于构建灵活、可维护的 Spring 应用至关重要。

**BeanPostProcessor 扩展点**：

BeanPostProcessor 是 Spring 中最重要的扩展点之一，它允许在 Bean 初始化前后插入自定义逻辑。Spring 内部大量使用 BeanPostProcessor 来实现核心功能，如 @Autowired 注解的依赖注入、@PostConstruct 和 @PreDestroy 注解的处理、AOP 代理的创建等[(78)](http://m.toutiao.com/group/7592874726532383267/?upstream_biz=doubao)。

自定义 BeanPostProcessor 的实现示例：



```
@Component

public class CustomBeanPostProcessor implements BeanPostProcessor {

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {

&#x20;       System.out.println("Before initialization: " + beanName);

&#x20;       return bean;

&#x20;   }

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {

&#x20;       System.out.println("After initialization: " + beanName);

&#x20;       return bean;

&#x20;   }

}
```

**BeanFactoryPostProcessor 扩展点**：

BeanFactoryPostProcessor 在 BeanDefinition 加载完成后、Bean 实例化之前执行，主要用于修改 BeanDefinition 的配置信息。常见应用场景包括：替换占位符、动态注册 BeanDefinition、根据环境条件修改 Bean 配置等[(86)](https://blog.csdn.net/weixin_46425661/article/details/144420284)。

**ApplicationListener 扩展点**：

除了用于业务事件外，Spring 自身也发布了大量应用事件，开发者可以监听这些事件来扩展应用功能：



* ApplicationStartingEvent：应用开始启动时发布

* ApplicationEnvironmentPreparedEvent：环境准备完成后发布

* ApplicationPreparedEvent：ApplicationContext 准备完成后发布

* ApplicationStartedEvent：应用启动完成后发布

* ApplicationReadyEvent：应用准备好接收请求时发布

* ApplicationFailedEvent：应用启动失败时发布[(185)](https://blog.csdn.net/weixin_43004044/article/details/131754324)

监听这些事件可以实现一些高级功能，如：在应用启动完成后执行特定初始化任务；在应用准备好时发送健康检查通知；在应用启动失败时执行清理操作等。

**EnvironmentPostProcessor 扩展点**：

EnvironmentPostProcessor 允许在 ApplicationContext 刷新之前对 Environment 进行修改。这是 Spring Boot 提供的一个非常早期的扩展点，常用于：


* 从特殊位置加载配置（如云端配置中心）

* 对配置值进行加密解密处理

* 根据环境变量动态修改配置

创建自定义 EnvironmentPostProcessor：



```
public class CustomEnvironmentPostProcessor implements EnvironmentPostProcessor {

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {

&#x20;       // 自定义环境处理逻辑

&#x20;       MutablePropertySources propertySources = environment.getPropertySources();

&#x20;       propertySources.addLast(new ResourcePropertySource("custom-property-source", "classpath:custom.properties"));

&#x20;   }

}
```

然后在 META-INF/spring.factories 中声明：



```
org.springframework.boot.env.EnvironmentPostProcessor=\\

com.example.CustomEnvironmentPostProcessor
```

**ApplicationRunner 和 CommandLineRunner 扩展点**：

这两个接口允许在 Spring 应用启动完成后执行特定的初始化任务。它们的区别在于：



* ApplicationRunner 的 run 方法接收 ApplicationArguments 参数，提供了更友好的参数访问方式

* CommandLineRunner 的 run 方法接收 String 数组参数，与 main 方法的参数一致

通常使用场景包括：



* 加载数据到缓存

* 建立外部系统连接

* 执行数据迁移或初始化脚本

* 注册定时任务或监听器



```
@Component

public class ApplicationStartupRunner implements ApplicationRunner {

&#x20;  &#x20;

&#x20;   @Override

&#x20;   public void run(ApplicationArguments args) throws Exception {

&#x20;       System.out.println("Application started with arguments: " + args.getSourceArgs());

&#x20;       // 执行应用启动后的初始化任务

&#x20;   }

}
```

**最佳实践总结**：



1. **合理使用扩展点**：优先使用已有的扩展点，避免直接修改 Spring 核心代码。这样可以保证与 Spring 版本的兼容性，便于后续升级。

2. **控制扩展逻辑的复杂度**：扩展点方法应该保持简洁，避免复杂的业务逻辑。如果逻辑过于复杂，应该将其封装到独立的 Service 中。

3. **注意执行顺序**：某些扩展点（如 BeanPostProcessor）的执行顺序可能影响功能，需要通过实现 Ordered 接口或 @Order 注解来控制顺序。

4. **避免循环依赖**：在扩展点中获取 Bean 时要小心，避免产生循环依赖。可以考虑使用 ApplicationContextAware 接口来延迟获取 Bean。

5. **测试扩展逻辑**：扩展点的逻辑也需要进行充分测试，可以通过 Spring Boot Test 框架来验证扩展功能的正确性。

6. **文档说明**：对于自定义的扩展点实现，应该提供清晰的文档说明，包括使用场景、配置方式、注意事项等，便于团队其他成员理解和维护。

## 7. 架构设计与性能优化

### 7.1 基于 Spring 的微服务架构设计

随着云原生技术的发展，基于 Spring 的微服务架构已成为企业级应用的主流选择。Spring Cloud 作为 Spring 生态系统的重要组成部分，提供了完整的微服务解决方案。

**核心组件与架构模式**：

1. **服务注册与发现（Eureka/Nacos）**：

* Eureka 是 Netflix 开源的服务注册中心，Spring Cloud 集成了 Eureka Client 和 Eureka Server

* Nacos 是阿里巴巴开源的服务发现和配置管理平台，提供了更丰富的功能

* 服务注册：每个微服务启动时向注册中心注册自己的服务信息

* 服务发现：客户端通过注册中心获取服务列表，实现服务间的调用

1. **服务网关（Spring Cloud Gateway）**：

* 统一的入口网关，负责请求路由、负载均衡、流量控制、安全认证等功能

* 基于 Spring WebFlux 构建，支持异步非阻塞处理

* 提供了丰富的路由规则和过滤器链机制

1. **配置中心（Spring Cloud Config）**：

* 集中管理各微服务的配置，支持版本控制、环境隔离、配置继承等功能

* 可以与 Git、SVN 等版本控制系统集成

* 支持配置的动态刷新（配合 @RefreshScope 注解）

1. **服务调用（Feign/RestTemplate）**：

* Feign：声明式的 HTTP 客户端，简化了服务间的 RESTful 调用

* RestTemplate：Spring 提供的同步 HTTP 客户端

* 支持负载均衡（配合 Ribbon）和断路器（配合 Hystrix 或 Resilience4J）

1. **分布式事务（Seata）**：

* 解决微服务架构中的分布式事务问题

* 提供了 AT、TCC、SAGA 等多种事务模式

* 与 Spring Cloud 和 Spring Boot 有良好的集成

**架构设计最佳实践**：



1. **服务拆分原则**：

* 单一职责原则：每个微服务应该专注于单一业务功能

* 高内聚低耦合：服务内部功能高度相关，服务间依赖尽可能少

* 业务边界清晰：按照业务领域进行服务划分

* 规模适中：避免服务过大或过小，通常建议每个服务包含 10-100 个类

1. **服务间通信策略**：

* 优先使用 RESTful API 进行服务间通信

* 采用异步消息队列处理非关键业务流程

* 使用 gRPC 或其他高性能协议处理对性能要求极高的场景

* 实现服务接口的幂等性，确保重试安全

1. **数据一致性设计**：

* 采用最终一致性模型处理跨服务数据操作

* 使用事件源（Event Sourcing）和 CQRS 模式处理复杂业务场景

* 实现分布式事务时优先考虑可靠事件模式

1. **监控与链路追踪**：

* 集成 Spring Boot Actuator 提供健康检查和指标监控

* 使用 Spring Cloud Sleuth 实现分布式链路追踪

* 配合 Zipkin、Jaeger 等工具进行链路分析

* 实现统一的日志收集和分析系统

1. **部署与运维**：

* 使用 Docker 容器化部署，配合 Kubernetes 进行容器编排

* 实现蓝绿部署、灰度发布等高级部署策略

* 建立完善的告警和故障恢复机制

* 使用配置管理工具（如 Ansible）进行基础设施自动化管理

### 7.2 性能优化策略与实践

在企业级应用开发中，性能优化是一个持续的过程。基于 Spring 的应用可以从多个层面进行性能优化。

**应用层性能优化**：



1. **减少 Bean 创建和依赖注入开销**：

* 使用 @Lazy 注解延迟加载非必需的 Bean

* 避免在循环中使用 @Autowired 注入 Bean

* 优先使用构造器注入，避免使用 setter 注入

* 合理设置 Bean 的作用域，避免不必要的 prototype Bean

1. **优化 AOP 使用**：

* 避免在高频调用的方法上使用 AOP

* 精确设置切点表达式，避免匹配过多的方法

* 考虑使用静态代理替代动态代理（如 CGLIB）

* 使用 @EnableAspectJAutoProxy 的 proxyTargetClass 属性优化代理策略

1. **缓存优化**：

* 使用 Spring Cache 抽象统一管理缓存

* 合理设置缓存过期时间，避免内存溢出

* 使用多级缓存策略（如 Redis + 本地缓存）

* 实现缓存预热，减少首次访问延迟

1. **异步处理优化**：

* 对耗时操作使用 @Async 异步处理

* 合理配置线程池大小，避免线程过多导致的上下文切换开销

* 使用 CompletableFuture 进行异步编排，提高代码可读性

**数据库访问优化**：



1. **查询优化**：

* 使用索引优化查询性能

* 避免 SELECT \*，只查询需要的字段

* 使用分页查询（Pageable）处理大数据量

* 优化关联查询，避免 N+1 问题

1. **连接池优化**：

* 合理配置数据库连接池大小（通常为 CPU 核心数的 2-4 倍）

* 使用 HikariCP 作为连接池，它在性能上表现优异

* 配置合适的连接超时时间和空闲连接回收策略

1. **事务优化**：

* 减少事务的作用范围，只在必要时使用事务

* 使用批量操作减少数据库交互次数

* 优化事务隔离级别，在保证数据一致性的前提下使用较低的隔离级别

**Web 层性能优化**：



1. **HTTP 请求优化**：

* 使用压缩（Gzip）减少响应数据大小

* 实现浏览器缓存策略，设置合适的 Cache-Control 头

* 使用 CDN 加速静态资源访问

* 优化 Cookie 策略，减少 Cookie 大小和数量

1. **响应式编程**：

* 使用 Spring WebFlux 构建响应式 Web 应用

* 利用 Reactor 的背压机制处理高并发场景

* 使用 WebClient 替代 RestTemplate 进行异步 HTTP 调用

1. **异步 Servlet 支持**：

* 启用 Spring MVC 的异步请求处理

* 使用 DeferredResult 或 Callable 处理长时间运行的请求

* 避免在异步处理中阻塞线程

**基础设施优化**：



1. **垃圾回收优化**：

* 使用 G1 垃圾回收器，合理设置堆大小

* 监控 GC 日志，优化 GC 参数

* 避免创建大量短期对象

1. **类加载优化**：

* 使用类加载器层次结构优化类加载速度

* 合理使用缓存类加载器

* 避免过多的类加载器

1. **网络优化**：

* 使用连接池复用网络连接

* 优化 TCP 参数（如拥塞窗口、超时时间等）

* 使用 NIO 替代传统的 BIO 进行网络通信

**性能监控与调优流程**：


1. **性能基线建立**：

* 使用 JMeter、Gatling 等工具进行基准测试

* 记录各项性能指标（响应时间、吞吐量、错误率等）

* 建立性能监控仪表盘

1. **性能瓶颈识别**：

* 使用 APM 工具（如 New Relic、AppDynamics）进行性能分析

* 分析线程转储识别线程竞争

* 分析 GC 日志识别内存问题

* 使用 SQL 监控工具识别慢查询

1. **优化实施与验证**：

* 制定优化方案，优先处理最严重的性能瓶颈

* 实施优化措施，注意不要引入新的问题

* 重新进行性能测试，验证优化效果

* 记录优化过程和结果，建立性能优化知识库

### 7.3 面试真题解析与答题技巧

Spring 相关的技术问题通常涉及深度和广度两个维度。以下是一些常见的面试真题及其解析，帮助你更好地准备面试。

**基础概念类问题**：

1. **问题**：什么是 Spring IoC 容器？它的核心作用是什么？

   **参考答案**：Spring IoC 容器是 Spring 框架的核心，它负责创建、配置和管理 Bean 对象。IoC（控制反转）的核心思想是将对象的创建和依赖关系的管理从应用代码中转移到容器中。容器通过读取配置元数据（XML、注解或 Java 配置）来了解需要创建哪些对象以及它们之间的依赖关系，然后在运行时创建和装配这些对象。这种设计实现了对象之间的松耦合，提高了代码的可测试性和可维护性。

2. **问题**：BeanFactory 和 ApplicationContext 有什么区别？

   **参考答案**：BeanFactory 是 Spring IoC 容器的基础接口，提供了基本的 Bean 管理功能，如获取 Bean、检查 Bean 是否存在等。ApplicationContext 是 BeanFactory 的子接口，它在 BeanFactory 基础上添加了更多企业级功能，包括：与 Spring AOP 功能的集成；国际化消息支持；事件发布机制；Web 应用支持（WebApplicationContext）等。在实际应用中，ApplicationContext 提供了更丰富的功能和更便捷的开发体验，是大多数应用的首选。

3. **问题**：Spring 如何解决循环依赖？

   **参考答案**：Spring 通过三级缓存机制解决单例 Bean 的循环依赖问题。一级缓存（singletonObjects）存储完全初始化的 Bean；二级缓存（earlySingletonObjects）存储提前暴露的原始 Bean；三级缓存（singletonFactories）存储 Bean 的 ObjectFactory。当创建 Bean A 时，先将其 ObjectFactory 放入三级缓存；在填充 A 的属性时发现需要 Bean B，于是创建 B 并将其 ObjectFactory 放入三级缓存；当 B 需要 A 时，从三级缓存获取 A 的 ObjectFactory 生成早期引用。这种机制通过提前暴露未完全初始化的 Bean 引用来打破循环依赖。

**原理机制类问题**：



1. **问题**：@Transactional 注解的底层原理是什么？

   **参考答案**：@Transactional 基于 Spring AOP 实现，通过动态代理机制在目标方法前后织入事务逻辑。当 Spring 容器扫描到 @Transactional 注解时，会为目标 Bean 创建代理对象；在调用目标方法时，代理会拦截方法调用，根据配置创建事务、执行方法、处理提交或回滚。核心组件包括 TransactionInterceptor（事务拦截器）和 PlatformTransactionManager（事务管理器）。

2. **问题**：Spring AOP 的代理机制是怎样的？JDK 动态代理和 CGLIB 代理有什么区别？

   **参考答案**：Spring AOP 默认使用 JDK 动态代理，如果目标对象没有实现接口，则使用 CGLIB 代理。JDK 动态代理基于接口实现，只能代理接口方法；CGLIB 通过继承目标类实现代理，可以代理类的所有非 final 方法。在性能方面，CGLIB 在高频调用场景下通常优于 JDK 代理，但会增加类加载的开销。可以通过设置 spring.aop.proxy-target-class=true 来强制使用 CGLIB 代理。

3. **问题**：Spring MVC 的工作流程是怎样的？

   **参考答案**：Spring MVC 的核心是 DispatcherServlet，它作为前端控制器统一处理所有请求。流程包括：接收请求→查找 HandlerMapping→获取 HandlerAdapter→执行处理器→解析视图→渲染响应。关键组件包括 DispatcherServlet、HandlerMapping、HandlerAdapter、ViewResolver 等。这个流程实现了请求的统一处理和组件的解耦，使得开发者可以专注于业务逻辑的实现。

**高级应用类问题**：



1. **问题**：如何实现 Spring Boot 的自定义 Starter？

   **参考答案**：创建自定义 Starter 需要以下步骤：创建 Maven 项目；添加 spring-boot-autoconfigure 依赖；编写自动配置类，使用 @Configuration 和 @Conditional 注解；在 META-INF/spring.factories 中声明自动配置类；提供必要的属性配置类。Starter 的命名应遵循 spring-boot-starter-\* 的规范，便于识别和使用。

2. **问题**：Spring Boot 的自动配置原理是什么？

   **参考答案**：自动配置基于 @EnableAutoConfiguration 注解，它会触发 SpringFactoriesLoader 加载所有自动配置类。每个自动配置类使用 @Conditional 系列注解来判断是否应该生效。自动配置类通常包含 @Bean 方法，用于注册特定功能的 Bean。开发者可以通过自定义配置来覆盖自动配置的默认值，实现 "约定优于配置" 的设计理念。

3. **问题**：如何实现 Spring 应用的事件驱动架构？

   **参考答案**：Spring 的事件驱动基于 ApplicationEvent 和 ApplicationListener 接口。自定义事件需要继承 ApplicationEvent 类；事件监听器需要实现 ApplicationListener 接口或使用 @EventListener 注解。通过 ApplicationEventPublisher 发布事件。这种机制实现了组件间的松耦合通信，特别适合实现观察者模式、发布订阅模式等。

**性能优化类问题**：



1. **问题**：如何优化 Spring 应用的启动时间？

   **参考答案**：优化启动时间的方法包括：减少 Bean 的数量，只加载必要的 Bean；使用延迟加载（@Lazy）非必需 Bean；优化自动配置，排除不必要的自动配置类；使用类路径扫描优化，减少扫描范围；启用 Spring Boot 的 lazy 初始化模式；使用 AOT 编译（Spring Native）将应用编译为本地可执行文件。

2. **问题**：如何提高 Spring MVC 应用的并发性能？

   **参考答案**：提高并发性能的策略包括：启用异步请求处理，避免阻塞线程；使用响应式编程模型（Spring WebFlux）；优化数据库连接池配置；使用缓存减少数据库访问；实现合理的限流和熔断机制；使用连接池和线程池复用资源；优化垃圾回收策略。

3. **问题**：如何诊断和解决 Spring 应用的内存泄漏问题？

   **参考答案**：诊断内存泄漏需要：使用内存分析工具（如 Eclipse Memory Analyzer）分析堆转储；查找持有强引用的对象；检查静态集合类是否正确清理；分析对象的生命周期，确保及时释放资源。常见的内存泄漏原因包括：未正确关闭的资源（如数据库连接、流等）；注册后未注销的监听器；静态缓存中存储的大量对象等。

**答题技巧总结**：



1. **理解问题核心**：仔细分析面试官的问题，明确考察的知识点和深度要求。

2. **结构化回答**：采用 "总 - 分 - 总" 的方式，先给出总体回答，再详细展开，最后总结要点。

3. **结合实际经验**：在回答中结合具体的项目经验，说明如何在实践中应用相关技术。

4. **避免死记硬背**：理解原理比记忆细节更重要，要能够用自己的语言解释技术概念。

5. **关注最新特性**：了解 Spring 5.x 和 Spring Boot 3.x 的新特性，这些通常是面试的热点。

6. **展示架构思维**：不仅回答 "是什么" 和 "怎么做"，还要说明 "为什么" 和 "如何优化"。

7. **诚实面对不足**：对于不熟悉的知识点，要诚实说明，避免编造答案。

8. **准备反问问题**：准备一些有深度的问题，展示你的思考能力和学习态度。

通过系统学习和充分准备，结合实际项目经验的总结，相信你能够在 Spring 架构师面试中表现出色。记住，面试不仅是技术能力的展示，也是思维方式和学习能力的体现。保持自信，展现真实的技术水平，相信你一定能够获得理想的机会。

**参考资料&#x20;**

\[1] Core Technologies(pdf)[ https://docs.spring.io/spring-framework/docs/5.3.18/reference/pdf/core.pdf](https://docs.spring.io/spring-framework/docs/5.3.18/reference/pdf/core.pdf)

\[2] (二)Spring 核心之控制反转(IoC)—— 体系结构设计及原理详解\_ioc体系结构设计-CSDN博客[ https://blog.csdn.net/mrluo735/article/details/135665427](https://blog.csdn.net/mrluo735/article/details/135665427)

\[3] Spring中BeanFactory与ApplicationContext的核心区别解析[ https://www.iesdouyin.com/share/video/7491571455465934114/?region=\&mid=7491572325003938597\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=UDWfPhnt8GEMMStoDDB9FMNQODOQDrjh9XFwVZ5.oI8-\&share\_version=280700\&ts=1769260967\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7491571455465934114/?region=\&mid=7491572325003938597\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=UDWfPhnt8GEMMStoDDB9FMNQODOQDrjh9XFwVZ5.oI8-\&share_version=280700\&ts=1769260967\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[4] Chapter 3. The IoC container[ https://docs.spring.io/spring-framework/docs/2.5.x/reference/beans.html](https://docs.spring.io/spring-framework/docs/2.5.x/reference/beans.html)

\[5] 源码分析:Spring IOC容器初始化过程这篇文章，我们将通过剖析 Spring 5.x源码，深度分析 Spring - 掘金[ https://juejin.cn/post/7481601410229960739](https://juejin.cn/post/7481601410229960739)

\[6] 深入解析Spring IoC容器:从启动流程到BeanPostProcessor扩展点-腾讯云开发者社区-腾讯云[ https://cloud.tencent.com.cn/developer/article/2560564](https://cloud.tencent.com.cn/developer/article/2560564)

\[7] 深入源码剖析Spring容器启动过程与设计模式-开发者社区-阿里云[ https://developer.aliyun.com/article/1652205](https://developer.aliyun.com/article/1652205)

\[8] 深入理解 Spring IOC 容器:架构设计、核心机制与实战优化-CSDN博客[ https://blog.csdn.net/zqmgx13291/article/details/150005449](https://blog.csdn.net/zqmgx13291/article/details/150005449)

\[9] Core Technologies[ https://docs.spring.io/spring-framework/docs/5.3.6/reference/html/core.html](https://docs.spring.io/spring-framework/docs/5.3.6/reference/html/core.html)

\[10] The IoC Container[ https://docs.spring.io/spring-framework/reference/6.0/core/beans.html](https://docs.spring.io/spring-framework/reference/6.0/core/beans.html)

\[11] 手把手带你吃透Spring Bean生命周期!从实例化到销毁全流程解析\_springbean什么时候实例化-CSDN博客[ https://blog.csdn.net/weixin\_42148384/article/details/148770132](https://blog.csdn.net/weixin_42148384/article/details/148770132)

\[12] Spring Bean 完整生命周期-CSDN博客[ https://blog.csdn.net/m0\_73077921/article/details/153744858](https://blog.csdn.net/m0_73077921/article/details/153744858)

\[13] Spring Bean 生命周期详解-CSDN博客[ https://blog.csdn.net/u011265143/article/details/155617376](https://blog.csdn.net/u011265143/article/details/155617376)

\[14] Spring核心机制解析：Bean生命周期、MVC流程与自动配置原理[ https://www.iesdouyin.com/share/video/7547681594941754659/?region=\&mid=7547681974123891508\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=mTXIZ5vyPOx02yEFuUStsseUP.0rSP9EsGl7p6uITaQ-\&share\_version=280700\&ts=1769260967\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7547681594941754659/?region=\&mid=7547681974123891508\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=mTXIZ5vyPOx02yEFuUStsseUP.0rSP9EsGl7p6uITaQ-\&share_version=280700\&ts=1769260967\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[15] 深入剖析 Spring Bean 的生命周期:从诞生到销毁的完整旅程\_spirng bean的生命周期-CSDN博客[ https://blog.csdn.net/cyf123\_\_/article/details/150447650](https://blog.csdn.net/cyf123__/article/details/150447650)

\[16] spring bean生命周期-腾讯云开发者社区-腾讯云[ https://cloud.tencent.cn/developer/article/2561170](https://cloud.tencent.cn/developer/article/2561170)

\[17] 通俗易懂的讲解SpringBean生命周期\_51CTO博客\_springbean生命周期详解[ https://blog.51cto.com/liaozhiweiblog/14201840](https://blog.51cto.com/liaozhiweiblog/14201840)

\[18] Spring 循环依赖问题原理-CSDN博客[ https://blog.csdn.net/2403\_88625492/article/details/153690120](https://blog.csdn.net/2403_88625492/article/details/153690120)

\[19] Spring 循环 依赖 三级 缓存 原理 ， 一 口气 讲 清楚 ！ # Java # java 面试 # java 程序员 # 程序员 # 抖音 知识 年终 大 赏[ https://www.iesdouyin.com/share/video/7590624683045719348/?region=\&mid=7590624816974072618\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=EhXHQ71xHTfbhom9lBQaTXQImPAvVGDhGyNkYg9ybAg-\&share\_version=280700\&ts=1769260986\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7590624683045719348/?region=\&mid=7590624816974072618\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=EhXHQ71xHTfbhom9lBQaTXQImPAvVGDhGyNkYg9ybAg-\&share_version=280700\&ts=1769260986\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[20] Spring 中的循环依赖与三级缓存机制\_springbean的依赖注入与三级缓存-CSDN博客[ https://blog.csdn.net/weixin\_43598533/article/details/147937146](https://blog.csdn.net/weixin_43598533/article/details/147937146)

\[21] Spring如何解决循环依赖问题。\_spring三级缓存如何解决循环依赖-CSDN博客[ https://blog.csdn.net/weixin\_43521001/article/details/148685461](https://blog.csdn.net/weixin_43521001/article/details/148685461)

\[22] 深入浅出:Spring三级缓存如何巧妙化解Bean循环依赖?-CSDN博客[ https://blog.csdn.net/yiridancan/article/details/148850486](https://blog.csdn.net/yiridancan/article/details/148850486)

\[23] Spring如何解决循环依赖:深入剖析三级缓存机制\_循环依赖解决方案(三级缓存机制)-CSDN博客[ https://blog.csdn.net/qq\_16242613/article/details/146992861](https://blog.csdn.net/qq_16242613/article/details/146992861)

\[24] Spring 三级缓存解决循环依赖深度剖析\_getsingleton singletonlock-CSDN博客[ https://blog.csdn.net/qq\_45750561/article/details/145735468](https://blog.csdn.net/qq_45750561/article/details/145735468)

\[25] Spring循环依赖三级缓存如此easy!!-CSDN博客[ https://blog.csdn.net/2301\_81675556/article/details/146494774](https://blog.csdn.net/2301_81675556/article/details/146494774)

\[26] Spring 是怎么解决循环依赖的?-CSDN博客[ https://blog.csdn.net/2401\_87189717/article/details/149151159](https://blog.csdn.net/2401_87189717/article/details/149151159)

\[27] Spring如何解决Bean的循环依赖-CSDN博客[ https://blog.csdn.net/m0\_64630668/article/details/150533339](https://blog.csdn.net/m0_64630668/article/details/150533339)

\[28] Spring循环依赖解决方案：三级缓存机制解析[ https://www.iesdouyin.com/share/video/7589097284321026481/?region=\&mid=7589097247170743076\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=dxNwEVTUZJhVLGPbT7MYf\_dkbSVGovvgIEqlEk2kXQ8-\&share\_version=280700\&ts=1769260986\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7589097284321026481/?region=\&mid=7589097247170743076\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=dxNwEVTUZJhVLGPbT7MYf_dkbSVGovvgIEqlEk2kXQ8-\&share_version=280700\&ts=1769260986\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[29] 【Spring】Spring是如何解决循环依赖问题的\_spring怎么解决循环依赖-CSDN博客[ https://blog.csdn.net/weixin\_45325628/article/details/146216031](https://blog.csdn.net/weixin_45325628/article/details/146216031)

\[30] Spring循环依赖的三级缓存?图解+源码彻底讲透\_编程我最懂[ http://m.toutiao.com/group/7488546569550037513/?upstream\_biz=doubao](http://m.toutiao.com/group/7488546569550037513/?upstream_biz=doubao)

\[31] ​Spring 解决循环依赖的核心机制\_spring 三级缓存获取a的objectfactory-CSDN博客[ https://blog.csdn.net/weixin\_73344672/article/details/146469739](https://blog.csdn.net/weixin_73344672/article/details/146469739)

\[32] 面试官:“什么是BeanDefinition?BeanDefinition中有哪些属性?”​ BeanDefinitio - 掘金[ https://juejin.cn/post/7473299165290381350](https://juejin.cn/post/7473299165290381350)

\[33] Spring-CSDN博客[ https://blog.csdn.net/qq\_45000512/article/details/149825750](https://blog.csdn.net/qq_45000512/article/details/149825750)

\[34] 手写Bean创建过程解析：单例模式与依赖注入[ https://www.iesdouyin.com/share/video/7554311802851446052/?region=\&mid=7554311949115231015\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=bMmX\_aGK44qdkLrDzk66TEJpmA36fyi17U.A6U9jxRI-\&share\_version=280700\&ts=1769260986\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7554311802851446052/?region=\&mid=7554311949115231015\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=bMmX_aGK44qdkLrDzk66TEJpmA36fyi17U.A6U9jxRI-\&share_version=280700\&ts=1769260986\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[35] Spring Bean 深度剖析:生命周期、加载机制与作用域的底层实现\_java 静态类中每次获取bean是否消耗服务资源、是否增加程序执行时间-CSDN博客[ https://blog.csdn.net/weixin\_38436441/article/details/149390229](https://blog.csdn.net/weixin_38436441/article/details/149390229)

\[36] Spring源码解析 - Bean的创建详解!\_spring bean 解析过程-CSDN博客[ https://blog.csdn.net/weixin\_66592566/article/details/144920994](https://blog.csdn.net/weixin_66592566/article/details/144920994)

\[37] \[20章完结]Java高手提薪精选--Spring源码解析到手写核心组件/s/1PRGmpJQ2W4I4JyydhChN - 掘金[ https://juejin.cn/post/7493887688829124643](https://juejin.cn/post/7493887688829124643)

\[38] Spring框架中Bean是如何加载的?从底层源码入手，详细解读Bean的创建流程-阿里云开发者社区[ https://developer.aliyun.com/article/1608415](https://developer.aliyun.com/article/1608415)

\[39] 面试官:“Spring中的BeanFactory和FactoryBean的区别是什么?”-CSDN博客[ https://blog.csdn.net/Like0703/article/details/145719948](https://blog.csdn.net/Like0703/article/details/145719948)

\[40] Spring源码之FactoryBean接口的作用和实现原理\_spring factory接口-CSDN博客[ https://blog.csdn.net/chaochao2113/article/details/127394913](https://blog.csdn.net/chaochao2113/article/details/127394913)

\[41] Spring隐藏技能:FactoryBean的“& “魔法与单例缓存黑科技!在Spring框架的底层设计中，Factory - 掘金[ https://juejin.cn/post/7520514539638505498](https://juejin.cn/post/7520514539638505498)

\[42] 解析Spring中BeanFactory与FactoryBean的区别及应用场景[ https://www.iesdouyin.com/share/video/7438526333380021516/?region=\&mid=7438526342246763290\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=mVo\_mmX2MrqHKIGoDR7tH8dYV.xAMl\_bzdgMY9zGoSQ-\&share\_version=280700\&ts=1769260995\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7438526333380021516/?region=\&mid=7438526342246763290\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=mVo_mmX2MrqHKIGoDR7tH8dYV.xAMl_bzdgMY9zGoSQ-\&share_version=280700\&ts=1769260995\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[43] Spring中FactoryBean接口详解FactoryBean是Spring框架中的一个重要接口，通过实现这个接口， - 掘金[ https://juejin.cn/post/7459299503528738831](https://juejin.cn/post/7459299503528738831)

\[44] Spring 源码解析 - FactoryBean 获取 Bean 过程\_factorybean实现的bean-CSDN博客[ https://blog.csdn.net/qq\_43692950/article/details/131109087](https://blog.csdn.net/qq_43692950/article/details/131109087)

\[45] spring 源码 getObjectForBeanInstance\_mob6454cc71b244的技术博客\_51CTO博客[ https://blog.51cto.com/u\_16099279/13503894](https://blog.51cto.com/u_16099279/13503894)

\[46] BeanFactory和FactoryBean\_bean factory-CSDN博客[ https://blog.csdn.net/weixin\_46425661/article/details/144705005#BeanFactory\_5](https://blog.csdn.net/weixin_46425661/article/details/144705005#BeanFactory_5)

\[47] 解析Spring中BeanFactory与FactoryBean的核心差异[ https://www.iesdouyin.com/share/video/7438416635871907072/?region=\&mid=7438416624799304499\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=x9.7FHZrN6DBhOajA2pGJFlY29Jbxb2MVQj1ZenrtyA-\&share\_version=280700\&ts=1769260995\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7438416635871907072/?region=\&mid=7438416624799304499\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=x9.7FHZrN6DBhOajA2pGJFlY29Jbxb2MVQj1ZenrtyA-\&share_version=280700\&ts=1769260995\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[48] 一文搞懂BeanFactory 和 FactoryBean\_重写factory.beanfactory-CSDN博客[ https://blog.csdn.net/fan521dan/article/details/133886723](https://blog.csdn.net/fan521dan/article/details/133886723)

\[49] 🎭 FactoryBean vs BeanFactory:Spring界的"双胞胎"之谜!🎬 开场白:谁是谁? 嘿， - 掘金[ https://juejin.cn/post/7564047018794713107](https://juejin.cn/post/7564047018794713107)

\[50] Bean Factory与 Factory Bean 的区别核心区别:Bean Factory 是 Spring 的容器 - 掘金[ https://juejin.cn/post/7547425901316210726](https://juejin.cn/post/7547425901316210726)

\[51] BeanFactory 和 FactoryBean 的区别是什么?-CSDN博客[ https://blog.csdn.net/smart\_an/article/details/144922051](https://blog.csdn.net/smart_an/article/details/144922051)

\[52] spring中的“工厂大师”:factorybean深度解析与实战[ http://m.toutiao.com/group/7592897525301756457/?upstream\_biz=doubao](http://m.toutiao.com/group/7592897525301756457/?upstream_biz=doubao)

\[53] 深入剖析Spring FactoryBean-CSDN博客[ https://blog.csdn.net/ZuanShi1111/article/details/150416286](https://blog.csdn.net/ZuanShi1111/article/details/150416286)

\[54] Spring框架中FactoryBean的使用\_org.springframework.beans.factory-CSDN博客[ https://blog.csdn.net/2501\_92540271/article/details/149776079](https://blog.csdn.net/2501_92540271/article/details/149776079)

\[55] 第 15 集 ： 依赖 注入 的 方式 有 哪些 3 动力 节点 Spring 面试 题 视频 教程 # spring 面试 题 # spring 面试 # 动力 节点 # java # java 培训[ https://www.iesdouyin.com/share/video/7525750475317153074/?region=\&mid=7525750538299050802\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=Bt.VQv035XLkfrDCsYTB\_2kDeGo5ImCBQJu1XKGHS0g-\&share\_version=280700\&ts=1769260995\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7525750475317153074/?region=\&mid=7525750538299050802\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=Bt.VQv035XLkfrDCsYTB_2kDeGo5ImCBQJu1XKGHS0g-\&share_version=280700\&ts=1769260995\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[56] Spring FactoryBean 深度解析-CSDN博客[ https://blog.csdn.net/weixin\_51786043/article/details/148202104](https://blog.csdn.net/weixin_51786043/article/details/148202104)

\[57] spring BeanFactory和FactoryBean作用和区别-CSDN博客[ https://blog.csdn.net/tan809417133/article/details/151868848](https://blog.csdn.net/tan809417133/article/details/151868848)

\[58] 深入理解Spring FactoryBean:灵活创建复杂对象的秘密武器引言 在 Spring 框架中，Bean 的创建 - 掘金[ https://juejin.cn/post/7473015463284490292](https://juejin.cn/post/7473015463284490292)

\[59] Day06: 条件装配的奥秘:@Conditional实现原理深度揭秘-CSDN博客[ https://blog.csdn.net/yezi1238/article/details/149164390](https://blog.csdn.net/yezi1238/article/details/149164390)

\[60] 深入 Spring 条件化配置底层:从硬编码到通用注解的实现原理\_硬编码 声明式注解-CSDN博客[ https://blog.csdn.net/weixin\_44289947/article/details/153060560](https://blog.csdn.net/weixin_44289947/article/details/153060560)

\[61] 【 Java 面试 】 @ Conditional 注解 的 原理 是 什么 ？ 实际 用 过 吗 ？&#x20;

&#x20;\# 程序员 # Java 面试 # Java 后端 # 找 工作 # IT[ https://www.iesdouyin.com/share/video/7596160606789438783/?region=\&mid=7596160639836474122\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=zAnutlwYLj7WVc3EvV1ZMskZD4OoPtDV\_Qqoa9SkbrQ-\&share\_version=280700\&ts=1769261008\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7596160606789438783/?region=\&mid=7596160639836474122\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=zAnutlwYLj7WVc3EvV1ZMskZD4OoPtDV_Qqoa9SkbrQ-\&share_version=280700\&ts=1769261008\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[62] Spring框架中@Conditional注解全面解析Spring框架中@Conditional注解全面解析 一、引言 - 掘金[ https://juejin.cn/post/7483704056648237068](https://juejin.cn/post/7483704056648237068)

\[63] 深度解析 Spring @Conditional:实现智能条件化配置的利器-CSDN博客[ https://blog.csdn.net/qq\_33181292/article/details/155532272](https://blog.csdn.net/qq_33181292/article/details/155532272)

\[64] Externalized Configuration :: Spring Boot[ https://docs.spring.io/spring-boot/reference/features/external-config.html](https://docs.spring.io/spring-boot/reference/features/external-config.html)

\[65] 深入剖析 Spring/Spring Boot 配置类加载顺序与原理\_图灵spring类加载原理-CSDN博客[ https://guosy.blog.csdn.net/article/details/152333868](https://guosy.blog.csdn.net/article/details/152333868)

\[66] Spring Boot配置读取的六种方式解析与应用场景[ https://www.iesdouyin.com/share/video/7511990566175968564/?region=\&mid=7511991021035670312\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=LYb9SeCmz3LI3Rzogm8Gm5RRAzyuhh0wMQ5GS2TJNkU-\&share\_version=280700\&ts=1769261008\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7511990566175968564/?region=\&mid=7511991021035670312\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=LYb9SeCmz3LI3Rzogm8Gm5RRAzyuhh0wMQ5GS2TJNkU-\&share_version=280700\&ts=1769261008\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[67] Spring Boot配置文件加载顺序:从原理到实战全解析\_从程序员到架构师[ http://m.toutiao.com/group/7598877763747054134/?upstream\_biz=doubao](http://m.toutiao.com/group/7598877763747054134/?upstream_biz=doubao)

\[68] 从配置到日志:Java / Spring 在云原生时代的配置管理与日志体系\_RendaZhang[ http://m.toutiao.com/group/7576231759542387235/?upstream\_biz=doubao](http://m.toutiao.com/group/7576231759542387235/?upstream_biz=doubao)

\[69] application.yaml加载原理Spring Boot 中 application.yaml 的加载原理涉及环境 - 掘金[ https://juejin.cn/post/7487542928132751423](https://juejin.cn/post/7487542928132751423)

\[70] 环境配置与读取yml\_environment.yml配置环境-CSDN博客[ https://blog.csdn.net/m0\_49054880/article/details/146542182](https://blog.csdn.net/m0_49054880/article/details/146542182)

\[71] 5. Resources[ https://docs.spring.io/spring-framework/docs/4.0.4.RELEASE/spring-framework-reference/html/resources.html](https://docs.spring.io/spring-framework/docs/4.0.4.RELEASE/spring-framework-reference/html/resources.html)

\[72] Resources[ https://www.spring-doc.cn/spring-framework/6.1.21/core\_resources.en.html](https://www.spring-doc.cn/spring-framework/6.1.21/core_resources.en.html)

\[73] 🗂️ Spring的Resource:代码界的"万能钥匙"!🎬 开场白:文件都藏哪儿了? 嘿，小伙伴们!👋 今天我 - 掘金[ https://juejin.cn/post/7564235452455157801](https://juejin.cn/post/7564235452455157801)

\[74] 第 23 集 ： 依赖 注入 的 方式 有 哪些 \_ 扩展 问题 1 动力 节点 Spring 面试 题 视频 教程 # spring 面试 题 # spring 面试 # 动力 节点 # java # java 培训[ https://www.iesdouyin.com/share/video/7530186750912646419/?region=\&mid=7530186828882266926\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=PuWlBtsFdvRIMzcSzBsXBgAommt1acGT2kiCZwGqz30-\&share\_version=280700\&ts=1769261008\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7530186750912646419/?region=\&mid=7530186828882266926\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=PuWlBtsFdvRIMzcSzBsXBgAommt1acGT2kiCZwGqz30-\&share_version=280700\&ts=1769261008\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[75] Spring Framework资源加载优先级:Classpath与文件系统策略-CSDN博客[ https://blog.csdn.net/gitblog\_00979/article/details/151337702](https://blog.csdn.net/gitblog_00979/article/details/151337702)

\[76] bestJavaer/spring/spring-resource.md at master · crisxuan/bestJavaer · GitHub[ https://github.com/crisxuan/bestJavaer/blob/master/spring/spring-resource.md](https://github.com/crisxuan/bestJavaer/blob/master/spring/spring-resource.md)

\[77] 资源 (Resources) | Spring Framework6.1.14-SNAPSHOT中文文档|Spring官方文档|SpringBoot 教程|Spring中文网[ https://www.spring-doc.cn/spring-framework/6.1.14-SNAPSHOT/core\_resources.html](https://www.spring-doc.cn/spring-framework/6.1.14-SNAPSHOT/core_resources.html)

\[78] BeanPostProcessor 详解:Spring Bean 生命周期的核心扩展点\_搬梯子摘星星[ http://m.toutiao.com/group/7592874726532383267/?upstream\_biz=doubao](http://m.toutiao.com/group/7592874726532383267/?upstream_biz=doubao)

\[79] Spring Bean 完整生命周期-CSDN博客[ https://blog.csdn.net/m0\_73077921/article/details/153744858](https://blog.csdn.net/m0_73077921/article/details/153744858)

\[80] Spring前置处理器和后置处理器执行顺序如何?\_编程语言-CSDN问答[ https://ask.csdn.net/questions/8909562](https://ask.csdn.net/questions/8909562)

\[81] Spring核心机制解析：Bean生命周期、MVC流程与自动配置原理[ https://www.iesdouyin.com/share/video/7547681594941754659/?region=\&mid=7547681974123891508\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=mTXIZ5vyPOx02yEFuUStsseUP.0rSP9EsGl7p6uITaQ-\&share\_version=280700\&ts=1769261030\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7547681594941754659/?region=\&mid=7547681974123891508\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=mTXIZ5vyPOx02yEFuUStsseUP.0rSP9EsGl7p6uITaQ-\&share_version=280700\&ts=1769261030\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[82] Spring框架中的Bean后置处理器:IoC容器强大扩展性的幕后英雄-腾讯云开发者社区-腾讯云[ https://cloud.tencent.com.cn/developer/article/2560668?frompage=seopage\&policyId=undefined](https://cloud.tencent.com.cn/developer/article/2560668?frompage=seopage\&policyId=undefined)

\[83] 【Spring】Bean的生命周期-CSDN博客[ https://blog.csdn.net/qq\_36634055/article/details/142848337](https://blog.csdn.net/qq_36634055/article/details/142848337)

\[84] Spring 中最重要的扩展点之 BeanPostProcessor详解\_奔跑的蜗牛[ http://m.toutiao.com/group/7570741517300613641/?upstream\_biz=doubao](http://m.toutiao.com/group/7570741517300613641/?upstream_biz=doubao)

\[85] Spring中BeanFactoryPostProcessor和BeanPostProcessor的区别\_beanfactorypostprocessor作用-CSDN博客[ https://blog.csdn.net/weixin\_37522117/article/details/146333248](https://blog.csdn.net/weixin_37522117/article/details/146333248)

\[86] 一文看懂 BeanFactoryPostProcessor 与 BeanPostProcessor 接口\_beanpostprocessor和beanfactorypostprocessor-CSDN博客[ https://blog.csdn.net/weixin\_46425661/article/details/144420284](https://blog.csdn.net/weixin_46425661/article/details/144420284)

\[87] Spring的发布处理器(BeanPostProcessor)\_spring 发布bean-CSDN博客[ https://blog.csdn.net/lzghxjt/article/details/51870186](https://blog.csdn.net/lzghxjt/article/details/51870186)

\[88] 解析BeanFactoryPostProcessor的核心作用与Spring源码机制[ https://www.iesdouyin.com/share/video/7521726987006790927/?region=\&mid=7521727301269179163\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=ByDeY6FfkSp8PzKwYXjLUz\_cUqiHyk.rmyiy\_qwg2Mc-\&share\_version=280700\&ts=1769261029\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7521726987006790927/?region=\&mid=7521727301269179163\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=ByDeY6FfkSp8PzKwYXjLUz_cUqiHyk.rmyiy_qwg2Mc-\&share_version=280700\&ts=1769261029\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[89] BeanPostProcessor 和 BeanFactoryPostProcessor的作用，能实现什么功能 - CSDN文库[ https://wenku.csdn.net/answer/1tpea9gpb2](https://wenku.csdn.net/answer/1tpea9gpb2)

\[90] Spring - Bean的实例化流程及生命周期\_beandefinition创建出普通对象流程-CSDN博客[ https://blog.csdn.net/weixin\_51351637/article/details/131087131](https://blog.csdn.net/weixin_51351637/article/details/131087131)

\[91] Spring Framework后处理器:BeanPostProcessor与BeanFactoryPostProcessor-CSDN博客[ https://blog.csdn.net/gitblog\_00628/article/details/152592673](https://blog.csdn.net/gitblog_00628/article/details/152592673)

\[92] 深度解析Spring事件机制:基于观察者模式的解耦利器-CSDN博客[ https://blog.csdn.net/zuiyuelong/article/details/150165155](https://blog.csdn.net/zuiyuelong/article/details/150165155)

\[93] 深入理解 Spring 的事件驱动模型:构建松耦合、可扩展的 Java 应用\_java事件驱动模型应用-CSDN博客[ https://yishuo.blog.csdn.net/article/details/154937646](https://yishuo.blog.csdn.net/article/details/154937646)

\[94] 深入浅出 Spring Event:原理剖析与实战指南-CSDN博客[ https://blog.csdn.net/Crayon26/article/details/156397959](https://blog.csdn.net/Crayon26/article/details/156397959)

\[95] Spring 框架——事件驱动模型\_spring事件驱动模型-CSDN博客[ https://blog.csdn.net/weixin\_43004044/article/details/131754324](https://blog.csdn.net/weixin_43004044/article/details/131754324)

\[96] Spring 事件监听机制介绍以及源码分析-CSDN博客[ https://blog.csdn.net/weixin\_46619605/article/details/146566012](https://blog.csdn.net/weixin_46619605/article/details/146566012)

\[97] Spring 订阅发布模式(事件驱动模型)Spring 订阅发布模式(事件驱动模型) Spring 订阅发布模式基于 - 掘金[ https://juejin.cn/post/7569072953901334579](https://juejin.cn/post/7569072953901334579)

\[98] spring的事件驱动有时候比消息队列好用-腾讯云开发者社区-腾讯云[ https://cloud.tencent.com.cn/developer/article/2526812](https://cloud.tencent.com.cn/developer/article/2526812)

\[99] aop---深入理解核心概念 → 代理原理 → 通知类型与执行顺序 → 切点表达式(execution/annotati - 掘金[ https://juejin.cn/post/7590608345923272730](https://juejin.cn/post/7590608345923272730)

\[100] Spring aop 五种通知类型-CSDN博客[ https://blog.csdn.net/2509\_94228653/article/details/156811599](https://blog.csdn.net/2509_94228653/article/details/156811599)

\[101] Spring AOP 核心知识笔记Spring AOP 核心知识笔记 一、AOP 核心思想与实现原理 AOP(面向切面编 - 掘金[ https://juejin.cn/post/7582425536463699994](https://juejin.cn/post/7582425536463699994)

\[102] Spring AOP通知类型及执行顺序解析[ https://www.iesdouyin.com/share/video/6940661185809288489/?region=\&mid=6940661216373132069\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=QntMBeZFvm2vO\_xByeNQsidYwSra3532QSMKaINA\_rM-\&share\_version=280700\&ts=1769261066\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/6940661185809288489/?region=\&mid=6940661216373132069\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=QntMBeZFvm2vO_xByeNQsidYwSra3532QSMKaINA_rM-\&share_version=280700\&ts=1769261066\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[103] SpringAOP通知细节-避坑指南\_博观而约取，厚积而薄发的技术博客\_mob6454cc659b12的技术博客\_51CTO博客[ https://blog.51cto.com/u\_16099194/14325738](https://blog.51cto.com/u_16099194/14325738)

\[104] 一文读懂Spring AOP:从原理到实战，再也不怕面试问-CSDN博客[ https://blog.csdn.net/2303\_79998710/article/details/155392437](https://blog.csdn.net/2303_79998710/article/details/155392437)

\[105] spring AOP的复习和5中通知的执行顺序\_前置、环绕、最终、异常通知,返回5种通知的执行顺序-CSDN博客[ https://blog.csdn.net/jaryle/article/details/72910929](https://blog.csdn.net/jaryle/article/details/72910929)

\[106] Spring AOP:JDK与CGLIB代理机制解析\_spring选择cglib和jdk动态代理的代码逻辑分析-CSDN博客[ https://blog.csdn.net/2301\_80141552/article/details/150989549](https://blog.csdn.net/2301_80141552/article/details/150989549)

\[107] 【Spring】AOP深度解析:代理机制、拦截器链与事务失效全解-CSDN博客[ https://blog.csdn.net/Txx318026/article/details/156722083](https://blog.csdn.net/Txx318026/article/details/156722083)

\[108] 完整教程:Spring Boot 机制四: AOP 代理机制源码级深度解析(JDK / CGLIB 全链路)\_51CTO博客\_spring boot aop原理[ https://blog.51cto.com/u\_15469972/14437262](https://blog.51cto.com/u_15469972/14437262)

\[109] Spring AOP代理方式选择逻辑及性能对比解析[ https://www.iesdouyin.com/share/note/7534669206277705012/?region=\&mid=7168534822384732167\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&schema\_type=37\&share\_sign=vMTqJ5q21Mz4\_pKVqIjBDb85doJ8VtIVG04Sb7q8xWQ-\&share\_version=280700\&ts=1769261066\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/note/7534669206277705012/?region=\&mid=7168534822384732167\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&schema_type=37\&share_sign=vMTqJ5q21Mz4_pKVqIjBDb85doJ8VtIVG04Sb7q8xWQ-\&share_version=280700\&ts=1769261066\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[110] 深入理解 Spring AOP 代理机制:JDK 动态代理与 CGLIB 的对比与选择\_aop jdk和cglib-CSDN博客[ https://blog.csdn.net/Landcc/article/details/149224101](https://blog.csdn.net/Landcc/article/details/149224101)

\[111] JDK 动态代理 vs CGLIB:原理、区别与 Spring AOP 底层揭秘本文深入解析 JDK 动态代理与 CGL - 掘金[ https://juejin.cn/post/7561442148370137139](https://juejin.cn/post/7561442148370137139)

\[112] spring AOP 之四:@AspectJ切入点标识符语法详解-CSDN博客[ https://blog.csdn.net/weixin\_30449239/article/details/99446198](https://blog.csdn.net/weixin_30449239/article/details/99446198)

\[113] 在Spring框架中使用AspectJ实现AOP(面向切面编程)\_spring aspectj-CSDN博客[ https://blog.csdn.net/Liu\_Downloads/article/details/146905252](https://blog.csdn.net/Liu_Downloads/article/details/146905252)

\[114] 深度解析切入点表达式 及 详细代码展示\_切入点表达式源码怎么看-CSDN博客[ https://blog.csdn.net/csdn\_tom\_168/article/details/148921168](https://blog.csdn.net/csdn_tom_168/article/details/148921168)

\[115] Spring AOP切点表达式核心用法与应用技巧解析[ https://www.iesdouyin.com/share/video/7589646713829330217/?region=\&mid=7589646909789588262\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=UFK6FylKLD3rO8cBexCSksN0eVuHSigkAr7EGFz956c-\&share\_version=280700\&ts=1769261065\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7589646713829330217/?region=\&mid=7589646909789588262\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=UFK6FylKLD3rO8cBexCSksN0eVuHSigkAr7EGFz956c-\&share_version=280700\&ts=1769261065\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[116] Spring-AOP @AspectJ语法基础-阿里云开发者社区[ https://developer.aliyun.com/article/1408845](https://developer.aliyun.com/article/1408845)

\[117] Spring AOP 中如何定义切点?-JavaScript中文网-JavaScript教程资源分享门户[ https://www.javascriptcn.com/interview-spring/677f4e6d828d7b20c6314504.html](https://www.javascriptcn.com/interview-spring/677f4e6d828d7b20c6314504.html)

\[118] 分析 Spring 的 AOP 实现原理与切点表达式​ #创作者训练营话题并加 #AI编程 目录 一、AOP 核心概念 - 掘金[ https://juejin.cn/post/7509042958204420133](https://juejin.cn/post/7509042958204420133)

\[119] Spring 事务核心原理 深度解析-CSDN博客[ https://blog.csdn.net/xiaolyuh123/article/details/156830720](https://blog.csdn.net/xiaolyuh123/article/details/156830720)

\[120] @Transactional的底层原理@Transactional 是 Spring 框架中用于声明式事务管理的注解，其 - 掘金[ https://juejin.cn/post/7528576810286071848](https://juejin.cn/post/7528576810286071848)

\[121] \`@Transactional\` 注解深度解析:Spring 声明式事务的核心\_org.springframework.transaction.annotation.transac-CSDN博客[ https://blog.csdn.net/csdn\_tom\_168/article/details/148925487](https://blog.csdn.net/csdn_tom_168/article/details/148925487)

\[122] Spring事务管理核心原理与实现机制解析[ https://www.iesdouyin.com/share/note/7490466122802793768/?region=\&mid=7043672073613936641\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&schema\_type=37\&share\_sign=.Fm4rF226r8DefdwU8LoIsu3gQuetLbhVZZl9sL38fA-\&share\_version=280700\&ts=1769261080\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/note/7490466122802793768/?region=\&mid=7043672073613936641\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&schema_type=37\&share_sign=.Fm4rF226r8DefdwU8LoIsu3gQuetLbhVZZl9sL38fA-\&share_version=280700\&ts=1769261080\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[123] Spring 声明式事务:原理、使用及失效场景详解-CSDN博客[ https://blog.csdn.net/2201\_75355172/article/details/156796590](https://blog.csdn.net/2201_75355172/article/details/156796590)

\[124] Spring 事务管理全解析:原理、源码与实战\_spring 事务原理解析-CSDN博客[ https://blog.csdn.net/Prince140678/article/details/146466434](https://blog.csdn.net/Prince140678/article/details/146466434)

\[125] Spring事务原理Spring声明式事务管理的原理基于 AOP(面向切面编程) 和 事务管理器(PlatformTra - 掘金[ https://juejin.cn/post/7474823756672450587](https://juejin.cn/post/7474823756672450587)

\[126] MySQL Spring 事务全家桶:事务本质、隔离级别、传播机制与失效场景全解析-CSDN博客[ https://blog.csdn.net/z55947810/article/details/155201266](https://blog.csdn.net/z55947810/article/details/155201266)

\[127] 别再踩坑!Spring事务@Transactional失效?一文读懂参数与8大失效场景-CSDN博客[ https://blog.csdn.net/lhmyy521125/article/details/150565066](https://blog.csdn.net/lhmyy521125/article/details/150565066)

\[128] @Transactional详解(作用、失效场景与解决方法)-CSDN博客[ https://blog.csdn.net/qq\_57581439/article/details/132086303](https://blog.csdn.net/qq_57581439/article/details/132086303)

\[129] 解析@Transactional失效的常见场景与原因[ https://www.iesdouyin.com/share/video/7429329879021161779/?region=\&mid=7429330001905945344\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=q9L1AXWc2g2ubX8o61pXDeLTFJ.q8y7S3lG.8WMHrl0-\&share\_version=280700\&ts=1769261080\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7429329879021161779/?region=\&mid=7429330001905945344\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=q9L1AXWc2g2ubX8o61pXDeLTFJ.q8y7S3lG.8WMHrl0-\&share_version=280700\&ts=1769261080\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[130] 【spring】@Transactional 注解失效的原因及解决办法\_transactional 失效-CSDN博客[ https://blog.csdn.net/m0\_74823611/article/details/144339267](https://blog.csdn.net/m0_74823611/article/details/144339267)

\[131] @Transactional注解使用及事务失效场景详解\_聪蛋U三国[ http://m.toutiao.com/group/7490395925387510287/?upstream\_biz=doubao](http://m.toutiao.com/group/7490395925387510287/?upstream_biz=doubao)

\[132] 事物注解 @Transactional的 13个失效场景 | 程序员小富[ http://www.xiaofucode.com/guide/springboot101/base/%E4%BA%8B%E7%89%A9%E6%B3%A8%E8%A7%A3%20@Transactional%E7%9A%84%2013%E4%B8%AA%E5%A4%B1%E6%95%88%E5%9C%BA%E6%99%AF.html](http://www.xiaofucode.com/guide/springboot101/base/%E4%BA%8B%E7%89%A9%E6%B3%A8%E8%A7%A3%20@Transactional%E7%9A%84%2013%E4%B8%AA%E5%A4%B1%E6%95%88%E5%9C%BA%E6%99%AF.html)

\[133] Spring MVC 数据验证与类型转换深度解析-CSDN博客[ https://blog.csdn.net/python15397/article/details/153139481](https://blog.csdn.net/python15397/article/details/153139481)

\[134] Java Bean Validation[ https://docs.spring.io/spring-framework/reference/6.0/core/validation/beanvalidation.html](https://docs.spring.io/spring-framework/reference/6.0/core/validation/beanvalidation.html)

\[135] 数据校验新范式:Spring自定义校验注解全攻略-CSDN博客[ https://blog.csdn.net/gitblog\_00102/article/details/151282299](https://blog.csdn.net/gitblog_00102/article/details/151282299)

\[136] 使用自定义约束注解实现接口参数优雅校验[ https://www.iesdouyin.com/share/video/7550920913504439592/?region=\&mid=7550921291646192393\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=bD1NG5343RPDEwMIGjxBxTj8Bf1uedQx6O0RDLzSgmw-\&share\_version=280700\&ts=1769261080\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7550920913504439592/?region=\&mid=7550921291646192393\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=bD1NG5343RPDEwMIGjxBxTj8Bf1uedQx6O0RDLzSgmw-\&share_version=280700\&ts=1769261080\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[137] Spring Validation中9个数据校验工具Spring Validation作为Spring生态系统的重要组成 - 掘金[ https://juejin.cn/post/7502248648074526772](https://juejin.cn/post/7502248648074526772)

\[138] 还在用 if 硬刚参数校验?这波操作土到掉渣!SpringBoot 高阶玩法直接封神-51CTO.COM[ https://www.51cto.com/article/814437.html](https://www.51cto.com/article/814437.html)

\[139] Springboot 数据校验\_张一雄的技术博客\_51CTO博客[ https://blog.51cto.com/xiongod/14268519](https://blog.51cto.com/xiongod/14268519)

\[140] Spring MVC-CSDN博客[ https://blog.csdn.net/xiake1573/article/details/145406538](https://blog.csdn.net/xiake1573/article/details/145406538)

\[141] 讲讲Spring MVC的处理流程-CSDN博客[ https://blog.csdn.net/gdpu2400502251/article/details/157095160](https://blog.csdn.net/gdpu2400502251/article/details/157095160)

\[142] SpringMVC的执行流程\_springmvc执行流程-CSDN博客[ https://blog.csdn.net/weixin\_73739493/article/details/151051268](https://blog.csdn.net/weixin_73739493/article/details/151051268)

\[143] Spring MVC核心流程解析：DispatcherServlet的请求处理机制[ https://www.iesdouyin.com/share/video/7514735876019064104/?region=\&mid=7514737312341805835\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=MgyZrG5JrxvekX5nteTVn3uwuLEj5Mo3zulFXgQwWYA-\&share\_version=280700\&ts=1769261096\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7514735876019064104/?region=\&mid=7514737312341805835\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=MgyZrG5JrxvekX5nteTVn3uwuLEj5Mo3zulFXgQwWYA-\&share_version=280700\&ts=1769261096\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[144] Spring MVC 核心枢纽:DispatcherServlet 的深度解析与实践价值-CSDN博客[ https://blog.csdn.net/qq\_52331401/article/details/148563845](https://blog.csdn.net/qq_52331401/article/details/148563845)

\[145] 详细介绍:Spring MVC 请求执行流程详解\_mob64ca13fdd43c的技术博客\_51CTO博客[ https://blog.51cto.com/u\_16213606/14234261](https://blog.51cto.com/u_16213606/14234261)

\[146] Spring MVC 中 \`DispatcherServlet\` 处理请求的完整流程\_spring mvc 的 dispatcherservlet 工作流程?-CSDN博客[ https://blog.csdn.net/csdn\_tom\_168/article/details/148585823](https://blog.csdn.net/csdn_tom_168/article/details/148585823)

\[147] SpringMVC核心原理与实战全解析-CSDN博客[ https://blog.csdn.net/2401\_83675559/article/details/156147464](https://blog.csdn.net/2401_83675559/article/details/156147464)

\[148] Spring MVC核心流程深度解析:从请求到响应的完美掌控-CSDN博客[ https://blog.csdn.net/qq\_44870477/article/details/157137210](https://blog.csdn.net/qq_44870477/article/details/157137210)

\[149] SpringBoot全局异常处理:三大核心方案深度解析与选型-CSDN博客[ https://blog.csdn.net/m0\_46091798/article/details/157258114](https://blog.csdn.net/m0_46091798/article/details/157258114)

\[150] SpringMVC 开发避坑指南:十大常见问题深度解析与解决方案\_spring mvc 异常详情没有抛出-CSDN博客[ https://blog.csdn.net/qq\_35766758/article/details/148744777](https://blog.csdn.net/qq_35766758/article/details/148744777)

\[151] SpringMvc常见问题-CSDN博客[ https://blog.csdn.net/m0\_74891778/article/details/151427694](https://blog.csdn.net/m0_74891778/article/details/151427694)

\[152] 详解@ControllerAdvice:Spring MVC全局统一处理的终极解决方案\_搬梯子摘星星[ http://m.toutiao.com/group/7595053131176722994/?upstream\_biz=doubao](http://m.toutiao.com/group/7595053131176722994/?upstream_biz=doubao)

\[153] Spring MVC中的拦截器和Servlet中的filter(过滤器)有什么区别?\_1、spring mvc中的拦截器和servlet中的filter有什么区别?-CSDN博客[ https://blog.csdn.net/darling\_cats/article/details/142108333](https://blog.csdn.net/darling_cats/article/details/142108333)

\[154] 拦截器和过滤器的区别-CSDN博客[ https://blog.csdn.net/2303\_78263863/article/details/146186380](https://blog.csdn.net/2303_78263863/article/details/146186380)

\[155] 38-Springmvc拦截器概述以及简单实现(和过滤器的区别)\_mvc拦截器能作用到其他模块吗-CSDN博客[ https://blog.csdn.net/qq\_47128897/article/details/131465244](https://blog.csdn.net/qq_47128897/article/details/131465244)

\[156] 过滤器与拦截器的核心区别对比分析[ https://www.iesdouyin.com/share/video/7482798033780886823/?region=\&mid=7468608251074824208\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=ZOx4tNvC9kQDeImzqXWkDCeuFHJrBuhBzb3Tlyxeau4-\&share\_version=280700\&ts=1769261096\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7482798033780886823/?region=\&mid=7468608251074824208\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=ZOx4tNvC9kQDeImzqXWkDCeuFHJrBuhBzb3Tlyxeau4-\&share_version=280700\&ts=1769261096\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[157] 过滤器和拦截器的区别?-CSDN博客[ https://blog.csdn.net/weixin\_43249298/article/details/150987620](https://blog.csdn.net/weixin_43249298/article/details/150987620)

\[158] Spring MVC 拦截器与过滤器深度解析\_springmvc过滤器和拦截器的区别-CSDN博客[ https://blog.csdn.net/shine19/article/details/147165713](https://blog.csdn.net/shine19/article/details/147165713)

\[159] SpringMVC拦截器(Interceptor)与Servlet过滤器(Filter)区别\_springmvc拦截器和过滤器的区别-CSDN博客[ https://blog.csdn.net/2501\_90417743/article/details/145917826](https://blog.csdn.net/2501_90417743/article/details/145917826)

\[160] Spring Boot自动配置:原理、利弊与实践指南-CSDN博客[ https://blog.csdn.net/weixin\_44289947/article/details/152934791](https://blog.csdn.net/weixin_44289947/article/details/152934791)

\[161] Spring Boot 核心原理解析与实践(含代码示例)-阿里云开发者社区[ https://developer.aliyun.com:443/article/1689530](https://developer.aliyun.com:443/article/1689530)

\[162] 解密 Spring Boot 自动配置:原理、流程与核心组件协同-CSDN博客[ https://blog.csdn.net/yzyyishi/article/details/150590104](https://blog.csdn.net/yzyyishi/article/details/150590104)

\[163] 解析Spring Boot自动配置原理及核心流程[ https://www.iesdouyin.com/share/video/7521646695231261995/?region=\&mid=7521646742656437055\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=aEttfIuhoXE0nfBfbatqGrVGlTrR6k35rN94kVHqqwo-\&share\_version=280700\&ts=1769261127\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7521646695231261995/?region=\&mid=7521646742656437055\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=aEttfIuhoXE0nfBfbatqGrVGlTrR6k35rN94kVHqqwo-\&share_version=280700\&ts=1769261127\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[164] SpringBoot自动装配原理详解附带图解-CSDN博客[ https://blog.csdn.net/qq\_65138851/article/details/148332237](https://blog.csdn.net/qq_65138851/article/details/148332237)

\[165] springboot的自动配置原理\_spring.datasource.enabled-CSDN博客[ https://blog.csdn.net/qq\_56851614/article/details/146333065](https://blog.csdn.net/qq_56851614/article/details/146333065)

\[166] springboot(三)springboot的自动配置原理[ https://blog.51cto.com/u\_16213627/14365236](https://blog.51cto.com/u_16213627/14365236)

\[167] Externalized Configuration[ https://docs.spring.io/spring-boot/3.4/reference/features/external-config.html](https://docs.spring.io/spring-boot/3.4/reference/features/external-config.html)

\[168] 深度解析 Spring Boot 配置机制深度解析 Spring Boot 配置机制 优先级、Profile 与外部化配 - 掘金[ https://juejin.cn/post/7486170504270741530](https://juejin.cn/post/7486170504270741530)

\[169] Spring Boot高频面试真题解析：配置加载顺序及优先级[ https://www.iesdouyin.com/share/video/7501862246765022524/?region=\&mid=7501862088302725907\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=AR3Ic.2\_Ax03BPgIBcSa5rlRPz5yyEUcfPHwvQKvMc8-\&share\_version=280700\&ts=1769261127\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7501862246765022524/?region=\&mid=7501862088302725907\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=AR3Ic.2_Ax03BPgIBcSa5rlRPz5yyEUcfPHwvQKvMc8-\&share_version=280700\&ts=1769261127\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[170] SpringBoot读取配置按什么优先级顺序?\_springboot配置文件优先级-CSDN博客[ https://blog.csdn.net/ma\_nong33/article/details/140558777](https://blog.csdn.net/ma_nong33/article/details/140558777)

\[171] springboot的外部配置加载顺序\_springboot外部配置加载顺序-CSDN博客[ https://blog.csdn.net/m0\_46580983/article/details/146154041](https://blog.csdn.net/m0_46580983/article/details/146154041)

\[172] SpringBoot读取配置优先级顺序是什么?-腾讯云开发者社区-腾讯云[ https://cloud.tencent.cn/developer/article/2491457?frompage=seopage\&policyId=20240000\&traceId=01k1a51b2jjzxqd577052azznc](https://cloud.tencent.cn/developer/article/2491457?frompage=seopage\&policyId=20240000\&traceId=01k1a51b2jjzxqd577052azznc)

\[173] Embedded Web Servers[ https://docs.spring.io/spring-boot/3.3/how-to/webserver.html](https://docs.spring.io/spring-boot/3.3/how-to/webserver.html)

\[174] 嵌入式 Web 服务器 (Embedded Web Servers) | Spring Boot3.3.1中文文档|Spring官方文档|SpringBoot 教程|Spring中文网[ https://www.spring-doc.cn/spring-boot/3.3.1/how-to\_webserver.html](https://www.spring-doc.cn/spring-boot/3.3.1/how-to_webserver.html)

\[175] Spring Boot 支持的内嵌服务器(Tomcat、Jetty、Undertow、Netty(用于 WebFlux 响应式应用))详解\_springboot tomcat jetty undertow netty-CSDN博客[ https://blog.csdn.net/zp357252539/article/details/147181807](https://blog.csdn.net/zp357252539/article/details/147181807)

\[176] Spring Boot内置Tomcat启动原理解析[ https://www.iesdouyin.com/share/video/7499012255057448192/?region=\&mid=7499012742410504999\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=VfVc6VxrtAKug1yn9BhaNrpGxqYVGvVIkkPrNNBbVlY-\&share\_version=280700\&ts=1769261127\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7499012255057448192/?region=\&mid=7499012742410504999\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=VfVc6VxrtAKug1yn9BhaNrpGxqYVGvVIkkPrNNBbVlY-\&share_version=280700\&ts=1769261127\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[177] Spring Boot内嵌服务器全解析:Tomcat vs Jetty vs Undertow 选型指南\_springboot tomcat undertow-CSDN博客[ https://blog.csdn.net/qq479850581/article/details/147049836](https://blog.csdn.net/qq479850581/article/details/147049836)

\[178] 深入理解 Spring Boot 嵌入式 Web 容器:从原理到性能调优\_嵌入式容器技术、原生容器、实时容器技术-CSDN博客[ https://blog.csdn.net/haohaizi\_liu/article/details/154063534](https://blog.csdn.net/haohaizi_liu/article/details/154063534)

\[179] 【Spring Boot 支持哪些内嵌的 Web 服务器详细分析】\_springboot web服务器-CSDN博客[ https://blog.csdn.net/qq\_33545255/article/details/144545176](https://blog.csdn.net/qq_33545255/article/details/144545176)

\[180] 深度解析Spring事件机制:基于观察者模式的解耦利器-CSDN博客[ https://blog.csdn.net/zuiyuelong/article/details/150165155](https://blog.csdn.net/zuiyuelong/article/details/150165155)

\[181] Interface ApplicationListener\<E extends ApplicationEvent>[ https://docs.spring.io/spring-framework/docs/6.1.1/javadoc-api/org/springframework/context/ApplicationListener.html](https://docs.spring.io/spring-framework/docs/6.1.1/javadoc-api/org/springframework/context/ApplicationListener.html)

\[182] 介绍一下spring的ApplicationListener-CSDN博客[ https://blog.csdn.net/T2\_phage/article/details/145138635](https://blog.csdn.net/T2_phage/article/details/145138635)

\[183] 解析Spring事件监听机制与观察者设计模式的核心原理[ https://www.iesdouyin.com/share/video/7463456755255741711/?region=\&mid=7463458976496192292\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=RwUGqPdYJCdW5ReQHF\_r6lNJJi2d2ntohfEu.C5qGbE-\&share\_version=280700\&ts=1769261148\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7463456755255741711/?region=\&mid=7463458976496192292\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=RwUGqPdYJCdW5ReQHF_r6lNJJi2d2ntohfEu.C5qGbE-\&share_version=280700\&ts=1769261148\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[184] 事件驱动架构实战:SpringBoot中ApplicationEvent与EventListener应用场景揭秘 - CSDN文库[ https://wenku.csdn.net/column/1eueaky33y](https://wenku.csdn.net/column/1eueaky33y)

\[185] Spring 框架——事件驱动模型\_spring事件驱动模型-CSDN博客[ https://blog.csdn.net/weixin\_43004044/article/details/131754324](https://blog.csdn.net/weixin_43004044/article/details/131754324)

\[186] インターフェース ApplicationListener\<E extends ApplicationEvent>[ https://spring.pleiades.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationListener.html](https://spring.pleiades.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationListener.html)

\[187] Task Execution and Scheduling[ https://docs.spring.io/spring-framework/reference/6.1/integration/scheduling.html](https://docs.spring.io/spring-framework/reference/6.1/integration/scheduling.html)

\[188] Spring之定时任务和异步任务\_异步定时任务-CSDN博客[ https://blog.csdn.net/weixin\_43583736/article/details/147847773](https://blog.csdn.net/weixin_43583736/article/details/147847773)

\[189] SpringBoot使用Schedule实现异步执行定时任务(多线程)\_springboot schedule 异步多线程-CSDN博客[ https://blog.csdn.net/AlbenXie/article/details/108347392](https://blog.csdn.net/AlbenXie/article/details/108347392)

\[190] Spring Boot线程池优化实现百万数据高效批量插入[ https://www.iesdouyin.com/share/video/7487601655442083084/?region=\&mid=7487603076375857930\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=ZVmbXAMupFlmgwknrLoT30EqE.fxKKJv4h2j1Y7bdbg-\&share\_version=280700\&ts=1769261148\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7487601655442083084/?region=\&mid=7487603076375857930\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=ZVmbXAMupFlmgwknrLoT30EqE.fxKKJv4h2j1Y7bdbg-\&share_version=280700\&ts=1769261148\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[191] Spring Boot 中 @Async 与 @Scheduled 的线程池配置与常见问题一、默认行为:不配置线程池的风 - 掘金[ https://juejin.cn/post/7477875020913164297](https://juejin.cn/post/7477875020913164297)

\[192] Spring Boot 异步与任务调度\_spring boot 任务调度-CSDN博客[ https://blog.csdn.net/Flying\_Fish\_roe/article/details/144826101](https://blog.csdn.net/Flying_Fish_roe/article/details/144826101)

\[193] Task Execution and Scheduling[ https://docs.spring.io/spring-boot/reference/features/task-execution-and-scheduling.html](https://docs.spring.io/spring-boot/reference/features/task-execution-and-scheduling.html)

\[194] Spring Boot 中的 EnvironmentPostProcessor:自定义环境配置-51CTO.COM[ https://www.51cto.com/article/825785.html](https://www.51cto.com/article/825785.html)

\[195] Spring Boot 核心接口与扩展点详细指南-CSDN博客[ https://blog.csdn.net/lilinhai548/article/details/156463403](https://blog.csdn.net/lilinhai548/article/details/156463403)

\[196] SpringBoot扩展点之ApplicationRunner与CommandLineRunner\_springboot applicantionrunner command-CSDN博客[ https://blog.csdn.net/dongdong199033/article/details/129330816](https://blog.csdn.net/dongdong199033/article/details/129330816)

\[197] Spring Boot中Environment接口获取配置信息的使用解析[ https://www.iesdouyin.com/share/video/7535450268717042996/?region=\&mid=7535450352322120482\&u\_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with\_sec\_did=1\&video\_share\_track\_ver=\&titleType=title\&share\_sign=CIEx8yijB\_JagCzqZWLcN5XYA9438CzSSPWq2au1bsw-\&share\_version=280700\&ts=1769261149\&from\_aid=1128\&from\_ssr=1\&share\_track\_info=%7B%22link\_description\_type%22%3A%22%22%7D](https://www.iesdouyin.com/share/video/7535450268717042996/?region=\&mid=7535450352322120482\&u_code=0\&did=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&iid=MS4wLjABAAAANwkJuWIRFOzg5uCpDRpMj4OX-QryoDgn-yYlXQnRwQQ\&with_sec_did=1\&video_share_track_ver=\&titleType=title\&share_sign=CIEx8yijB_JagCzqZWLcN5XYA9438CzSSPWq2au1bsw-\&share_version=280700\&ts=1769261149\&from_aid=1128\&from_ssr=1\&share_track_info=%7B%22link_description_type%22%3A%22%22%7D)

\[198] SpringBoot 核心扩展点详解与案例SpringBoot 核心扩展点详解与案例 SpringBoot 提供了丰富的 - 掘金[ https://juejin.cn/post/7563843746165997611](https://juejin.cn/post/7563843746165997611)
