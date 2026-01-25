
这部分章节涵盖了Spring 框架不可或缺的全部核心技术。
其中最关键的是 Spring 框架的控制反转（IoC）容器。文档在对 Spring 框架 IoC 容器进行深入阐述后，紧接着全面介绍了 Spring 的面向切面编程（AOP）技术。Spring 框架拥有自研的 AOP 框架，该框架概念清晰易懂，能够很好地满足 Java 企业级编程中 80% 的 AOP 核心需求。
文档还介绍了 Spring 与 AspectJ 的集成方案 ——AspectJ 是目前 Java 企业级领域中功能最丰富、且最为成熟的 AOP 实现方案。
AOT技术可用于对应用程序进行提前优化，该技术通常适用于基于 GraalVM 的原生镜像部署场景。

章节概要

- IoC 容器核心概念与基本原理
- Bean 的定义、创建与生命周期管理
- 依赖注入（DI）机制与配置方式
- Singleton、Prototype、Request、Session 等作用域
- Bean 初始化和销毁回调的定制
- Bean 定义的继承与复用
- BeanPostProcessor、BeanFactoryPostProcessor 等扩展点
- @Autowired、@Component、@Repository 等注解的使用
- 组件扫描（Component Scan）与自动装配
- @Inject、@Named 等 JSR 330 标准注解
- @Configuration、@Bean 编程式配置
- Environment 接口与 Profile 管理
- LoadTimeWeaver 注册与字节码增强
- ApplicationContext 的高级功能
- BeanFactory 核心接口 API

## 1.1 IoC容器和Beans
本章介绍 Spring 框架对控制反转（IoC）原则的实现。依赖注入（DI）是控制反转的一种特定形式，在这种模式下，对象仅通过以下三种方式定义自身的依赖项（即协同工作的其他对象）：构造函数参数、工厂方法参数，或对象实例被构造完成、或从工厂方法返回后为其设置的属性。IoC 容器会在创建 Bean 时，自动注入这些依赖项。这一过程与 Bean 自身通过直接实例化类、或采用服务定位器模式等方式控制依赖项的实例化和获取路径，在逻辑上是完全相反的 —— 这也是 “控制反转” 名称的由来。
org.springframework.beans 和 org.springframework.context 这两个包是 Spring 框架 IoC 容器的基础。BeanFactory 接口提供了一套高级配置机制，能够管理任意类型的对象。ApplicationContext 是 BeanFactory 的子接口，它在父接口的基础上新增了以下功能：

- 更便捷地与 Spring 面向切面编程（AOP）特性集成
- 消息资源处理（用于国际化场景）
- 事件发布功能
- 适用于应用层的特定上下文，例如 Web 应用专用的 WebApplicationContext

简而言之，BeanFactory 提供了配置框架和基础功能，而 ApplicationContext 则在此之上添加了更多企业级特性。ApplicationContext 是 BeanFactory 的完整超集，因此本章在描述 Spring IoC 容器时，均以 ApplicationContext 为核心展开。若你需要了解如何使用 BeanFactory 而非 ApplicationContext，可参考《BeanFactory API》相关章节。

在 Spring 中，构成应用程序核心、且由 Spring IoC 容器管理的对象被称为 Bean。Bean 是由 Spring IoC 容器负责实例化、组装和管理的对象；除此之外，它与应用程序中的其他普通对象并无区别。Bean 及其之间的依赖关系，均体现在容器所使用的配置元数据中。

# 1.2 容器
org.springframework.context.ApplicationContext 接口代表 Spring 控制反转（IoC）容器，负责 Bean 的实例化、配置和组装。容器通过读取配置元数据，获取需要实例化、配置和组装的组件相关指令。配置元数据可通过带注解的组件类、包含工厂方法的配置类，或外部 XML 文件、Groovy 脚本进行定义。无论采用哪种格式，你都可以构建应用程序，并处理这些组件之间复杂的依赖关系。
ApplicationContext 接口的多个实现类都属于 Spring 核心模块。在独立应用程序中，通常会创建 AnnotationConfigApplicationContext 或 ClassPathXmlApplicationContext 的实例。
在大多数应用场景下，无需编写显式的用户代码来实例化一个或多个 Spring IoC 容器。例如，在普通 Web 应用场景中，只需在应用的 web.xml 文件中编写一段简单的模板化 Web 描述符 XML 即可（详情参见《Web 应用的便捷 ApplicationContext 实例化》）。在 Spring Boot 场景中，应用上下文会基于通用的配置约定，为你自动隐式启动。

## 1.2.1 配置元数据

正如图中所示，Spring 控制反转（IoC）容器会读取某种格式的配置元数据。这些配置元数据的作用，是让应用开发者能够告知 Spring 容器，应当如何实例化、配置并组装应用中的各个组件。
Spring IoC 容器本身与配置元数据的实际编写格式完全解耦。如今，许多开发者会为自己的 Spring 应用选择基于 Java 的配置方式：

- 基于注解的配置：在应用的组件类上，通过基于注解的配置元数据来定义 Bean。
- 基于 Java 的配置：借助基于 Java 的配置类，在应用类外部定义 Bean。若要使用这些功能，请参考 @Configuration、@Bean、@Import 和 @DependsOn 注解。

Spring 配置至少包含一个、通常包含多个需要由容器管理的 Bean 定义。基于 Java 的配置一般会在 @Configuration 标注的类中，使用 @Bean 注解标注方法，每个方法对应一个 Bean 定义。
这些 Bean 定义对应着构成应用的实际对象。通常情况下，你可以定义服务层对象、持久层对象（例如资源库或数据访问对象（DAO））、表现层对象（例如 Web 控制器），以及基础设施类对象（例如 JPA 实体管理器工厂、Java 消息服务队列等）。一般而言，无需在容器中配置细粒度的领域对象，因为创建和加载领域对象的职责通常由资源库和业务逻辑来承担。

Spring 提供了多种 `ApplicationContext` 的实现。在独立应用中，常见的实现有 `ClassPathXmlApplicationContext` 和 `FileSystemXmlApplicationContext`。虽然 XML 是传统的配置格式，但可以通过少量 XML 来启用注解或 Java 配置的支持，从而让容器使用注解或代码作为元数据来源。

在大多数场景下，无需在应用代码中显式创建 `ApplicationContext` 实例。例如在 Web 应用中，通常只需在 `web.xml` 中添加几行标准配置模板（参见“Web 应用的便捷 ApplicationContext 实例化”）。如果使用 Spring Tools for Eclipse 等 IDE，也可以通过图形化操作快速生成这些配置。

Spring 的配置通常由若干 bean 定义组成。基于 XML 的元数据将这些 bean 作为顶层 `<beans/>` 元素下的多个 `<bean/>` 定义；Java 配置则通常在 `@Configuration` 类中的 `@Bean` 方法里声明。

这些 bean 定义对应于构成应用程序的实际对象，例如服务层对象、数据访问对象（DAO）、表示层对象（如控制器实例）、以及基础设施对象（如 Hibernate 的 SessionFactory、JMS 队列等）。通常不会将细粒度的领域对象放到容器中管理，因为这类对象通常由 DAO 或业务逻辑在运行时创建。然而，借助 Spring 与 AspectJ 的集成，也可以对在容器外部创建的对象应用依赖注入，详见“使用 AspectJ 与 Spring 进行依赖注入域对象”。

## 1.2.3 使用容器

`ApplicationContext` 是一个高级工厂接口，维护着容器中各个 bean 及其依赖关系的注册表。通常通过 `getBean(String name, Class<T> requiredType)` 或 `getBean(Class<T> requiredType)` 等方法检索 bean 实例。

例如，使用基于 XML 的配置可以这样创建并获取 bean：

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

更灵活的做法是使用 `GenericApplicationContext` 结合不同的 `BeanDefinitionReader`（例如 `XmlBeanDefinitionReader`）来从多种配置源加载 bean 定义：

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

你可以在同一 `ApplicationContext` 中混合不同的读取器，从不同配置源加载 bean 定义。

尽管可以在代码中直接调用 `getBean()`，但理想情况下应用逻辑不应依赖于该调用。更好的做法是使用依赖注入（例如通过注解或构造器注入），这样应用代码与 Spring API 解耦。在 Web 场景下，Spring 与各类 Web 框架集成后，会在控制器等组件上自动注入所需的依赖，无需显式调用 `getBean()`。

# 1.3 Bean
Spring IoC 容器管理一个或多个 Bean。这些 Bean 是通过你提供给容器的配置元数据创建的（例如，以 XML <bean/> 定义的形式）。
在容器内部，这些 Bean 定义以 BeanDefinition 对象的形式存在，其中包含（但不限于）以下元数据：

- 包限定的类名: 通常是被定义的Bean的实际实现类.
- Bean的行为配置，它说明了Bean在容器中应该表现的状态(scope, lifecycle callbacks等)
- 对其他Bean的引用，这些引用是Bean工作所需要的，也被成为依赖
- 要在新创建的对象中设置的其他配置元素。例如，池的大小、连接池的连接数等（即对象的属性）。

这些元数据组成一组属性，这些属性构成了每个Bean定义，如下表：

| Property                 | Explained in…                                                |
| :----------------------- | :----------------------------------------------------------- |
| Class                    | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| Name                     | [Naming Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanname) |
| Scope                    | [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) |
| Constructor arguments    | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Properties               | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Autowiring mode          | [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) |
| Lazy initialization mode | [Lazy-initialized Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lazy-init) |
| Initialization method    | [Initialization Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method       | [Destruction Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) |

除了包含创建特定 Bean 相关信息的 Bean 定义外，ApplicationContext 实现类还允许注册容器外部（由用户）创建的现有对象。这可通过 getAutowireCapableBeanFactory() 方法获取 ApplicationContext 的 BeanFactory（返回 DefaultListableBeanFactory 实现类），并调用其 registerSingleton(..) 和 registerBeanDefinition(..) 方法完成。不过，典型应用通常仅使用通过常规 Bean 定义元数据定义的 Bean。

Bean 元数据和手动提供的单例实例需尽早注册，以便容器在自动装配和其他内省步骤中正确处理它们。虽然容器在一定程度上支持覆盖现有元数据和单例实例，但不官方支持运行时注册新 Bean（与工厂的实时访问并发进行），这可能导致并发访问异常、Bean 容器状态不一致，或两者皆有。

## 覆盖Bean
当使用已被占用的标识符注册 Bean 时，会发生 Bean 覆盖。尽管 Bean 覆盖是可行的，但会降低配置的可读性。
   注意：Bean 覆盖功能将在未来版本中被废弃。

若要完全禁用 Bean 覆盖，可在 ApplicationContext 刷新前将 allowBeanDefinitionOverriding 标志设为 false。在此配置下，若尝试覆盖 Bean，容器会抛出异常。
   默认情况下，容器会以 INFO 级别记录每次 Bean 覆盖的尝试，方便你调整配置。尽管不推荐，但你也可将 allowBeanDefinitionOverriding 设为 true 来关闭这些日志。

Java 配置中的 Bean 覆盖 
使用 Java 配置时，只要 @Bean 方法的返回类型与扫描到的 Bean 类匹配，该方法会静默覆盖同名的组件类 Bean。这意味着容器会优先调用 @Bean 工厂方法，而非 Bean 类中预先声明的构造函数。
我们承认在测试场景中覆盖 Bean 较为便捷，Spring 也为此提供了显式支持（详见相关章节）。
## 3.Bean的命名
每个 Bean 有一个或多个标识符，这些标识符在托管该 Bean 的容器中必须唯一。Bean 通常只有一个标识符，若需多个，额外的标识符可视为别名（aliases）。
### 3.1 XML 配置中的 Bean 命名
在基于 XML 的配置元数据中，可通过 id 属性、name 属性或两者结合来指定 Bean 标识符：
id 属性：仅能指定一个唯一 ID，通常为字母数字格式（如 myBean、someService），也可包含特殊字符。
name 属性：可指定多个别名，用逗号（,）、分号（;）或空格分隔。
尽管 id 属性在 XML 规范中为 xsd:string 类型，但 Bean ID 的唯一性由容器（而非 XML 解析器）保证。
### 3.2 未显式命名的 Bean
你无需为 Bean 提供 name 或 id：若未显式指定，容器会为该 Bean 生成唯一名称。但如果需要通过 ref 元素或服务定位器（Service Locator）风格的查找来引用该 Bean，则必须提供名称。不命名的常见场景包括内部 Bean 和自动装配协作对象。
### 3.3 Bean 命名规范
Bean 命名遵循 Java 实例字段的标准命名规范：以小写字母开头，后续采用驼峰命名法（如 accountManager、accountService、userDao、loginController）。
统一的命名规范可提升配置的可读性，且在 Spring AOP 中，按名称为一组相关 Bean 应用通知时也会更便捷。
### 3.4 类路径扫描的 Bean 命名
类路径扫描时，Spring 会为未命名的组件生成 Bean 名称，规则如下：
取类的简单名称，将首字母转为小写；
特殊情况（罕见）：若类名前两个字符均为大写，则保留原大小写（与 java.beans.Introspector.decapitalize 规则一致，Spring 底层采用此方法）。
### 3.5 Bean 别名（外部定义）
在 Bean 定义内部，可通过 id 属性（最多一个）和 name 属性（多个）组合为 Bean 提供多个名称。但有时需要为其他位置定义的 Bean 引入别名（例如大型系统中，各子系统有独立配置）。

```xml
<alias name="fromName" alias="toName"/>
```
在这种情况下，名为fromName的bean（在同一个容器中）在使用这个别名定义之后，也可以被称为toName。
例如，子系统 A 的配置元数据可以用 `subsystemA-dataSource` 的名称来引用一个 DataSource。子系统B的配置元数据可以用`subsystemB-dataSource`的名称来引用DataSource。当组成使用这两个子系统的主应用程序时，主应用程序以`myApp-dataSource`的名称来引用DataSource。要让这三个名称都引用同一个对象，可以在配置元数据中添加以下别名定义。

```XML
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```
现在每个子系统和主系统都可以通过一个唯一的name引用dataSource，并且不与其他定义发生冲突，但引用的是同一个dataSource。

使用注解也可以定义多个Bean的别名。

## 4.实例化Bean
Bean 定义本质上是创建一个或多个对象的 “配方”。容器在收到请求时，会根据命名 Bean 的配方，使用 Bean 定义封装的配置元数据创建（或获取）实际对象。
在基于 XML 的配置中，<bean/> 元素的 class 属性指定要实例化的对象类型，该属性（对应 BeanDefinition 的 Class 属性）通常是必需的（例外情况见《通过实例工厂方法实例化》和《Bean 定义继承》）。Class 属性有两种使用方式：

- 直接构造：容器通过反射调用类的构造函数创建 Bean（等效于 Java 的 new 关键字）；
- 静态工厂方法：容器调用类的静态工厂方法创建 Bean，此时 class 属性指定包含工厂方法的类，返回对象的类型可能与该类相同或不同。

### 通过构造器来实例化Bean
通过构造函数创建 Bean 时，所有普通类都可与 Spring 兼容 —— 无需实现特定接口或遵循特殊编码规范，只需指定 Bean 类即可（若使用特定 IoC 方式，可能需要默认无参构造函数）。

Spring IoC 容器几乎可管理任意类（不限于标准 JavaBean），但大多数用户偏好使用 “默认无参构造函数 + 对应 getter/setter” 的标准 JavaBean。即使是不遵循 JavaBean 规范的遗留类（如老式连接池），Spring 也能管理。
```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

### 静态工厂方法来实例化Bean

当定义一个使用静态工厂方法创建的Bean时，使用class属性来指定包含静态工厂方法的类，并使用名为factory-method的属性来指定工厂方法本身的名称。你应该能够调用这个方法（有可选的参数，如后面所述）并返回一个活的对象，随后这个对象被当作是通过构造函数创建的。这种bean定义的一个用途是在遗留代码中调用静态工厂。

下面的Bean定义指定通过调用工厂方法来创建Bean。该定义没有指定返回对象的类型（类），只指定了包含工厂方法的类。在这个例子中，createInstance()方法必须是一个静态方法。下面的例子展示了如何指定工厂方法。

```java
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
    
    
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

### 实例工厂方法创建Bean
与静态工厂方法类似，但实例工厂方法是调用容器中现有 Bean 的非静态方法来创建新 Bean：
```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

### 确定Bean的运行时类型

确定 Bean 的运行时类型并非易事：
- Bean 元数据中的 class 仅为初始引用，结合工厂方法或 FactoryBean 时，实际运行时类型可能不同；
- AOP 代理可能为 Bean 包装接口代理，导致目标 Bean 的实际类型仅暴露其实现的接口。
推荐方式：调用 BeanFactory.getType(beanName) 获取 Bean 的实际运行时类型，该方法会考虑上述所有情况，返回与 BeanFactory.getBean(beanName) 一致的对象类型。

## 附录：Bean 的生命周期

![Bean生命周期](images/Bean生命周期.png)

Spring统一管理Bean的生命周期

1. Bean的实例化与属性注入

   在第一次调用getBean()时生成Bean的实例，通常通过反射或者CGLIB动态字节码来生成Bean实例或子类。但是并不是直接返回构造完成的对象实例，而是进行包裹返回相应的BeanWrapper实例，这样的好处是可以在接下来非常方便的设置对象的属性（不然如果直接返回实例对象，那么你要通过反射的API才能去增加对象的属性，很麻烦），而且BeanWrapper继承了PropertyAccessor接口可以方便的获取对象属性，也继承了PropertyEditorRegister和TypeConverter接口，可以方便转换属性类型。

2. 检查设置Aware接口

3. BeanPostprocessor的before和after方法,和中间的InitializingBean和init-method方法

4. 检查单例 Bean 是否实现了 `DisposableBean` 接口，并注册销毁时的回调。通常在 Spring 容器关闭时会调用这些销毁方法（例如通过调用 `ConfigurableApplicationContext.close()`）；不应依赖容器内部的手动方法来触发销毁。

下面是按方法执行的前后顺序排列

  - **BeanDefinitionRegistryPostProcessor**

    > 继承BeanFactoryPostProcessor，在运行BeanFactoryPostProcessor之前允许更多的bean definitions注册。特别的，这个类可以注册更多的bean definitions，甚至包括BeanFactoryPostProcessor的实例

    如Apollo里DefaultConfigPropertySourcesProcessorHelper就利用BeanDefinitionRegistryPostProcessor，向容器中注册了多个用于Processor的Bean。

  - **BeanFactoryPostProcessor.postProcessBeanFactory()**

    > 用来在所有的bean definitions加载后，但是还没有实例化bean之前调用本方法
    > 可以修改bean的定义，如是否是单例，是否lazy init，DependsOn，FactoryBeanName等等等等，一般用来修改属性值，一个典型的实现是PropertyResourceConfigurer，用来从配置文件里加载属性放进bean里，或者更换${...}placeHolder

    如Apollo里PropertySourcesProcessor利用BeanFactoryPostProcessor，将config包装成ConfigPropertySource添加到Spring中。

  - **bean的构造方法**

  - **BeanPostProcessor.postProcessBeforeInitialization()**

    >在bean进行初始化方法（如InitializingBean.afterPropertiesSet或者自定义的init方法@PostConstruct）的回调之前调用
    >调用此方法时，bean的属性值已经设置好
    >可以返回一个包装类

  - **@PostConstruct**

    >用来标注在bean的方法上，在**依赖注入后**，放入spring容器前执行一些init逻辑（init方法中可以使用依赖属性）。一个bean中只能有一个@PostConstruct
    >然后注意，这是java规定的注解！！
    >如果在拦截器中,（没见过这个用法，不太懂）
    >必须有InvocationContext方法参数，可以有Object返回值
    >如果不是在拦截器类中：
    >不能有返回值和方法参数
    >标注的方法可以是public, protected,package private or private
    >可以是final的
    >不能抛出运行时异常，会导致容器启动失败

  - **InitializingBean.afterPropertiesSet()**

    >用来BeanFactory将bean的所有属性都设置后，执行一些init逻辑或者只是check属性是否正确和缺失。

  - **BeanPostProcessor.postProcessAfterInitialization()**

    >在bean进行初始化方法（如InitializingBean.afterPropertiesSet或者自定义的init方法@PostConstruct）的回调之后调用
    >调用此方法时，bean的属性值已经设置好
    >可以返回一个包装类
    >可能被多次调用

  - **SmartInitializingSingleton**

    >这个接口可以被一个单例的bean实现，在BeanFactory实例化所有单例bean之后执行。
    >一般用来执行一些init逻辑，在想要急切获取其他bean时可以用来替换InitializingBean.afterPropertiesSet()的方案

  - **SmartLifecycle.start()**

    >Lifecycle接口的扩展，用来实现想在容器启动刷新，或者shutdown时执行一些逻辑。
    > isAutoStartup()用来表示在容器刷新时这个bean是否启动，否则只有容器start时重新创建
    > stop(Runnable)用于需要异步的关闭逻辑，必须要调用callback.run()
    > Phased用于控制多个bean的启动顺序，value较小的会先启动，shutdown时会后关闭。
    > 如ComponentB依赖componentA先启动，则componentA.phase()应该返回一个较小的值，关闭时B会先关闭。
    > 如果明确指定depends-on，以depends-on为准。
    > 任何没有实现SmartLifecycle的bean的phase会是0，这也就意味着如果SmartLifecycle Bean的phase返回负数，将优先于所有的容器bean启动，正数反之。
    > SmartLifecycle的DEFAULT_PHASE = Integer.MAX_VALUE;

# 1.6 自定义Bean的性质
Spring 框架提供了多个接口，可用于自定义 Bean 的特性。本节将这些接口分为以下三类介绍：

- 生命周期回调接口
- ApplicationContextAware 和 BeanNameAware 接口
- 其他 Aware 接口

## 生命周期回调
若要与容器对 Bean 生命周期的管理进行交互，可实现 Spring 提供的 InitializingBean 和 DisposableBean 接口。容器会为实现 InitializingBean 接口的 Bean 调用 afterPropertiesSet() 方法，为实现 DisposableBean 接口的 Bean 调用 destroy() 方法，让 Bean 能在初始化和销毁阶段执行特定操作。

在现代 Spring 应用中，接收生命周期回调的最佳实践是使用 JSR-250 规范的 @PostConstruct 和 @PreDestroy 注解。使用这两个注解的优势在于，Bean 无需耦合 Spring 专属的接口。相关细节可参考《使用 @PostConstruct 和 @PreDestroy》章节。
若你不想使用 JSR-250 注解，但仍希望解除代码与 Spring 的耦合，可考虑使用 Bean 定义元数据中的 init-method 和 destroy-method 属性。

Spring 框架内部会通过 BeanPostProcessor 的实现类，处理所有可识别的回调接口并调用对应的方法。如果需要实现 Spring 默认未提供的自定义特性或其他生命周期行为，你可以自行实现 BeanPostProcessor 接口。更多信息可参考《容器扩展点》章节。
除了初始化和销毁回调外，由 Spring 管理的对象还可实现 Lifecycle 接口，使这些对象能跟随容器自身的生命周期，参与启动和关闭流程。
下文将详细介绍各类生命周期回调接口。

### 初始化回调
org.springframework.beans.factory.InitializingBean 接口允许 Bean 在容器为其注入所有必要属性后，执行初始化操作。该接口仅定义了一个方法：
java
运行
```java
void afterPropertiesSet() throws Exception;
```
不推荐使用 InitializingBean 接口，因为它会让代码不必要地耦合到 Spring 框架。作为替代方案，建议使用 @PostConstruct 注解，或指定一个普通 Java 对象（POJO）的初始化方法。
- 基于 XML 的配置元数据中，可通过 init-method 属性指定初始化方法的名称，该方法需满足无返回值、无参数的签名要求；
- 基于 Java 的配置方式中，可使用 @Bean 注解的 initMethod 属性。

```java
public class ExampleBean {
    public void init() {
        // 执行初始化操作
    }
}
```
上述示例的效果，与以下实现 InitializingBean 接口的示例几乎完全一致：
```java
public class AnotherExampleBean implements InitializingBean {
	@Override
	public void afterPropertiesSet() {
		// 执行初始化操作
	}
}
```
不同的是，第一个示例的代码未与 Spring 框架耦合。

需要注意的是，@PostConstruct 注解标注的方法及各类初始化方法，均在容器的单例创建锁内执行。只有当 @PostConstruct 方法执行完成后，Bean 实例才会被视为完全初始化，并对外发布。这类独立的初始化方法，仅适用于验证配置状态，或根据给定配置初始化部分数据结构，不可在此阶段访问外部 Bean，否则存在初始化死锁的风险。

若需要执行耗时的初始化后操作（例如异步的数据库准备步骤），你的 Bean 应选择以下两种方式实现：
- 实现 SmartInitializingSingleton 接口，重写 afterSingletonsInstantiated() 方法；
- 监听容器刷新事件：实现 ApplicationListener<ContextRefreshedEvent> 接口，或使用等效的注解 @EventListener(ContextRefreshedEvent.class)。

以上两种方式的执行时机，均在所有常规单例 Bean 初始化完成后，因此不会处于任何单例创建锁的范围内。
此外，你也可以实现 (Smart)Lifecycle 接口，融入容器的整体生命周期管理，该接口支持自动启动机制、销毁前的停止步骤，以及可能的停止 / 重启回调（详见下文）。

### 销毁回调
实现 org.springframework.beans.factory.DisposableBean 接口的 Bean，会在其所属容器销毁时收到回调通知。该接口仅定义了一个方法：
```java
void destroy() throws Exception;
```

不推荐使用 DisposableBean 回调接口，因为它会让代码不必要地耦合到 Spring 框架。作为替代方案，建议使用 @PreDestroy 注解，或为 Bean 定义指定一个通用的销毁方法。

- 基于 XML 的配置元数据中，可在 <bean/> 标签上使用 destroy-method 属性；
- 基于 Java 的配置方式中，可使用 @Bean 注解的 destroyMethod 属性。

```java
public class ExampleBean {
    public void cleanup() {
        // 执行销毁操作（如释放连接池）
    }
}
```

```xml
<bean id="exampleDestructionBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

上述定义的效果，与以下实现 DisposableBean 接口的定义几乎完全一致：

pring 还支持自动推断销毁方法，会自动检测 Bean 中名为 close 或 shutdown 的公共方法。这是 Java 配置类中 @Bean 方法的默认行为，且会自动匹配实现了 java.lang.AutoCloseable 或 java.io.Closeable 接口的类，同样不会将销毁逻辑耦合到 Spring 框架。
在 XML 配置中，可将 <bean> 元素的 destroy-method 属性设为特殊值 (inferred)，指示 Spring 为该 Bean 定义自动检测其类中的公共 close 或 shutdown 方法作为销毁方法。也可在 <beans> 元素的 default-destroy-method 属性中设置该特殊值，让整个配置文件中的所有 Bean 定义都应用此行为（详见《默认的初始化和销毁方法》章节）。
若需要更精细的关闭阶段控制，可实现 Lifecycle 接口，在所有单例 Bean 的销毁方法执行前，接收提前的停止信号；也可实现 SmartLifecycle 接口，定义有时间限制的停止步骤，容器会等待所有此类停止操作完成后，再执行销毁方法。


Spring 容器保证：在为 Bean 注入所有依赖后，立即调用配置的初始化回调方法。因此，初始化回调方法是在原始 Bean 引用上执行的，这意味着 AOP 拦截器等代理逻辑尚未应用到该 Bean 上。容器会先完整创建目标 Bean，再为其应用包含拦截器链的 AOP 代理。若目标 Bean 和代理 Bean 是单独定义的，代码甚至可以绕过代理，直接与原始目标 Bean 交互。因此，将拦截器应用到初始化方法上是不符合逻辑的 —— 这会让目标 Bean 的生命周期与代理 / 拦截器耦合，且当代码直接操作原始目标 Bean 时，会出现异常的语义行为。

## 启动和关闭回调 Lifecycle
Lifecycle 接口为拥有自身生命周期需求的对象（例如需要启动和停止后台进程的对象）定义了核心方法：
```java
public interface Lifecycle {
void start();
void stop();
boolean isRunning();
}
```

所有由 Spring 管理的对象都可实现 Lifecycle 接口。当 ApplicationContext 自身接收到启动和停止信号时（例如运行时的停止 / 重启场景），会将这些调用级联传递给上下文中所有 Lifecycle 接口的实现类，该过程通过委托给 LifecycleProcessor 接口完成，接口定义如下：

```java
public interface LifecycleProcessor extends Lifecycle {
void onRefresh();
void onClose();
}
```

LifecycleProcessor 接口本身是 Lifecycle 接口的扩展，还新增了两个方法，用于响应容器的刷新和关闭事件。
需要注意的是，标准的 org.springframework.context.Lifecycle 接口仅为显式的启动和停止通知定义了契约，并不意味着容器刷新时会自动启动。若需要对自动启动进行精细控制，或需要优雅地停止特定 Bean（包括启动和停止阶段），应考虑实现扩展的 org.springframework.context.SmartLifecycle 接口。
此外，无法保证停止通知一定在销毁操作之前执行：在常规的容器关闭流程中，所有 Lifecycle Bean 会先接收停止通知，再执行通用的销毁回调；但在容器生命周期内的热刷新，或刷新尝试失败时，容器只会调用销毁方法。
启动和关闭的调用顺序至关重要：若两个对象之间存在 depends-on 依赖关系，被依赖的对象会先启动、后停止，依赖方则会后启动、先停止。但在某些场景下，对象间的直接依赖关系是未知的，你可能仅知道某类对象应在另一类对象之前启动。此时，SmartLifecycle 接口提供了另一个配置项 —— 其超接口 Phased 中定义的 getPhase() 方法。
Phased 接口的定义如下：
```java
public interface Phased {
int getPhase();
}
```

SmartLifecycle 接口的定义如下：

```java
public interface SmartLifecycle extends Lifecycle, Phased {
boolean isAutoStartup();
void stop(Runnable callback);
}
```

启动时，阶段值（phase）越小的对象越先启动；停止时，则按照相反的顺序执行。因此，实现 SmartLifecycle 接口且 getPhase() 方法返回 Integer.MIN_VALUE 的对象，会是最先启动、最后停止的对象；反之，阶段值为 Integer.MAX_VALUE 的对象，会最后启动、最先停止（通常因为该对象依赖其他进程的运行）。
需要注意的是，未实现 SmartLifecycle 接口的普通 Lifecycle 对象，默认阶段值为 0。因此：
阶段值为负数的对象，会在标准组件之前启动、之后停止；
阶段值为正数的对象，会在标准组件之后启动、之前停止。
SmartLifecycle 接口定义的 stop 方法接收一个回调参数，所有实现类必须在自身的关闭流程完成后，调用该回调的 run() 方法。这让容器在必要时支持异步关闭—— 因为 LifecycleProcessor 接口的默认实现类 DefaultLifecycleProcessor，会为每个阶段的对象组等待指定的超时时间，直到其调用回调方法。默认的单阶段超时时间为 30 秒。
你可以通过在容器中定义一个名为 lifecycleProcessor 的 Bean，覆盖默认的生命周期处理器实例；若仅需修改超时时间，配置如下即可：
````xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
        <!-- 超时时间，单位：毫秒 -->
<property name="timeoutPerShutdownPhase" value="10000"/>
        </bean>
````

如前所述，LifecycleProcessor 接口还为容器的刷新和关闭定义了回调方法：
容器关闭时的回调，会驱动关闭流程，效果等同于显式调用 stop() 方法；
容器刷新时的回调，则为 SmartLifecycle Bean 提供了另一项特性：当容器完成刷新（所有对象实例化并初始化完成）后，会调用该回调。此时，默认的生命周期处理器会检查每个 SmartLifecycle 对象的 isAutoStartup() 方法返回的布尔值 —— 若返回 true，则该对象会在此时自动启动，而非等待显式调用容器或自身的 start() 方法（与容器刷新不同，标准的容器实现不会自动执行容器的 start() 方法）。启动顺序仍由阶段值和 depends-on 依赖关系决定。

### 组合使用多种生命周期机制
从 Spring 2.5 开始，控制 Bean 生命周期行为有三种可选方式：
- 实现 InitializingBean 和 DisposableBean 回调接口；
- 自定义 init() 和 destroy() 方法；
- 使用 @PostConstruct 和 @PreDestroy 注解。

你可以为同一个 Bean 组合使用上述多种机制。
若为一个 Bean 配置了多种生命周期机制，且每种机制指定的方法名不同，那么所有配置的方法会按照下述顺序依次执行；若为多种生命周期机制配置了相同的方法名（例如，将 init() 同时指定为多种机制的初始化方法），则该方法只会执行一次，如前文所述。
为同一个 Bean 配置多种生命周期机制且初始化方法名不同时，执行顺序如下：
1. 标注 @PostConstruct 注解的方法；
2. 实现 InitializingBean 接口的 afterPropertiesSet() 方法；
3. 自定义配置的 init() 方法。

销毁方法的执行顺序与初始化方法一致：

1. 标注 @PreDestroy 注解的方法；
2. 实现 DisposableBean 接口的 destroy() 方法；
3. 自定义配置的 destroy() 方法。

## 其他aware接口
```java
public interface ApplicationContextAware {
    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

```java
@Component
@Slf4j
public class SpringUtil implements ApplicationContextAware {

    private static ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        if(context == null){
            context = applicationContext;
        }
    }

    public static ApplicationContext getApplicationContext(){
        return context;
    }

    public static <T> T getBean(String name) {
        if (context == null){
            throw new IllegalStateException("applicaitonContext un inject");
        }
        try {
            return (T)context.getBean(name);
        } catch (BeansException e) {
            log.error(" get Bean error",e);
        }
        return null;
    }

    public static <T> T getBeanByClass(Class className){
        if (context == null){
            throw new IllegalStateException("applicaitonContext un inject");
        }
        try {
            return (T)context.getBean(className);
        } catch (BeansException e) {
            log.error("get Bean by className error",e);
        }
        return null;
    }
    /**
    * @description  发布event
    */
    public static void publishEvent(ApplicationEvent event){
        getApplicationContext().publishEvent(event);
    }

}
```

除了前文介绍的 ApplicationContextAware 和 BeanNameAware，Spring 还提供了一系列 Aware 回调接口，让 Bean 能向容器表明自身需要的特定基础设施依赖。通用规则是：接口名称会直接指示依赖的类型。
下表总结了最常用的 Aware 接口：


| Name                             | Injected Dependency会注入的依赖                              | Explained in…                                                |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `ApplicationContextAware`        | 注入`ApplicationContext`，最常用和最全面的                   | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| `ApplicationEventPublisherAware` | 注入`ApplicationEventPublisher`用于event发布                 | [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction) |
| `MessageSourceAware`             | 注入用于配置国际化的`MessageSource`。Configured strategy for resolving messages (with support for parametrization and internationalization). | [Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction) |
| `ResourceLoaderAware`            | 获取Resource。Configured loader for low-level access to resources. | [Resources](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources) |

上面是applicationContext特有的aware接口，也会更可能用到。

| Name                         | Injected Dependency                                          | Explained in…                                                |
| :--------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `BeanClassLoaderAware`       | Class loader used to load the bean classes.                  | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| `BeanFactoryAware`           | Declaring `BeanFactory`.                                     | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| `BeanNameAware`              | Name of the declaring bean.                                  | [`ApplicationContextAware` and `BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| `LoadTimeWeaverAware`        | Defined weaver for processing class definition at load time. | [Load-time Weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw) |
| `NotificationPublisherAware` | Spring JMX notification publisher.                           | [Notifications](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jmx-notifications) |
| `ServletConfigAware`         | Current `ServletConfig` the container runs in. Valid only in a web-aware Spring `ApplicationContext`. | [Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc) |
| `ServletContextAware`        | Current `ServletContext` the container runs in. Valid only in a web-aware Spring `ApplicationContext`. | [Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc) |

请再次注意，使用这些接口会将您的代码和Spring API耦合，并且违反IOC风格。因此，建议将他们用于需要对容器进行编程访问的基础Bean。

# 容器扩展点（重点！！！）

## 使用`BeanPostProcessor`自定义Bean

`BeanPostProcessor` 接口定义了回调方法，你可以实现这些方法来提供自定义的（或覆盖容器默认的）实例化逻辑、依赖解析逻辑等。如果希望在 Spring 容器完成 Bean 的实例化、配置和初始化后执行一些自定义逻辑，你可以接入一个或多个自定义的 `BeanPostProcessor` 实现类。

###  多 BeanPostProcessor 执行顺序

你可以配置多个 `BeanPostProcessor` 实例，并通过设置 `order` 属性控制它们的执行顺序。**仅当 `BeanPostProcessor` 实现 `Ordered` 接口时，才能设置该属性**。如果你编写自定义的 `BeanPostProcessor`，也建议实现 `Ordered` 接口。更多细节可参考 `BeanPostProcessor` 和 `Ordered` 接口的 Javadoc，同时可参考「以编程方式注册 BeanPostProcessor 实例」的说明。

### 核心特性与作用域

- `BeanPostProcessor` 作用于 **Bean（或对象）实例**：Spring IoC 容器先实例化 Bean 实例，再由 `BeanPostProcessor` 执行处理逻辑。
- 作用域为**每个容器独立**：仅对定义它的容器内的 Bean 生效，即使容器存在层级关系，跨容器的 `BeanPostProcessor` 也不会处理彼此的 Bean。
- 区别于 `BeanFactoryPostProcessor`：`BeanPostProcessor` 操作 Bean 实例，而修改 Bean 定义（即 Bean 的配置蓝图）需使用 `BeanFactoryPostProcessor`（详见下文）。

### 核心方法

`org.springframework.beans.factory.config.BeanPostProcessor` 接口仅包含两个回调方法：

1. `postProcessBeforeInitialization`：容器调用 Bean 初始化方法（如 `InitializingBean.afterPropertiesSet()` 或自定义 `init-method`）**之前**执行；
2. `postProcessAfterInitialization`：Bean 初始化回调执行**之后**执行。

处理器可对 Bean 实例执行任意操作（包括完全忽略回调），典型场景包括：检查回调接口、**为 Bean 包装代理对象。Spring AOP 的自动代理功能正是通过 `BeanPostProcessor` 实现代理包装逻辑的**。

自定义例子：

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // 初始化前：直接返回原 Bean 实例
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean;
    }

    // 初始化后：打印 Bean 名称和 toString() 结果
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

### 自动检测与注册

- `ApplicationContext` 会自动检测配置元数据中所有实现 `BeanPostProcessor` 接口的 Bean，并将其注册为后置处理器，在创建 Bean 时调用。
- 特殊注意：通过配置类的 `@Bean` 工厂方法声明 `BeanPostProcessor` 时，工厂方法的返回类型需为实现类本身或至少是 `BeanPostProcessor` 接口 —— 否则 `ApplicationContext` 无法在完全创建该 Bean 前通过类型自动检测到它（`BeanPostProcessor` 需提前实例化以处理其他 Bean 的初始化）。

### 编程式注册

尽管推荐使用 `ApplicationContext` 自动检测，但也可通过 `ConfigurableBeanFactory` 的 `addBeanPostProcessor` 方法编程式注册。这种方式适用于注册前需执行条件逻辑，或跨容器层级复制处理器的场景，但需注意：

- 编程式注册的 `BeanPostProcessor` 不遵循 `Ordered` 接口，执行顺序由注册顺序决定；
- 编程式注册的处理器**始终优先于**自动检测的处理器执行（无论显式排序如何）。

### 与 AOP 自动代理的兼容问题

实现 `BeanPostProcessor` 的类是容器的特殊组件：

- 所有 `BeanPostProcessor` 及其直接引用的 Bean 会在 `ApplicationContext` 启动阶段提前实例化；
- AOP 自动代理本身也是 `BeanPostProcessor` 实现，因此 `BeanPostProcessor` 实例及其直接引用的 Bean **不支持自动代理**，无法织入切面；
- 若通过自动装配 /`@Resource` 注入依赖，Spring 可能会访问意外的 Bean，导致这些 Bean 失去自动代理或后置处理的资格。

### 典型应用：AutowiredAnnotationBeanPostProcessor

Spring 内置的 `AutowiredAnnotationBeanPostProcessor` 是 `BeanPostProcessor` 的经典应用 —— 它能自动装配标注了 `@Autowired` 的字段、setter 方法和任意配置方法，是扩展 Spring IoC 容器的常用方式。

- **像注解的处理和AOP代理都是通过BeanPostProcessor来实现的**！！即可以找出添加某个注解和实现某个接口的所有实现类，进行处理设置属性等。如@Async，即在实现BeanPostProcessor的实现类里，检查如果添加@Async，就创建一个代理对象。

- 如Apollo源码里，实现了BeanPostProcessor接口，解析每个Bean中@Value的字段为SpringValue对象，在配置文件更新后反射更新其值。

##  二、使用`BeanFactoryPostProcessor`自定义配置元数据

`org.springframework.beans.factory.config.BeanFactoryPostProcessor` 是另一个核心扩展点，与 `BeanPostProcessor` 语义相似，但核心区别是：

- `BeanFactoryPostProcessor` 操作 **Bean 配置元数据**（而非 Bean 实例）；
- 容器会在实例化除 `BeanFactoryPostProcessor` 外的所有 Bean **之前**，调用 `BeanFactoryPostProcessor` 读取并修改配置元数据。

### 2.1 核心规则

- **执行顺序**：可通过 `Ordered` 接口的 `order` 属性控制多个 `BeanFactoryPostProcessor` 的执行顺序；
- **禁止操作 Bean 实例**：技术上可通过 `BeanFactory.getBean()` 获取 Bean 实例，但会导致 Bean 提前实例化，违反容器生命周期，可能绕过后置处理；
- **作用域**：与 `BeanPostProcessor` 一致，仅对定义它的容器内的 Bean 定义生效；
- **延迟初始化**：`BeanFactoryPostProcessor` 不支持延迟初始化（即使配置 `default-lazy-init="true"`，容器也会提前实例化）。

### 2.2 自动检测与注册

`ApplicationContext` 会自动检测实现 `BeanFactoryPostProcessor` 接口的 Bean，并在合适时机调用其处理逻辑，可像普通 Bean 一样部署。

### 2.3 示例 1：PropertySourcesPlaceholderConfigurer（属性占位符替换）

用于将 Bean 定义中的属性值外置到 `.properties` 文件，方便配置环境相关参数（如数据库信息）。

```properties
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

```xml
<!-- 注册属性占位符处理器 -->
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<!-- 使用占位符定义 DataSource -->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

#### 关键特性

- 占位符格式为 `${property-name}`，支持自定义前缀、后缀、默认值分隔符；
- 默认会从指定 Properties 文件、Spring Environment 和 Java 系统属性中查找属性；
- 可用于动态替换类名（运行时选择实现类）

### 2.4 示例 2：PropertyOverrideConfigurer（属性覆盖）

与 `PropertySourcesPlaceholderConfigurer` 类似，但支持为 Bean 属性设置默认值：

- 若覆盖文件中无对应属性，使用 Bean 定义中的默认值；
- 覆盖规则：`beanName.property=value`，支持复合属性名；
- 多处理器覆盖同一属性时，最后一个生效。

当在ApplicationContext内声明一个BeanFactoryPostProcessor时，它就会自动运行，以便对定义容器的配置元数据应用更改。Spring包含了许多预定义的BeanFactoryPostProcessor，如`PropertyOverrideConfigurer`和`PropertySourcesPlaceholderConfigurer`。您也可以使用自定义的BeanFactoryPostProcessor--例如，注册自定义属性编辑器。

ApplicationContext 会自动检测部署到它的任何实现 BeanFactoryPostProcessor 接口的 Bean。它在适当的时候将这些bean用作BeanFactoryPostProcessor。你可以像部署其他bean一样部署这些后处理器bean。

与BeanPostProcessors一样，你通常不希望将BeanFactoryPostProcessors配置为懒惰初始化。如果没有其他 bean 引用 Bean(Factory)PostProcessor，那么这个后处理器将不会被实例化。因此，将它标记为懒惰初始化将被忽略，即使你在 `<beans />` 元素的声明中将 default-lazy-init 属性设置为 true，Bean(Factory)PostProcessor 也会被急切地实例化

## 三、使用 FactoryBean 自定义实例化逻辑

实现 `org.springframework.beans.factory.FactoryBean` 接口的类本身是「工厂」，可定制复杂的 Bean 实例化逻辑（适合用 Java 代码替代冗长的 XML 配置）。

### 3.1 核心方法

`FactoryBean<T>` 接口包含三个方法：

1. `T getObject()`：返回工厂创建的对象实例（可返回单例或原型对象）；
2. `boolean isSingleton()`：返回 `true` 表示单例，`false` 表示原型（默认返回 `true`）；
3. `Class<?> getObjectType()`：返回 `getObject()` 方法的返回类型（未知则返回 `null`）。

### 3.2 核心特性

- Spring 框架内置了 50 多个 `FactoryBean` 实现，是容器实例化逻辑的重要扩展点；
- 获取 FactoryBean 本身：调用 `ApplicationContext.getBean()` 时，在 Bean ID 前加 `&` 符号（如 `getBean("&myBean")` 返回 FactoryBean 实例，`getBean("myBean")` 返回工厂创建的 Bean 实例）。

### 总结

1. **BeanPostProcessor**：操作 Bean 实例，在 Bean 初始化前后执行，用于修改 Bean 实例、包装代理等；
2. **BeanFactoryPostProcessor**：操作 Bean 配置元数据，在 Bean 实例化前执行，用于动态修改配置（如属性占位符替换）；
3. **FactoryBean**：自定义 Bean 实例化逻辑，适合复杂初始化场景，可通过 `&` 符号区分获取工厂本身和工厂创建的 Bean。

# 1.9 基于注解的容器配置

Spring 对基于注解的配置提供了全面支持，其核心是通过在组件类对应的类、方法或字段声明上添加注解，直接操作组件类自身的元数据。正如《示例：AutowiredAnnotationBeanPostProcessor》一节所述，Spring 会将 `BeanPostProcessor` 与注解结合使用，使核心 IoC 容器能够识别特定注解。

例如，`@Autowired` 注解具备《自动装配协作对象》章节中描述的全部功能，同时还提供了更细粒度的控制能力和更广的适用范围。此外，Spring 还支持 JSR-250 注解（如 `@PostConstruct` 和 `@PreDestroy`），以及 `jakarta.inject` 包中包含的 JSR-330（Java 依赖注入）注解（如 `@Inject` 和 `@Named`）。有关这些注解的详细信息可参考对应章节。

**注解注入的执行时机早于外部属性注入**。因此，当采用混合配置方式时，外部配置（例如通过 XML 指定的 Bean 属性）会有效覆盖通过注解配置的属性值。

从技术层面来说，你可以将这些后置处理器注册为独立的 Bean 定义，但在 `AnnotationConfigApplicationContext` 中，它们已经被隐式注册完成。

Spring 基于注解的配置通过 `BeanPostProcessor` 识别注解元数据，`@Autowired` 注解相比传统自动装配具备更细粒度的控制能力。

**执行顺序**：注解注入先于外部属性注入执行，外部配置可覆盖注解配置的属性值。

# 使用 @Autowired 注解

在本节的示例中，你可以使用 JSR 330 规范的 `@Inject` 注解替代 Spring 自带的 `@Autowired` 注解。相关详情可参考此处。

## 1. @Autowired 注解的应用场景

### 1.1 构造函数注入

你可以将 `@Autowired` 注解应用于构造函数，示例如下：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // 其他方法...
}
```

> **注意**：如果目标 Bean 仅定义了一个构造函数，则无需在该构造函数上添加 `@Autowired` 注解。但如果存在多个构造函数，且没有标记为主要（primary）或默认构造函数时，必须为其中至少一个构造函数添加 `@Autowired` 注解，以告知容器应使用哪个构造函数。具体可参考「构造函数解析」相关说明。

### 1.2 Setter 方法注入

`@Autowired` 也可应用于传统的 setter 方法，示例如下：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // 其他方法...
}
```

### 1.3 任意方法注入

你可以将 `@Autowired` 应用于任意名称、包含多个参数的方法，示例如下：

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // 其他方法...
}
```

### 1.4 字段注入（支持与构造函数混合使用）

`@Autowired` 还可直接应用于字段，甚至与构造函数注入混合使用，示例如下：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // 其他方法...
}
```

> **重要提示**：确保你标注 `@Autowired` 的注入点所使用的类型，与目标组件（如 `MovieCatalog` 或 `CustomerPreferenceDao`）的声明类型保持一致。否则，运行时可能因 “未找到类型匹配项” 错误导致注入失败。
>
> - 对于 XML 定义的 Bean 或通过类路径扫描发现的组件类，容器通常能提前知晓其具体类型；
> - 对于 `@Bean` 工厂方法，需确保声明的返回类型足够明确（至少与注入点所需类型一致）。

## 2. 自注入（Self Injection）

`@Autowired` 也支持注入自身引用（即指向当前正在被注入的 Bean 的引用），但需注意：

- 自注入是**兜底机制**：对其他组件的常规依赖始终优先于自注入，自引用不会参与常规的自动装配候选选择，因此永远不会成为 “主要（primary）” 候选，优先级最低；
- 实际使用中应仅作为最后手段：例如通过 Bean 的事务代理调用同一实例的其他方法。更优方案是将相关方法抽离到独立的委托 Bean 中，或使用 `@Resource`（可通过唯一名称获取指向当前 Bean 的代理）；
- 同一 `@Configuration` 类中注入 `@Bean` 方法的返回结果，本质上也属于自引用场景：需在实际需要的方法签名中延迟解析此类引用（而非在配置类的自动装配字段中），或将受影响的 `@Bean` 方法声明为 `static`，使其与配置类实例及其生命周期解耦。否则，此类 Bean 仅会在兜底阶段被考虑，容器会优先选择其他配置类中匹配的 Bean（若存在）。

## 3. 注入集合 / 数组类型

你可以为期望数组类型的字段或方法添加 `@Autowired` 注解，让 Spring 从 `ApplicationContext` 中注入所有特定类型的 Bean，示例如下：

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // 其他方法...
}
```

该规则同样适用于带类型的集合，示例如下：

```java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // 其他方法...
}
```

### 3.1 集合排序

若希望数组 / 列表中的元素按特定顺序排列，目标 Bean 可实现 `org.springframework.core.Ordered` 接口，或使用 `@Order` 注解、标准 `@Priority` 注解。否则，元素顺序将遵循容器中对应目标 Bean 定义的注册顺序。

> **补充说明**：
>
> - `@Order` 注解可标注在目标类级别和 `@Bean` 方法上（适用于同一 Bean 类有多个定义的场景）；
> - `@Order` 值仅影响注入点的优先级，不影响单例 Bean 的启动顺序（启动顺序由依赖关系和 `@DependsOn` 声明决定）；
> - 配置类上的 `@Order` 仅影响启动时配置类的解析顺序，不影响内部 `@Bean` 方法；
> - 标准的 `jakarta.annotation.Priority` 注解无法用于 `@Bean` 方法（不能标注在方法上），可通过 `@Order` 结合 `@Primary` 注解模拟其语义。

### 3.2 注入 Map 类型

只要 Map 的键类型为 `String`，带类型的 Map 实例也可被自动装配：Map 的值为所有匹配类型的 Bean，键为对应的 Bean 名称，示例如下：

```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // 其他方法...
}
```

## 4. 非必需注入（可选依赖）

默认情况下，若某个注入点没有匹配的候选 Bean，自动装配会失败（数组 / 集合 / Map 类型要求至少有一个匹配元素）。

默认行为下，标注注解的方法和字段被视为 “必需依赖”。你可以通过设置 `@Autowired` 的 `required` 属性为 `false`，将注入点标记为非必需，让框架跳过无法满足的注入点，示例如下：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // 其他方法...
}
```

> **行为说明**：
>
> - 若非必需方法的依赖（或多参数方法中的任一依赖）不可用，该方法不会被调用；
> - 若非必需字段的依赖不可用，该字段不会被赋值，保持默认值；
> - `required=false` 表示该属性对自动装配而言是可选的，无法装配时会被忽略，允许通过依赖注入选择性覆盖属性的默认值。

### 4.1 构造函数 / 工厂方法参数的特殊规则

构造函数和工厂方法参数是特殊情况：由于 Spring 的构造函数解析算法可能处理多个构造函数，`@Autowired` 的 `required` 属性语义略有不同。

- 构造函数 / 工厂方法参数默认是 “必需的”，但在单构造函数场景下有特殊规则：若注入点为数组 / 集合 / Map 类型且无匹配 Bean，会解析为空实例；
- 这支持一种常见实现模式：将所有依赖声明在唯一的多参数构造函数中（例如仅声明一个公共构造函数，无需添加 `@Autowired` 注解）；
- 一个 Bean 类中仅能有一个构造函数将 `@Autowired` 的 `required` 属性设为 `true`；若 `required` 保持默认值 `true`，则仅能有一个构造函数标注 `@Autowired`；若多个构造函数标注该注解，需均设置 `required=false`，容器会选择依赖数量最多且能匹配到 Bean 的构造函数。

### 4.2 其他可选依赖声明方式

除了 `required=false`，还可通过以下方式声明非必需依赖：

1. 使用 Java 的 `java.util.Optional` 包装依赖类型：

   ```java
   public class SimpleMovieLister {
   
       @Autowired
       public void setMovieFinder(Optional<MovieFinder> movieFinder) {
           // 业务逻辑...
       }
   }
   ```

2. 使用参数级别的 `@Nullable` 注解（任意包下的该注解均可，如 JSpecify 的 `org.jspecify.annotations.Nullable`）：

   ```java
   public class SimpleMovieLister {
   
       @Autowired
       public void setMovieFinder(@Nullable MovieFinder movieFinder) {
           // 业务逻辑...
       }
   }
   ```

3. Kotlin 可直接利用内置的空安全特性。

## 5. 注入 Spring 内置容器对象

`@Autowired` 可用于注入 Spring 内置的、可自动解析的依赖接口：`BeanFactory`、`ApplicationContext`、`Environment`、`ResourceLoader`、`ApplicationEventPublisher` 和 `MessageSource`。这些接口及其扩展接口（如 `ConfigurableApplicationContext`、`ResourcePatternResolver`）无需特殊配置即可自动解析，示例如下：

```java
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // 其他方法...
}
```

## 6. 注解生效限制

`@Autowired`、`@Inject`、`@Value` 和 `@Resource` 注解由 Spring 的 `BeanPostProcessor` 实现类处理。这意味着：

- 你**无法**在自定义的 `BeanPostProcessor` 或 `BeanFactoryPostProcessor` 类型中应用这些注解；
- 此类处理器必须通过 XML 或 Spring `@Bean` 方法**显式装配**。

### 总结

1. **@Autowired 核心用法**：支持构造函数、setter 方法、任意方法、字段注入，可注入单个 Bean 或集合 / 数组 / Map 类型的多个 Bean，集合可通过 `@Order`/`Ordered` 控制顺序；
2. **可选依赖配置**：可通过 `required=false`、`Optional`、`@Nullable` 声明非必需依赖，避免注入失败；
3. **特殊场景限制**：自注入优先级最低，Spring 内置处理器类中无法使用 `@Autowired`，需显式装配。


# 1.12 Java-based 容器配置

这节介绍如何在Java代码中通过注解的方式配置Spring容器，它包括：

- [基本概念: `@Bean` and `@Configuration`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-basic-concepts)
- [通过`AnnotationConfigApplicationContext实例化spring容器`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-instantiating-container)
- [使用 `@Bean` Annotation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-bean-annotation)
- [使用`@Configuration` annotation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-configuration-annotation)
- [Composing Java-based Configurations](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-composing-configuration-classes)
- [Bean Definition Profiles](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles)
- [`PropertySource` Abstraction](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-property-source-abstraction)
- [使用 `@PropertySource`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-using-propertysource)
- [Placeholder Resolution in Statements](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-placeholder-resolution-in-statements)

## 1.12.1 基础概念：`@Bean` 和 `@Configuration`

Spring新的Java-configuration的核心构件是`@Configuration`注解的类和`@Bean`注解的方法

@Bean注解用于表示一个方法实例化、配置和初始化一个新的对象，以便Spring Ioc容器管理。@Bean注解的作用与XML配置文件中的`<bean/>`元素作用相同。您可以在任何Spring @Component中使用@Bean注解的方法。然而，它们最常被用于@Configuration beans。

@Configuration注解的类表明它的主要目的是作为bean定义的来源。此外，@Configuration类还允许通过调用同一类中的其他@Bean方法来定义bean间的依赖关系。最简单的@Configuration类的内容如下:

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

**完整的@Configuration与 "精简 "的@Bean模式？**

当@Bean方法被声明在没有注解@Configuration的类中时，它们被称为以 "精简 "模式处理。在@Component甚至在一个普通的类中声明的Bean方法被认为是 "精简 "的，包含类的主要目的不同，@Bean方法在那里是一种奖励。例如，服务组件可能会通过在每个适用的组件类上额外的@Bean方法向容器暴露管理视图。在这样的场景下，@Bean方法是一种通用的工厂方法机制。

与完整的@Configuration不同，精简的@Bean方法不能声明bean间的依赖关系。相反，它们是在它们所包含的组件的内部状态上进行操作，并且可以选择在它们可能声明的参数上进行操作。因此，这样的@Bean方法不应该调用其他@Bean方法。每个这样的方法从字面上看只是一个特定bean引用的工厂方法，没有任何特殊的运行时语义。这里的积极副作用是，无需在运行时应用CGLIB子类，因此在类设计方面没有限制（即包含的类可以是final等）。

在常见的场景中，@Bean方法要在@Configuration类中声明，确保始终使用 "完整 "模式，因此跨方法引用会被重定向到容器的生命周期管理中。这样可以防止同一个@Bean方法通过常规的Java调用被意外调用，这有助于减少在 "精简 "模式下运行时难以追踪的细微bug。

## 1.12.2 通过AnnotationConfigApplicationContext实例化spring容器

下面的章节记录了Spring在Spring 3.0中引入的`AnnotationConfigApplicationContext`。这个多功能的`ApplicationContext`实现不仅能接受@Configuration类作为输入，还能接受普通的`@Component`类和用JSR-330元数据注释的类。

当`@Configuration`类被提供为输入时，`@Configuration`类本身被注册为bean定义，并且该类中所有声明的`@Bean`方法也被注册为bean定义。

当提供`@Component`和JSR-330类时，它们被注册为bean定义，并且假定在这些类中必要时使用了DI元数据，如@Autowired或@Inject。

**简单的实例化**

与实例化ClassPathXmlApplicationContext时使用Spring XML文件作为输入一样，您可以在实例化AnnotationConfigApplicationContext时使用@Configuration类作为输入。这就允许完全不使用XML的Spring容器，如下例所示：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

如前所述，AnnotationConfigApplicationContext并不限于只与@Configuration类一起工作。任何@Component或JSR-330注解类都可以作为输入提供给构造函数，如下例所示:

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

**使用`register(Class<?>…)`编程构建容器**

您可以使用无参数构造函数实例化一个AnnotationConfigApplicationContext，然后使用register()方法配置它。当以编程方式构建AnnotationConfigApplicationContext时，这种方法特别有用。下面的示例展示了如何做到这一点：

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

**`scan(String…)`开启包扫描**

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}
```

在前面的示例中，`com.acme`包被扫描以查找任何`@Component-annotated`类，这些类被注册为容器中的Spring bean定义。`AnnotationConfigApplicationContext`公开了`scan(String...)`方法，以允许同样的组件扫描功能，如下例所示：

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

请记住，`@Configuration`类是用`@Component`进行元注解的，所以它们是组件扫描的候选对象。在前面的例子中，假设AppConfig是在`com.acme`包（或下面的任何包）中声明的，那么在调用scan()的过程中，它就会被选中。一旦刷新()，它的所有@Bean方法都会被处理并在容器中注册为bean定义。

## 1.12.3 @Bean的使用

@Bean是个方法级别的注解，和XML中的 `<bean/>` 相同作用。这个注解也支持 \* [init-method](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) * [destroy-method](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) * [autowiring](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) *属性。

### Declaring a Bean

声明一个Bean，可以在方法上使用@Bean注解，方法的返回值就是Bean的类型。默认的，**Bean 的name就是方法名**！如：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

当然也可以声明返回接口

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

类似于

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

 Both declarations make a bean named `transferService` available in the `ApplicationContext`, bound to an object instance of type `TransferServiceImpl`, 如下:

```
transferService -> com.acme.TransferServiceImpl
```

### Bean的依赖

一个Bean注解的方法可以有任意数量的参数，这些参数描述了构建该Bean所需的依赖。例如如果我们的TransferService需要一个AccountRepository，我们可以用方法参数来具体化这个依赖关系，如下例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

上面的解决机制和基于构造的依赖差不多。

### 接受生命周期的回调

任何使用 @Bean 注解定义的类都支持常见的生命周期回调，并且可以使用 JSR-250 中的 `@PostConstruct` 和 `@PreDestroy` 注解。更多细节请参见 [JSR-250 annotations](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)。

常规的Spring生命周期回调也是完全支持的。如果一个bean实现了`InitializingBean`, `DisposableBean`或`Lifecycle`，它们各自的方法就会被容器调用。

标准的`*Aware`接口集（如BeanFactoryAware、BeanNameAware、MessageSourceAware、ApplicationContextAware等）也是完全支持的。

@Bean注解支持指定任意的初始化和销毁的回调方法，就像Spring XML在bean元素上的init-method和destroy-method属性一样，如下例所示。

```java
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

默认情况下，用 Java 配置的具有公共`close`或`shutdown`方法的 bean 会自动地被征集一个销毁回调。如果您有一个公共的关闭或关闭方法，并且您不希望它在容器关闭时被调用，您可以将@Bean(destroyMethod="")添加到您的bean定义中以禁用默认（推断）模式。

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws NamingException {
    return (DataSource) jndiTemplate.lookup("MyDS");
}
```

直接在code中调用`init()`也是可以的

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        BeanOne beanOne = new BeanOne();
        beanOne.init();
        return beanOne;
    }

    // ...
}
```

当你直接通过Java代码工作时，你可以对你的对象做任何你喜欢的事情，而不是总是需要依赖容器的生命周期。

### 指定Bean的scope

Bean的默认scope是 `singleton`,但是可以使用 `@Scope` 指定

```java
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

### 自定义Bean的名称

使用name属性可以指定Bean的名称，默认Bean的名称是方法名。

```java
@Configuration
public class AppConfig {

    @Bean(name = "myThing")
    public Thing thing() {
        return new Thing();
    }
}
```

```java
@Configuration
public class AppConfig {

    // 可以指定多个别名 Aliasing
    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

```java
@Configuration
public class AppConfig {

    // 对Bean进行描述，在一些如JMX连接时可以提供更多信息
    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```

## 1.12.4  `@Configuration`注解的使用

@Configuration是类级别的注解，表示该是Bean定义的源。`@Configuration`类内部通过@Bean注解的public方法来声明Bean，`@Configuration`类中的@Bean方法也可以用来声明Bean之间的依赖关系。

### Bean之间的依赖表达

当Bean依赖另一个Bean时，只需要简单的调用另一个Bean方法即可，举例如下：

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        // 通过beanTwo()注入BeanTwo
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

这种通过方法来声明依赖Bean的方式只能在@Configuration的类中使用，不能用于@Component注解的类中。

### Lookup Method Injection

如前所述，查找方法注入是一个高级功能，你应该很少使用。在单例Bean依赖原型Bean的情况下，它很有用。使用Java进行这种类型的配置，为实现这种模式提供了一种自然的手段。下面举例：

```java
public abstract class CommandManager {
    public Object process(Object commandState) {
        // 获取一个合适的Command实例
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // 但是谁来实现这个方法呢？
    protected abstract Command createCommand();
}
```
通过使用Java配置，您可以创建CommandManager的子类，其中抽象的createCommand()方法被重载，以使其查找一个新的（原型）命令对象。

```java
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with createCommand()
    // overridden to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

### 进一步探究Java-based的配置工作原理

考虑下面一种场景，一个@Bean方法被调用了两次：

```java
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

ClientDao()已经在clientService1()和clientService2()中被调用过一次。由于该方法创建了一个新的 ClientDaoImpl 实例并将其返回，您通常会期望有两个实例（每个服务一个）。这肯定有问题，在Spring中，实例化的Bean默认是单例的。这就是神奇的地方，所有的@Configuration类在启动时都会被CGLIB实现子类。在子类中，重写的方法首先会检查容器中是否有缓存的Bean，没有才会去调用父方法创建一个新的实例。

这里有一些限制，因为CGLIB是在运行时动态的添加功能，所以配置类不能final。然而从Spring4.3开始，配置类中可以使用任何构造方法，包括使用`@Autowired`或者非默认的构造来进行默认注入。

如果你更倾向于避免CGLIB施加的限制，你可以在非`@Configuration`注解的类中使用@Bean（如使用@Component代理)，这样，@Bean方法之间的交叉调用就不会被拦截，你可以完全依赖构造或方法级别的注入。

## 1.12.5 Java-based配置的组合使用

Spring’s Java-based configuration允许你组合注解，减少配置的复杂性。

### 使用@Import注解

这个注解允许你从其他配置类加载@Bean定义，如：

```java
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

现在，你不需要在实例化容器的时候同时指定 `ConfigA.class` 和`ConfigB.class` ，只需要明确的指定ConfigB即可，

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

这种方式简化了容器实例化，你只需要处理一个配置类，而不是记住所有的配置类。

从Spring4.2，@Import也支持导入一个普通的Bean，类似`AnnotationConfigApplicationContext.register`方法，这在你想避免component scanning，只需要很少的配置类来明确定义你所有的Components。

**在导入的@Bean定义上注入依赖关系**

前面的例子是可行的，但很简单。在大多数实际场景中，bean在不同的配置类中都有相互依赖关系。当使用XML时，这不是问题，因为不涉及编译器，你可以声明`ref="someBean"`，并相信Spring会在容器初始化期间解决它。当使用`@Configuration`类时，Java编译器会对配置模型进行约束，即对其他bean的引用必须是有效的Java语法。

幸运的是，解决这个问题很简单。正如我们已经讨论过的，一个`@Bean`方法可以有任意数量的参数来描述bean的依赖关系。考虑以下更真实的场景，有几个`@Configuration`类，每个类都依赖于其他类中声明的bean。

```java
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

还有另一种方法可以达到同样的效果。请记住，@Configuration类最终只是容器中的另一个Bean。这意味着它们可以像其他bean一样利用@Autowired和@Value注入以及其他功能。

确保你以这种方式注入的依赖关系只是最简单的那种。在上下文的初始化过程中，@Configuration类的处理时间相当早，强迫以这种方式注入依赖关系可能会导致意外的早期初始化。只要有可能，就像前面的例子一样，采用基于参数的注入方式。

另外，对于通过@Bean定义的BeanPostProcessor和BeanFactoryPostProcessor要特别小心。这些通常应该被声明为静态的@Bean方法，而不是触发其包含配置类的实例化。否则，@Autowired和@Value可能对配置类本身不起作用，因为它有可能比AutowiredAnnotationBeanPostProcessor更早地创建它作为bean实例。

下面的例子展示了如何将一个Bean自动注入到另一个Bean：

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
在@Configuration类中的构造函数注入只在Spring Framework 4.3中支持。还需要注意的是，**如果目标Bean只定义了一个构造函数，就不需要指定@Autowired**。

**完全合格的beans ，方便导航**

在前面的方案中，使用@Autowired效果很好，并且提供了所需的模块化，但是确定自动生成的bean定义到底是在哪里声明的，还是有些模糊。例如，作为一个负责`ServiceConfig`的开发人员，你怎么知道`@Autowired AccountRepository` bean到底是在哪里声明的？在代码中并没有明确，这可能就好办了。请记住，Spring Tools for Eclipse提供的工具可以呈现出显示所有东西如何连接的图表，这可能是你所需要的全部。此外，你的Java IDE可以很容易地找到AccountRepository类型的所有声明和使用，并迅速向你展示返回该类型的@Bean方法的位置。

在不能接受这种模棱两可的情况下，如果您希望在您的IDE内从一个@Configuration类直接导航到另一个@Configuration类，请考虑对配置类本身进行自动布线。下面的例子展示了如何做到这一点：

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```

在前面的情况下，AccountRepository的定义是完全明确的。然而，ServiceConfig现在与RepositoryConfig紧密耦合。这就是权衡。通过使用基于接口或基于抽象类的@Configuration类，可以在一定程度上缓解这种紧耦合。考虑下面的例子:

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

现在ServiceConfig与具体的DefaultRepositoryConfig松散耦合，内置的IDE工具仍然有用。你可以很容易地得到RepositoryConfig实现的类型层次结构。这样一来，导航@Configuration类和它们的依赖关系就变得和通常导航基于接口的代码的过程没有什么不同。

如果你想影响某些bean的启动创建顺序，可以考虑将其中的一些bean声明为@Lazy（用于在首次访问时创建，而不是在启动时创建），或者声明为@DependsOn某些其他bean（确保特定的其他bean在当前bean之前创建，超出后者的直接依赖关系意味着什么）。

### **有条件地包含@Configuration类或@Bean方法**

基于一些任意的系统状态，有条件地启用或禁用一个完整的@Configuration类，甚至是单个的@Bean方法，通常是很有用的。一个常见的例子是使用@Profile注解仅在Spring环境中启用了特定的配置文件时才激活bean（详见 [Bean Definition Profiles](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles)）。

实际上，@Profile 注解是通过使用名为 `@Conditional` 这个更灵活的注解来实现的。@Conditional注解指示了在注册@Bean之前应该参考的特定`org.springframework.context.annotation.Condition`实现。

Condition接口的实现提供了一个match(...)方法，该方法返回true或false。例如，下面的列表显示了实际用于@Profile的Condition实现:

```java
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    // Read the @Profile annotation attributes
    MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
    if (attrs != null) {
        for (Object value : attrs.get("value")) {
            if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                return true;
            }
        }
        return false;
    }
    return true;
}
```

### Combining Java and XML Configuration

在 @Configuration 类是配置容器的主要机制的应用程序中，仍然可能需要使用至少一些 XML。在这些情况下，您可以使用 @ImportResource 并只定义您需要的 XML。这样做实现了一种 "以Java为中心 "的配置容器的方法，并将XML保持在最低限度。
