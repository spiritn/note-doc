

# 1.1 IoC容器和Beans

> This chapter covers the Spring Framework implementation of the Inversion of Control (IoC) principle. IoC is also known as dependency injection (DI). It is a process whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.

本章介绍了spring框架IOC容器实现的原理。IoC也被称为依赖注入DI。它是一个过程，对象仅通过构造函数、工厂方法或对象实例被构造或从工厂方法返回后在其上设置的属性来定义它们的依赖关系（即它们工作的其他对象），然后容器在创建bean时注入这些依赖。这个过程从根本上说是反过来的（故名Inversion of Control），即bean本身通过直接构造类或者服务Locator 模式机制来控制依赖的实例或位置。



 `org.springframework.beans` 和`org.springframework.context` 包构成了spring框架IoC容器的基础。  [BeanFactory](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/beans/factory/BeanFactory.html) 接口提供了高级的配置机制来管理任何类型的bean. [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/context/ApplicationContext.html)是 BeanFactory的子类.，增加了:

- 和Spring  AOP更简单的集成
- Message resource handling (用于国际化)
- 多种event发布
- 应用层特定的上下文，如 `WebApplicationContext` 用于web应用.

总而言之,  `BeanFactory` 提供了框架配置和基本功能, 而 `ApplicationContext` 增加了更多的企业功能。 `ApplicationContext`  是`BeanFactory` 的完整超集，是本章专门描述的spring IoC容器。如果要使用 `BeanFactory` 代替 `ApplicationContext,` 查看[The `BeanFactory`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanfactory)。

> In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and managed by a Spring IoC container. Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.

在spring中，构成应用主干并由spring IoC容器管理的对象称为Bean。Bean是由Spring IoC容器负责实例化，组装和管理的对象，否则，Bean只是您应用中众多对象中的一个。Bean以及他们之间的依赖，都反映在容器使用的配置元数据中。

# 1.2 容器

> The `org.springframework.context.ApplicationContext` interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata. The configuration metadata is represented in XML, Java annotations, or Java code. It lets you express the objects that compose your application and the rich interdependencies between those objects
>

 `org.springframework.context.ApplicationContext`接口代表了Spring IoC容器，并负责实例化，配置和组装Bean。容器通过读取配置元数据，来获取关于实例化，配置和组装哪些对象的指令。这里的配置元数据用XML，Java注解和Java代码表示。可以让你表达你应用中的对象和这些对象间的丰富依赖关系。

Spring提供了ApplicationContext接口的几种实现。在独立的应用程序中，通常创建一个ClassPathXmlApplicationContext或FileSystemXmlApplicationContext的实例。虽然 XML 一直是定义配置元数据的传统格式，但您可以通过提供少量的 XML 配置来声明性地启用对这些附加元数据格式的支持，从而指示容器使用 Java 注解或代码作为元数据格式。

在大多数应用场景中，不需要显式的通过用户代码来实例化Spring IoC容器的一个或多个实例。例如，在 Web 应用程序场景中，应用程序的 web.xml 文件中的简单八行（或左右）的模板 Web 描述符 XML 通常就足够了（请参阅 Web 应用程序的便捷 ApplicationContext 实例化）。如果您使用Spring Tools for Eclipse（一个Eclipse支持的开发环境），您可以通过点击几下鼠标或击键轻松创建这个模板配置。

> The following diagram shows a high-level view of how Spring works. Your application classes are combined with configuration metadata so that, after the `ApplicationContext` is created and initialized, you have a fully configured and executable system or application.

下图显示了Spring工作方式的高级视图。你的应用类与配置元数据相结合，这样，在ApplicationContext被创建和初始化后，你就拥有了一个完全配置好的可执行系统或应用。

![container magic](https://docs.spring.io/spring-framework/docs/current/reference/html/images/container-magic.png)

## 1.2.1 配置元数据

如上图所示，Spring IoC容器接受配置元数据。该配置元数据代表了您作为应用开发人员如何告诉Spring容器实例化，配置和组装应用中的对象。

配置元数据传统上是以简单直观的XML格式提供的，本章的大部分内容就用它来传达Spring IoC容器的关键概念和特性。

基于XML的元数据配置并不是唯一的形式，Spring IoC容器本身与这些配置元数据实际写入格式完全隔离。如今，许多开发者选择了基于Java的配置方式。

有关在spring中使用其他形式的元数据，请查看：

- [1.9 Annotation-based configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config): Spring 2.5引入的，通过注解配置元数据
- [1.12 Java-based configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java): 从Spring3.0开始，可以通过Java代码而不是xml文件来定义你应用的类。更多信息查看 [`@Configuration`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html), [`@Bean`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html), [`@Import`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html), and [`@DependsOn`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html) 注解。

spring 配置由容器必须管理的通常不止一个 bean 定义组成。基于 XML 的配置元数据将这些 bean 配置为顶层 `<beans/>` 元素中的 `<bean/>` 元素。Java 配置通常在 @Configuration 类中使用 @Bean 注释的方法。

这些bean定义对应于构成你的应用程序的实际对象。通常情况下，你会定义服务层对象、数据访问对象（DAO）、表现对象（如Struts Action实例）、基础设施对象（如Hibernate SessionFactories、JMS队列等）等。通常情况下，人们不会在容器中配置细粒度的域对象，因为创建和加载域对象通常是DAO和业务逻辑的责任。然而，您可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。请参阅使用 AspectJ 与 Spring 进行依赖注入域对象。

## 1.2.3 使用容器

ApplicationContext是一个高级工厂的接口，它能够维护不同Bean及其依赖关系的注册表。通过使用T `getBean(String name, Class<T> requiredType)`方法，你可以检索你的bean的实例。

ApplicationContext允许你读取Bean定义和获取他们，如：

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

最灵活的变体是 `GenericApplicationContext`和读取代理相结合，例如与XML文件的XmlBeanDefinitionReader相结合，如：

```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

你可以在同一个ApplicationContext上混合和匹配这样的读取代理，从不同的配置源读取Bean定义。

然后你可以使用getBean()来检索Bean。ApplicationContext接口还有其他一些方法用于检索Bean，但是，理想情况下，你的应用程序代码不应该使用它们。事实上，你的应用程序代码根本不应该调用getBean()方法，这样对Spring API完全没有依赖性。例如，Spring与Web框架的集成为各种Web框架组件（如控制器和JSF管理的Bean）提供了依赖注入，让你通过元数据（如autowiring注解）声明对特定Bean的依赖。

# 1.3 Bean

Spring容器管理一个或多个Bean。这些Bean通过配置元数据应用到容器中（例如XMl中的 `<bean/>` 定义）。

> Within the container itself, these bean definitions are represented as `BeanDefinition` objects, which contain (among other information) the following metadata:
>
> - A package-qualified class name: typically, the actual implementation class of the bean being defined.
> - Bean behavioral configuration elements, which state how the bean should behave in the container (scope, lifecycle callbacks, and so forth).
> - References to other beans that are needed for the bean to do its work. These references are also called collaborators or dependencies.
> - Other configuration settings to set in the newly created object — for example, the size limit of the pool or the number of connections to use in a bean that manages a connection pool.

在容器本身，这些Bean定义被表示为BeanDefinition对象，它包含以下(除了其他信息)元数据

- 包限定的类名: 通常是被定义的Bean的实际实现类.
- Bean的行为配置，它说明了Bean在容器中应该表现的状态(scope, lifecycle callbacks等)
- 对其他Bean的引用，这些引用是Bean工作所需要的，也被成为依赖
- 要在新创建的对象中设置的其他配置元素。例如，池的大小，Bean要管理的连接池的连接数等。也就是属性吧

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

> In addition to bean definitions that contain information on how to create a specific bean, the `ApplicationContext` implementations also permit the registration of existing objects that are created outside the container (by users). This is done by accessing the ApplicationContext’s BeanFactory through the `getBeanFactory()` method, which returns the BeanFactory `DefaultListableBeanFactory` implementation. `DefaultListableBeanFactory` supports this registration through the `registerSingleton(..)` and `registerBeanDefinition(..)` methods. However, typical applications work solely with beans defined through regular bean definition metadata.
>
> Bean metadata and manually supplied singleton instances need to be registered as early as possible, in order for the container to properly reason about them during autowiring and other introspection steps. While overriding existing metadata and existing singleton instances is supported to some degree, the registration of new beans at runtime (concurrently with live access to the factory) is not officially supported and may lead to concurrent access exceptions, inconsistent state in the bean container, or both.

除了包含如何创建特定Bean信息的Bean定义之外，ApplicationContext实现还允许注册在容器之外（由用户）创建的现有对象。这是通过getBeanFactory()方法访问ApplicationContext的BeanFactory来实现的，该方法返回BeanFactory的DefaultListableBeanFactory实现。DefaultListableBeanFactory通过registerSingleton(..)和registerBeanDefinition(..)方法支持这种注册。然而，典型的应用程序只使用通过常规bean定义元数据定义的bean。

Bean元数据和手动提供的单例需要尽可能早地注册，以便容器可以在自动注入和其他set设置时可以正确的推理。虽然在一定程度上支持覆盖现有的元数据和单例，但是官方并不支持在运行时注册新的bean（和工厂的访问同时进行），并且可能导致并发访问异常，bean状态不一致等。

## 1.3.1 Bean的命名

> Every bean has one or more identifiers. These identifiers must be unique within the container that hosts the bean. A bean usually has only one identifier. However, if it requires more than one, the extra ones can be considered aliases.
>
> In XML-based configuration metadata, you use the `id` attribute, the `name` attribute, or both to specify the bean identifiers. The `id` attribute lets you specify exactly one id. Conventionally, these names are alphanumeric ('myBean', 'someService', etc.), but they can contain special characters as well. If you want to introduce other aliases for the bean, you can also specify them in the `name` attribute, separated by a comma (`,`), semicolon (`;`), or white space. As a historical note, in versions prior to Spring 3.1, the `id` attribute was defined as an `xsd:ID` type, which constrained possible characters. As of 3.1, it is defined as an `xsd:string` type. Note that bean `id` uniqueness is still enforced by the container, though no longer by XML parsers.
>
> You are not required to supply a `name` or an `id` for a bean. If you do not supply a `name` or `id` explicitly, the container generates a unique name for that bean. However, if you want to refer to that bean by name, through the use of the `ref` element or a Service Locator style lookup, you must provide a name. Motivations for not supplying a name are related to using [inner beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-inner-beans) and [autowiring collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire).

每个Bean都有一个或多个标识符。这些标识符在对应的容器中必须是唯一的。一个Bean通常只有一个标识符。然而，如果它需要多个标识符，那么额外的标识符会被认为是别名。

在基于XML的元数据配置中，可以使用id，name属性，或者同时使用，来指定Bean的标识符。id属性可以让你精确的指定一个Bean。传统上，这些名称可以是字母数字（'myBean'、'someService'等），但也可以包含特殊字符。如果你想为Bean引入其他别名，也可以在name中指定，用逗号，分号或空格分隔。作为历史性的说明，在Spring 3.1之前的版本中，id属性被定义为xsd:ID类型，它限制了可能的字符。从3.1开始，它被定义为xsd:string类型。请注意，bean id 唯一性仍然由容器强制执行，尽管不再由 XML 解析器执行。

您不需要为Bean提供name或id，如果你不显示的提供name或id，容器就会为该Bean生成一个唯一的name。但是，如果你想通过使用ref元素来用name引用该Bean，您必须提供一个name。不提供名称的动机与使用内部Bean和自动注入合作者有关。



在命名bean时惯例使用标准的Java惯例来命名实例字段名。也就是说，bean名称以小写字母开头，驼峰表示。如accountManager、accountService、userDao、loginController等。

统一命名bean可以让你的配置更容易阅读和理解。此外，如果你使用Spring AOP，当你对一组名字相关的bean应用建议时，它有很大的帮助。

通过classpath中的组件扫描，Spring为未命名的组件生成bean名称，遵循前面描述的规则：基本上，取简单的类名，并将其首字母变成小写。然而，在特殊情况下，当有多个字符，并且第一个和第二个字符都是大写时，原始的格式会被保留。这些规则与java.beans.Introspector.decapitalize（Spring在这里使用）定义的规则相同。

### 在Bean定义之外为Bean取别名

> In a bean definition itself, you can supply more than one name for the bean, by using a combination of up to one name specified by the id attribute and any number of other names in the name attribute. These names can be equivalent aliases to the same bean and are useful for some situations, such as letting each component in an application refer to a common dependency by using a bean name that is specific to that component itself.
>
> Specifying all aliases where the bean is actually defined is not always adequate, however. It is sometimes desirable to introduce an alias for a bean that is defined elsewhere. This is commonly the case in large systems where configuration is split amongst each subsystem, with each subsystem having its own set of object definitions. In XML-based configuration metadata, you can use the <alias/> element to accomplish this. The following example shows how to do so:

在Bean定义中，您可以为Bean提供一个以上的名称，使用由id属性指定的最多一个名称和使用name属性指定任意数量的其他名称。这些名称可以是同一个Bean的等价别名，对于某些情况来说是很有用的，比如让应用程序中的每个组件使用一个特定于该组件本身的Bean名称来引用一个共同的依赖关系。

然而，在实际定义Bean的地方指定所有别名并不总是足够的。有时，我们需要为在其他地方定义的Bean引入一个别名。这种情况在大型系统中很常见，因为在这些系统中，配置被分割在每个子系统中，每个子系统都有自己的对象定义集。在基于XML的配置元数据中，您可以使用<alias/>元素来实现这一目的。下面的例子展示了如何做到这一点。

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

## 1.3.2 实例化Bean

> A bean definition is essentially a recipe for creating one or more objects. The container looks at the recipe for a named bean when asked and uses the configuration metadata encapsulated by that bean definition to create (or acquire) an actual object.
>

Bean definition 本质上是创建一个或多个对象的配方（食谱）。容器在被请求时查看Bean的配方，并使用该Bean定义封装的配置元数据来创建（或获取）一个实际的对象。

如果您使用基于 XML 的配置元数据，您可以在 `<bean/>` 元素的 class 属性中指定要实例化的对象的类型（或类）。这个类属性（在内部，它是BeanDefinition实例上的一个Class属性）通常是强制性的。（关于例外情况，请参见[Instantiation by Using an Instance Factory Method](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-instance-factory-method) and [Bean Definition Inheritance](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-child-bean-definitions).）。您可以用两种方式之一来使用Class属性。

- 通常，在容器本身通过反射调用其构造函数地直接创建bean的情况下，指定要构造的bean的class，有点相当于使用new操作符的Java代码。
- 在容器调用类的静态工厂方法来创建bean的情况下，要指定包含静态工厂方法的实际类，这种情况不太常见。从静态工厂方法的调用中返回的对象类型可能是同一个类，也可能完全是另一个类。

### 通过构造器来实例化Bean

> When you create a bean by the constructor approach, all normal classes are usable by and compatible with Spring. That is, the class being developed does not need to implement any specific interfaces or to be coded in a specific fashion. Simply specifying the bean class should suffice. However, depending on what type of IoC you use for that specific bean, you may need a default (empty) constructor.
>
> The Spring IoC container can manage virtually any class you want it to manage. It is not limited to managing true JavaBeans. Most Spring users prefer actual JavaBeans with only a default (no-argument) constructor and appropriate setters and getters modeled after the properties in the container. You can also have more exotic non-bean-style classes in your container. If, for example, you need to use a legacy connection pool that absolutely does not adhere to the JavaBean specification, Spring can manage it as well.

当你通过构造函数的方法创建一个bean时，所有的普通类都可以被Spring使用，并且与Spring兼容。也就是说，被开发的类不需要实现任何特定的接口，也不需要以特定的方式进行编码。简单地指定bean类就应该足够了。然而，根据您为该特定Bean使用的IoC类型，您可能需要一个默认（空）构造函数。

Spring IoC容器几乎可以管理您希望它管理的任何类。它不限于管理真正的JavaBeans。大多数Spring用户更喜欢实际的JavaBeans，只需要一个默认的（无参）构造函数和适当的setters 和getters ，以容器中的属性为模型。你也可以在你的容器中拥有更多奇特的非bean风格的类。例如，如果你需要使用一个绝对不遵守JavaBean规范的传统连接池，Spring也可以管理它。

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

类似于通过静态工厂方法进行实例化，使用实例工厂方法进行实例化会从容器中调用现有bean的非静态方法来创建新bean。 要使用此机制，请将class属性保留为空，并在factory-bean属性中，在当前（或父或祖先）容器中指定包含要创建该对象的实例方法的bean的名称。 使用factory-method属性设置工厂方法本身的名称。 以下示例显示了如何配置此类Bean：

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

> The runtime type of a specific bean is non-trivial to determine. A specified class in the bean metadata definition is just an initial class reference, potentially combined with a declared factory method or being a `FactoryBean` class which may lead to a different runtime type of the bean, or not being set at all in case of an instance-level factory method (which is resolved via the specified `factory-bean` name instead). Additionally, AOP proxying may wrap a bean instance with an interface-based proxy with limited exposure of the target bean’s actual type (just its implemented interfaces).
>
> The recommended way to find out about the actual runtime type of a particular bean is a `BeanFactory.getType` call for the specified bean name. This takes all of the above cases into account and returns the type of object that a `BeanFactory.getBean` call is going to return for the same bean name.

确定特定bean的运行时类型并非易事。 Bean元数据定义中的指定类只是初始类引用，可能与声明的工厂方法结合使用，或者是FactoryBean类，这可能导致Bean的运行时类型不同，或者在实例的情况下完全不进行设置 级工厂方法（通过指定的factory-bean名称解析）。 此外，AOP代理可以使用基于接口的代理包装bean实例，而目标Bean的实际类型（仅是其实现的接口）的暴露程度有限。

找出特定bean的实际运行时类型的推荐方法是对指定bean名称的BeanFactory.getType调用。 这考虑了上述所有情况，并返回了针对相同bean名称的BeanFactory.getBean调用将返回的对象的类型。

## 附录：Bean 的生命周期

![Bean生命周期](images/Bean生命周期.png)

Spring统一管理Bean的生命周期

1. Bean的实例化与属性注入

   在第一次调用getBean()时生成Bean的实例，通常通过反射或者CGLIB动态字节码来生成Bean实例或子类。但是并不是直接返回构造完成的对象实例，而是进行包裹返回相应的BeanWrapper实例，这样的好处是可以在接下来非常方便的设置对象的属性（不然如果直接返回实例对象，那么你要通过反射的API才能去增加对象的属性，很麻烦），而且BeanWrapper继承了PropertyAccessor接口可以方便的获取对象属性，也继承了PropertyEditorRegister和TypeConverter接口，可以方便转换属性类型。

2. 检查设置Aware接口

3. BeanPostprocessor的before和after方法,和中间的InitializingBean和init-method方法

4. 检查Singleton的Bean实例，是否实现了DisposableBean接口，注册对象销毁时的回调。一般是在Spring容器关闭的时候调用Bean的销毁方法，但是好像要手动调用applicationcontext.destroySingleton()才行。


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

## 1.6.2  `ApplicationContextAware` 和`BeanNameAware`

当ApplicationContext创建一个实现org.springframework.context.ApplicationContextAware接口的对象实例时，该实例会被提供一个对该ApplicationContext的引用。 `ApplicationContextAware` 定义如下：

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

因此，Bean可以获取`ApplicationContext` ，或者是子类`ConfigurableApplicationContext`（他暴露了额外的功能），进行编程操作。一个用途是检索其他Bean。有时这些功能是有用的，然而通常情况下，应该尽量避免使用它，因为它将代码耦合到Spring，而且也不遵守反转控制的风格，应该是依赖作为属性提供给Bean。`ApplicationContext`的其他方法提供了对文件资源的访问，发布event和访问MessageSource。这些附加功能介绍详见[Additional Capabilities of the `ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction)。

自动注入是获取`ApplicationContext`的另一种选择。传统的构造和按类型注入可以提供的`ApplicationContext`的依赖。为了获得更多的灵活性，包含自动注入字段和多个参数的方法的能力，请使用基于注解的自动注入特性。如果您这样做，`ApplicationContext`会自动装配到字段，构造参数或者方法中。

举例：

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



当`ApplicationContext` 创建一个实现`org.springframework.beans.factory.BeanNameAware` 接口的实现类时，该类会提供一个name定义关联的对象，下面是`BeanNameAware` 的定义

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

> The callback is invoked after population of normal bean properties but before an initialization callback such as `InitializingBean`, `afterPropertiesSet`, or a custom init-method.

这个接口的回调在bean属性生成之后，但是在初始化方法（ `InitializingBean`.`afterPropertiesSet`, 或者自定义的init-method）回调之前。

## 1.6.3 其他Aware接口

除了之前讨论的 `ApplicationContextAware` 和`BeanNameAware` ，Spring还提供了很多其他Aware回调接口，让Bean向容器表明他们需要某个基础依赖。一般情况下，name表示依赖类型，下面展示一些最重要的Aware接口：

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

# 1.8 容器扩展点

通常情况下，应用开发者不需要实现ApplicationContext。相反，可以通过插入特殊集成接口的实现来扩展Spring IoC容器。接下来的几节将介绍这些集成接口。

## 1.8.1 使用`BeanPostProcessor`自定义Bean

 `BeanPostProcessor`接口定义了回调方法，你可以据此来提供自己的（或者覆盖容器的）实例化逻辑，依赖解析等。如果你想要在Spring容器完成实例化、配置和初始化Bean之后实现一些自定义逻辑，你可以插入一个或多个自定义的 `BeanPostProcessor`实现。

> 如Apollo源码里，实现了BeanPostProcessor接口，解析每个Bean中@Value的字段为SpringValue对象，在配置文件更新后反射更新其值。

您可以配置多个 `BeanPostProcessor`实现，并且可以通过设置Order属性来控制这些实现的执行顺序。只有当 `BeanPostProcessor`实现了Order接口，才可以设置这个属性。如果你写了自己的 `BeanPostProcessor`，应该考虑实现Order接口。

> `BeanPostProcessor` instances operate on bean (or object) instances. That is, the Spring IoC container instantiates a bean instance and then `BeanPostProcessor` instances do their work.
>
> To change the actual bean definition (that is, the blueprint that defines the bean), you instead need to use a `BeanFactoryPostProcessor`, as described in [Customizing Configuration Metadata with a `BeanFactoryPostProcessor`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-factory-postprocessors).

`BeanPostProcessor` 操作一个bean的实例。也就是说，Spring IOC容器是先实例化一个Bean，然后`BeanPostProcessor` 才会进行他们的工作。

`BeanPostProcessor` 实例的作用于是容器级别的。这只有使用容器层级结构有关。如果你在一个容器中定义了一个BeanPostProcessor，它只对该容器中的bean进行后处理。换句话说，在一个容器中定义的 bean 不会被另一个容器中定义的 BeanPostProcessor 后处理，即使这两个容器是同一层次结构的一部分。

要更改实际的 bean 定义（即定义 bean 的蓝图），则需要使用 BeanFactoryPostProcessor。

> The `org.springframework.beans.factory.config.BeanPostProcessor` interface consists of exactly two callback methods. When such a class is registered as a post-processor with the container, for each bean instance that is created by the container, the post-processor gets a callback from the container both before container initialization methods (such as `InitializingBean.afterPropertiesSet()` or any declared `init` method) are called, and after any bean initialization callbacks. The post-processor can take any action with the bean instance, including ignoring the callback completely. A bean post-processor typically checks for callback interfaces, or it may wrap a bean with a proxy. **Some Spring AOP infrastructure classes are implemented as bean post-processors in order to provide proxy-wrapping logic.**

`org.springframework.beans.factory.config.BeanPostProcessor`接口正好由两个回调方法组成。当这样的类被容器注册为后处理器时，对于容器创建的每一个 bean 实例，后处理器都会在容器初始化方法（如 `InitializingBean.afterPropertiesSet()`或任何声明的 init 方法）被调用之前，以及在任何 bean 初始化回调之后，从容器获得一个回调。后处理器可以对bean实例采取任何操作，包括完全忽略回调。bean后处理器通常会检查回调接口，或者它可能会用代理类包装一个bean。**一些Spring AOP基础架构类就是作为bean后处理器的实现，以提供代理封装逻辑**。也就是Spring AOP基于此实现的？！



ApplicationContext 会自动检测配置元数据中定义的实现 BeanPostProcessor 接口的任何 Bean。ApplicationContext 将这些 Bean 注册为后处理器，以便它们可以在以后创建 Bean 时被调用。Bean 后处理器可以像其他 Bean 一样部署在容器中。

需要注意的是，在配置类上使用@Bean工厂方法声明BeanPostProcessor时，工厂方法的返回类型应该是实现类本身，或者至少是`org.springframework.beans.factory.config.BeanPostProcessor`接口，明确指出该Bean的后处理性质。否则，ApplicationContext在完全创建它之前，无法按类型自动检测它。由于BeanPostProcessor需要尽早实例化，才能应用于上下文中其他Bean的初始化，所以这个早期的类型检测非常关键。

> `BeanPostProcessor` instances and AOP auto-proxying
>
> Classes that implement the `BeanPostProcessor` interface are special and are treated differently by the container. All `BeanPostProcessor` instances and beans that they directly reference are instantiated on startup, as part of the special startup phase of the `ApplicationContext`. Next, all `BeanPostProcessor` instances are registered in a sorted fashion and applied to all further beans in the container. Because AOP auto-proxying is implemented as a `BeanPostProcessor` itself, neither `BeanPostProcessor` instances nor the beans they directly reference are eligible for auto-proxying and, thus, do not have aspects woven into them.
>
> For any such bean, you should see an informational log message: `Bean someBean is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying)`.
>
> If you have beans wired into your `BeanPostProcessor` by using autowiring or `@Resource` (which may fall back to autowiring), Spring might access unexpected beans when searching for type-matching dependency candidates and, therefore, make them ineligible for auto-proxying or other kinds of bean post-processing. For example, if you have a dependency annotated with `@Resource` where the field or setter name does not directly correspond to the declared name of a bean and no name attribute is used, Spring accesses other beans for matching them by type.

实现BeanPostProcessor接口的类是特殊的，被容器区别对待。所有BeanPostProcessor实例和它们直接引用的bean都在启动时被实例化，作为ApplicationContext的特殊启动阶段的一部分。接下来，所有BeanPostProcessor实例都会以排序的方式注册，并应用到容器中所有进一步的bean。**由于AOP自动代理是作为BeanPostProcessor本身来实现的**，所以BeanPostProcessor实例和它们直接引用的bean都没有资格进行自动代理，因此，它们没有切面织入。

对于任何这样的bean，你应该看到一个信息日志消息：` `Bean someBean is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying)`

如果您通过使用autowiring或@Resource（可能会回退到autowiring）将bean注入到BeanPostProcessor中，Spring在按类型搜索匹配的依赖候选者时可能会访问意外的bean，因此，导致它们不符合自动代理或其他类型的bean后处理的条件。例如，如果你有一个用@Resource注解的依赖，其中的字段并不直接对应于Bean的声明名称，也没有使用名称属性，那么Spring就会访问其他Bean来按类型匹配它们。



自定义例子：

```java
import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // 我们可以潜在的返回任何对象
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```



使用回调接口或注解与自定义BeanPostProcessor实现相结合，是扩展Spring IoC容器的一种常见手段。一个例子是 Spring 的 RequiredAnnotationBeanPostProcessor--一个随 Spring 发行版提供的 BeanPostProcessor 实现，它可以确保标有（任意）注解的 Bean 上的 JavaBean 属性实际上被（配置为）依赖注入一个值。

**像注解的处理和AOP代理都是通过BeanPostProcessor来实现的**！！即可以找出添加某个注解和实现某个接口的所有实现类，进行处理设置属性等。

> 如@Async，即在实现BeanPostProcessor的实现类里，检查如果添加@Async，就创建一个代理对象。
>
> 还有其他的隐含的注解处理器
>
> [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html),[`CommonAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html), [`PersistenceAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html), 和前面提到的[RequiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/annotation/RequiredAnnotationBeanPostProcessor.html)。

## 1.8.2  `BeanFactoryPostProcessor`自定义配置元数据

下一个扩展点是`org.springframework.beans.factory.config.BeanFactoryPostProcessor`。这个接口的语义与`BeanPostProcessor`类似，但是有一个重大的区别。`BeanFactoryPostProcessor`可以操作Bean的配置元数据。也就是说，Spring IOC容器允许`BeanFactoryPostProcessor`读取配置元数据，然后**在容器实例化其他Bean之前改变它**。

你可以配置多个BeanFactoryPostProcessor实例，你可以通过设置order属性来控制这些BeanFactoryPostProcessor实例的运行顺序。但是，只有当BeanFactoryPostProcessor实现了Ordered接口时，你才能设置这个属性。如果你写了自己的BeanFactoryPostProcessor，你也应该考虑实现Ordered接口。

如果你想改变实际的bean实例（即从配置元数据中创建的对象），那么你需要使用BeanPostProcessor（在前面的使用BeanPostProcessor自定义bean中描述）。虽然技术上可以在BeanFactoryPostProcessor中使用bean实例（例如使用BeanFactory.getBean()），但这样做会导致过早的bean实例化，违反了标准的容器生命周期。这可能会引起负面的副作用，比如绕过bean后处理。

另外，BeanFactoryPostProcessor实例的作用域是每个容器。这只有在你使用容器层次结构时才有意义。如果你在一个容器中定义了一个BeanFactoryPostProcessor，那么它只应用于该容器中的bean定义。一个容器中的 Bean 定义不会被另一个容器中的 BeanFactoryPostProcessor 实例进行后处理，即使这两个容器是同一层次结构的一部分。



当在ApplicationContext内声明一个bean工厂后处理器时，它就会自动运行，以便对定义容器的配置元数据应用更改。Spring包含了许多预定义的bean工厂后处理器，如`PropertyOverrideConfigurer`和`PropertySourcesPlaceholderConfigurer`。您也可以使用自定义的BeanFactoryPostProcessor--例如，注册自定义属性编辑器。

ApplicationContext 会自动检测部署到它的任何实现 BeanFactoryPostProcessor 接口的 Bean。它在适当的时候将这些bean用作bean工厂后处理器。你可以像部署其他bean一样部署这些后处理器bean。

与BeanPostProcessors一样，你通常不希望将BeanFactoryPostProcessors配置为懒惰初始化。如果没有其他 bean 引用 Bean(Factory)PostProcessor，那么这个后处理器将不会被实例化。因此，将它标记为懒惰初始化将被忽略，即使你在 `<beans />` 元素的声明中将 default-lazy-init 属性设置为 true，Bean(Factory)PostProcessor 也会被急切地实例化

### PropertySourcesPlaceholderConfigurer

通常我们不想将类似于密码等重要的配置信息混杂到XML中，以免部署或维护期间改动复杂的配置文件出现问题。所以一般会将如数据库连接等重要的信息单独配置到properties文件中，`PropertySourcesPlaceholderConfigurer`允许我们来替换配置信息里的占位符（PlaceHolder）。

基本机制就是，当BeanFactory在第一阶段加载完成所有配置信息，其中保存的对象属性信息还是以占位符形式存在，当`PropertySourcesPlaceholderConfigurer`作为`BeanPostProcessors`应用时，它会使用配置文件中的信息来替换BeanDefinition中占位符表示的属性值。另外`PropertySourcesPlaceholderConfigurer`还会从System类中查找属性，并可控制是否进行覆盖。

您可以使用 PropertySourcesPlaceholderConfigurer 通过使用标准的 Java 属性格式，在一个单独的文件中从 bean 定义中外部化属性值。这样做可以使部署应用程序的人能够自定义特定环境的属性（如数据库 URL 和密码），而无需修改容器的主 XML 定义文件或文件的复杂性或风险。

### PropertyOverrideConfigurer

> The PropertyOverrideConfigurer, another bean factory post-processor, resembles the PropertySourcesPlaceholderConfigurer, but unlike the latter, the original definitions can have default values or no values at all for bean properties. If an overriding Properties file does not have an entry for a certain bean property, the default context definition is used.
>
> Note that the bean definition is not aware of being overridden, so it is not immediately obvious from the XML definition file that the override configurer is being used. In case of multiple PropertyOverrideConfigurer instances that define different values for the same bean property, the last one wins, due to the overriding mechanism.

`PropertyOverrideConfigurer`是另一个`BeanPostProcessors`，它类似于PropertySourcesPlaceholderConfigurer，但与后者不同的是，Bean属性的原始定义可以有默认值或完全没有值。如果覆盖的Properties文件中没有某个bean属性的条目，则使用默认的上下文定义。

[https://developer.aliyun.com/article/459770](https://developer.aliyun.com/article/459770)

如果说`PropertySourcesPlaceholderConfigurer`做的这些是明事的话，那么`PropertyOverrideConfigurer`就有点神不知鬼不觉，它是直接相应的文件中覆盖bean的属性。

需要注意的是，bean定义并不知道被覆盖，所以从XML定义文件中并不能立即看出正在使用覆盖配置器。如果多个`PropertyOverrideConfigurer`实例为同一个Bean属性定义了不同的值，由于覆盖机制的存在，最后一个获胜。

他们父类PropertyResourceConfigurer提供了一个converPropertyValue的方法，允许对配置项进行转换，如对加密后的字符串进行解密后再覆盖到相应的Bean定义中。

## 1.8.3 利用FactoryBean自定义实例逻辑

你可以为本身就是factories的对象实现`org.springframework.beans.factory.FactoryBean`接口。

FactoryBean接口是Spring IoC容器实例化逻辑的一个可插拔点。如果你有复杂的初始化代码，相对于（潜在的）啰嗦的XML量，最好用Java来表达，你可以创建自己的FactoryBean，在这个类里面写复杂的初始化，然后把你自定义的FactoryBean插入到容器中。

FactoryBean接口提供了三个方法。

- `Object getObject()`。返回这个工厂创建的对象的一个实例。这个实例可能是共享的，这取决于这个工厂返回的是单例还是原型。

- `boolean isSingleton()`：如果这个工厂Bean返回单例则返回true。否则返回false。

- Class getObjectType()。返回getObject()方法返回的对象类型，如果事先不知道类型，则返回null。

在Spring框架中，FactoryBean的概念和接口在很多地方被使用。Spring本身就有50多个FactoryBean接口的实现。

当你需要向容器索取一个实际的FactoryBean实例本身而不是它所产生的bean时，在调用ApplicationContext的getBean()方法时，用安培符号(&)将bean的id放在前面。因此，对于一个id为myBean的给定FactoryBean，在容器上调用getBean("myBean")会返回FactoryBean的产物，而调用getBean("&myBean")则会返回FactoryBean实例本身。

> 由BeanFactory中使用的对象实现的接口，这些对象本身就是单个对象的工厂。 如果bean实现此接口，则它将用作对象公开的工厂，而不是直接用作将自身公开的bean实例。
> 注意：实现此接口的bean不能用作普通bean。 FactoryBean是用bean样式定义的，但是为bean引用公开的对象（ getObject() ）始终是它创建的对象。
> FactoryBeans可以支持单例和原型，并且可以按需延迟创建对象，也可以在启动时急于创建对象。 SmartFactoryBean接口允许公开更细粒度的行为元数据。
> 此接口在框架本身中被大量使用，例如用于AOP `org.springframework.aop.framework.ProxyFactoryBean`或`org.springframework.jndi.JndiObjectFactoryBean` 。 它也可以用于自定义组件。 但是，这仅在基础结构代码中很常见。
> FactoryBean是程序性合同。 实现不应依赖于注解驱动的注入或其他反射功能。 getObjectType() getObject()调用可能会在引导过程的早期到达，甚至在任何后处理器设置之前。 如果需要使用其他bean，请实现BeanFactoryAware并以编程方式获取它们。
> 容器仅负责管理FactoryBean实例的生命周期，而不负责管理FactoryBean创建的对象的生命周期。 因此，不会自动调用暴露的bean对象上的destroy方法（例如java.io.Closeable.close() 。相反，FactoryBean应该实现DisposableBean并将任何此类close调用委托给基础对象。
> 最后，FactoryBean对象参与包含BeanFactory的Bean创建同步。 除了出于FactoryBean自身（或类似方式）内部的延迟初始化的目的之外，通常不需要内部同步。



更多关于FactoryBean的使用有很多博客可以参考，[https://blog.csdn.net/jisuanji12306/article/details/86557711](https://blog.csdn.net/jisuanji12306/article/details/86557711)。简单理解可以结合BeanDefinition批量生成Bean，然后提供给开发者注入使用。

使用参考`org.springframework.scheduling.quartz.CronTriggerFactoryBean`等

# 1.9 基于注解的容器配置

容器配置，基于注解比基于XML更好吗？

基于注解的配置的引入提出了这样一个问题：这种方法是否比XML "更好"。简短的回答是 "看情况"。长的答案是，每一种方法都有它的优点和缺点，而且，通常情况下，要由开发人员决定哪种策略更适合他们。由于它们的定义方式，注解在其声明中提供了大量的上下文，导致配置更短、更简洁。然而，XML擅长在不接触组件的源代码或重新编译它们的情况下进行织入。一些开发人员喜欢让布线接近源码，而另一些人则认为，注解类不再是POJO，而且，配置变得分散，更难控制。

无论如何选择，Spring都可以容纳这两种风格，甚至将它们混合在一起。值得指出的是，通过其JavaConfig选项，Spring可以让注解以一种非侵入式的方式使用，而不会触及目标组件的源代码

基于注解的配置提供了基于XML配置的替代方案，它依赖于字节码元数据来布线组件，而不是通过角括号声明。开发者不使用XML来描述bean布线，而是通过在相关的类、方法或字段声明上使用注解，将配置移到组件类本身。如例中提到的。所需的AnnotationBeanPostProcessor中，**使用BeanPostProcessor与注解相结合是扩展Spring IoC容器的常用手段**。例如，Spring 2.0引入了使用@Required注解强制执行所需属性的可能性。Spring 2.5使得遵循同样的通用方法来驱动Spring的依赖注入成为可能。本质上，@Autowired注解提供了与 [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) 中描述的相同的功能，但具有更精细的控制和更广泛的适用性。Spring 2.5还增加了对JSR-250注解的支持，如@PostConstruct和@PreDestroy。Spring 3.0增加了对JSR-330（Java依赖注入）注解的支持，这些注解包含在javax.inject包中，如@Inject和@Named。关于这些注解的细节可以在相关章节中找到。

注解注入是在XML注入之前执行的。因此，XML配置会覆盖注解注入的属性。





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

> `@Configuration` is a class-level annotation indicating that an object is a source of bean definitions. `@Configuration` classes declare beans through public `@Bean` annotated methods. Calls to `@Bean` methods on `@Configuration` classes can also be used to define inter-bean dependencies. See [Basic Concepts: `@Bean` and `@Configuration`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-basic-concepts) for a general introduction.

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

> By using Java configuration, you can create a subclass of `CommandManager` where the abstract `createCommand()` method is overridden in such a way that it looks up a new (prototype) command object. The following example shows how to do so:

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

>  There are a few restrictions due to the fact that CGLIB dynamically adds features at startup-time. In particular, configuration classes must not be final. However, as of 4.3, any constructors are allowed on configuration classes, including the use of `@Autowired` or a single non-default constructor declaration for default injection.
>
> If you prefer to avoid any CGLIB-imposed limitations, consider declaring your `@Bean` methods on non-`@Configuration` classes (for example, on plain `@Component` classes instead). Cross-method calls between `@Bean` methods are not then intercepted, so you have to exclusively rely on dependency injection at the constructor or method level there.

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

> The preceding example works but is simplistic. In most practical scenarios, beans have dependencies on one another across configuration classes. When using XML, this is not an issue, because no compiler is involved, and you can declare `ref="someBean"` and trust Spring to work it out during container initialization. When using `@Configuration` classes, the Java compiler places constraints on the configuration model, in that references to other beans must be valid Java syntax.
>
> Fortunately, solving this problem is simple. As [we already discussed](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-dependencies), a `@Bean` method can have an arbitrary number of parameters that describe the bean dependencies. Consider the following more real-world scenario with several `@Configuration` classes, each depending on beans declared in the others:

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

> Constructor injection in `@Configuration` classes is only supported as of Spring Framework 4.3. Note also that there is no need to specify `@Autowired` if the target bean defines only one constructor.

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

# 1.13  Environment 的抽象

> The [`Environment`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/core/env/Environment.html) interface is an abstraction integrated in the container that models two key aspects of the application environment: [profiles](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles) and [properties](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-property-source-abstraction).
>
> A profile is a named, logical group of bean definitions to be registered with the container only if the given profile is active. Beans may be assigned to a profile whether defined in XML or with annotations. The role of the `Environment` object with relation to profiles is in determining which profiles (if any) are currently active, and which profiles (if any) should be active by default.
>
> Properties  play an important role in almost all applications and may originate from a variety of sources: properties files, JVM system properties, system environment variables, JNDI, servlet context parameters, ad-hoc `Properties` objects, `Map` objects, and so on. The role of the `Environment` object with relation to properties is to provide the user with a convenient service interface for configuring property sources and resolving properties from them.

 [`Environment`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/core/env/Environment.html)是集成在容器中的一个抽象概念，它建模了应用程序的两个关键方面： [profiles](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles) and [properties](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-property-source-abstraction).

**profile**是一个命名的，逻辑的Bean定义组。只有在给定的profile激活时才会注册到容器。不管是在XML定义还是用注解，bean都可以被分配给profile。[`Environment`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/core/env/Environment.html)和关联的Profiles的作用就是确定哪些Profiles（如果有的话）是激活的，哪些是在默认时激活的。

**Properties** 在几乎所有的应用中都扮演着重要的角色，可能来自：properties files， JVM system properties系统属性，system environment variables环境变量，JNDI， servlet 上下文参数，特定的属性对象，map对象等等。`Environment` 和关联属性，其作用是为用户提供一个方便的接口，**用于配置属性来源和从中解析属性**。

## 1.13.1 Bean定义的Profiles

Bean定义profiles 在核心容器中提供了一种机制，允许在不同环境中注册不同的Bean。"environment"这个词，对不同的用户来说可能意味着不同的东西，这个功能可以帮助实现许多场景，包括：

在开发中从内存中的数据源工作，而在QA或生产中则从JNDI中查找相同的数据源。

仅在将应用程序部署到性能环境中时才注册监控基础设施。

为客户A和客户B的部署注册自定义的Bean实现。



考虑实际应用中的第一个用例，它需要一个数据源。在测试环境中，配置可能类似于以下情况：

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

现在考虑如何将此应用程序部署到QA或生产环境中，假设应用程序的数据源在生产应用程序服务器的JNDI目录中注册。我们的dataSource bean现在看起来像下面的列表。

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

问题是如何根据当前环境在使用这两种变化之间进行切换。随着时间的推移，Spring的用户已经设计了许多方法来完成这个任务，通常依赖于系统环境变量和包含${placeholder}标记的XML <import/>语句的组合，这些标记根据环境变量的值解析到正确的配置文件路径。Bean定义profiles是核心容器的特性，它为这个问题提供了一个解决方案。

如果我们将前面环境特定的bean定义示例中所示的用例进行泛化，我们最终需要在某些环境中注册某些bean定义，而在其他环境中则不需要。你可以说，你想在情况A中注册某个Bean定义的配置文件，而在情况B中注册不同的配置文件，我们首先更新我们的配置来反映这个需求。

**使用@Profile**

通过@Profile注解，您可以指定当一个或多个指定的配置文件激活时，一个component 才有资格进行注册。使用我们前面的例子，我们可以重写dataSource配置如下。

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```

```java
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

profile string可以包含一个简单的profile 名称（例如`production`）或profile 表达式。profile 表达式允许表达更复杂的配置文件逻辑（例如，production & us-east）。配置文件表达式中支持以下运算符: `! & |`非 与 或的逻辑概念。

你不能在没有括号时混合使用 `&` and `|` operators . 例如 `production & us-east | eu-central` 不是一个合法的表达式. It must be expressed as `production & (us-east | eu-central)`.

如果一个@Configuration类被标记为@Profile，那么与该类相关的所有@Bean方法和@Import注解都会被绕过，除非一个或多个指定的profile是活动的。如果一个@Component或@Configuration类被标记为@Profile({"p1", "p2"})，那么除非配置文件'p1'或'p2'被激活，否则该类不会被注册或处理。如果给定的profile前缀有NOT操作符(!)，则只有在profile未激活的情况下才会注册被注解的元素。例如，给定@Profile({"p1", "!p2"})，如果配置文件'p1'已激活或配置文件'p2'未激活，则注册将发生。

你可以使用@profile作为元注解来自定义自己的注解：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

@Profile也可以在方法层声明，只包含一个配置类的特定bean（例如，对于特定bean的替代变体），如下例所示。

```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") // development被激活时才会注册
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") // production被激活时才会注册
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

> 在@Bean方法上使用@Profile，可能会出现一种特殊情况。在同一Java方法名的重载@Bean方法的情况下（类似于构造函数重载），需要在所有重载方法上一致地声明一个@Profile条件。如果条件不一致，只有重载方法中第一个声明的条件才重要。因此，@Profile不能用来选择一个具有特定参数签名的重载方法而不是另一个。在创建时，同一bean的所有工厂方法之间的解析遵循Spring的构造函数解析算法。
>
> **如果你想定义具有不同配置文件条件的备选Bean，请使用不同的Java方法名**，通过使用@Bean名属性指向同一个Bean名，如前例所示。如果参数签名都是相同的（例如，所有的变体都有无参数的工厂方法），这是首先在一个有效的Java类中表示这种安排的唯一方法（因为只能有一个特定名称和参数签名的方法）。

**激活Profile**

现在我们已经更新了配置，我们仍然需要指示Spring哪个配置文件是活动的。如果我们现在启动我们的示例应用程序，我们会看到一个NoSuchBeanDefinitionException抛出，因为容器找不到名为dataSource的Spring bean。

激活profile可以通过多种方式进行，但最直接的方式是通过ApplicationContext对Environment API进行编程。下面的例子展示了如何做到这一点。

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

此外，你还可以通过spring.profile.active属性来声明激活profile，它可以通过系统环境变量、JVM系统属性、web.xml中的servlet上下文参数，甚至作为JNDI中的一个条目来指定（see [`PropertySource` Abstraction](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-property-source-abstraction)）。在集成测试中，可以通过使用spring-test模块中的@ActiveProfiles注解来声明活动配置文件（see [context configuration with environment profiles](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-env-profiles)）。

请注意，配置文件不是一个 "非此即彼 "的命题。你可以同时激活多个配置文件。在编程上，您可以为 setActiveProfiles() 方法提供多个 profiles 名称，该方法接受数组 String... varargs。下面的例子可以激活多个profiles `ctx.getEnvironment().setActiveProfiles("profile1", "profile2");`

在配置文件中，通过逗号分隔，` -Dspring.profiles.active="profile1,profile2"`

**Default Profile**

default profile表示默认情况下启用：

```java
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有profile被激活，就会创建这个dataSource。你可以把它看作是为一个或多个bean提供默认定义的一种方式。如果启用了任何配置文件，则默认配置文件不适用。

你可以通过使用环境中的setDefaultProfiles()或声明性地使用spring.profile.default属性来更改默认配置文件的名称。

## 1.13.2 `PropertySource`的抽象

Spring的 Environment在可配置的属性源层次上提供搜索操作

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```

在这段代码中，我们看到了询问Spring是否为当前环境定义了my-property属性的高级方法。为了回答这个问题，Environment对象在一组PropertySource对象上执行搜索。**PropertySource是对任何来源 key-value 键值对的简单抽象**，Spring的StandardEnvironment配置了两个PropertySource对象--一个代表JVM系统属性集（System.getProperties()），一个代表系统环境变量集（System.getenv()）。

> 这些默认属性源存在于StandardEnvironment中，用于独立的应用程序。StandardServletEnvironment被填充了额外的默认属性源，包括servlet config和servlet上下文参数。它可以选择启用一个JndiPropertySource。

具体来说，当你使用StandardEnvironment时，如果运行时存在my-property系统属性或my-property环境变量，则调用env.containsProperty("my-property")返回true。

>  The search performed is hierarchical. By default, system properties have precedence over environment variables. So, if the `my-property` property happens to be set in both places during a call to `env.getProperty("my-property")`, the system property value “wins” and is returned. Note that property values are not merged but rather completely overridden by a preceding entry.
>
> For a common `StandardServletEnvironment`, the full hierarchy is as follows, with the highest-precedence entries at the top:

执行的搜索是分层的。默认情况下，系统属性优先于环境变量。因此，如果在调用env.getProperty("my-property")的过程中，my-property属性恰好在两个地方都被设置了，那么系统属性值就会 "获胜 "并被返回。请注意，属性值不会被合并，而是被前面的条目完全覆盖。

对于一个普通的StandardServletEnvironment，完整的层次结构如下，最高优先级的条目在顶部:

1. ServletConfig 参数(if applicable — 例如`DispatcherServlet` 的上下文)
2. ServletContext 参数(web.xml context-param entries)
3. JNDI environment variables (`java:comp/env/` entries)
4. JVM system properties系统属性 (`-D` command-line arguments)
5. JVM system environment (操作系统环境变量operating system environment variables)

最重要的是，整个机制是可配置的。假如你有一个自定义的属性源，你想把它集成到这个搜索中。要做到这一点，请实现并实例化您自己的`PropertySource`，并将其添加到当前`Environment`的`PropertySources`集合中。下面的例子展示了如何做到这一点：

```java
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

```java
public class ApolloEncryptApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    public ApolloEncryptApplicationContextInitializer() {
    }
	// 会让bootstrap.yml配置文件失效，因为改变了bootstrap.yml在application启动之前就加载的默认配置
    public void initialize(ConfigurableApplicationContext applicationContext) {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        Properties properties = new Properties();
        properties.setProperty("jasypt.encryptor.password", "aaaa");
        PropertiesPropertySource propertiesPropertySource = new PropertiesPropertySource("applicationConfig", properties);
        MutablePropertySources mutablePropertySources = environment.getPropertySources();
        mutablePropertySources.addFirst(propertiesPropertySource);
    }
}
```

在前面的代码中，MyPropertySource在搜索中以最高优先级被添加。如果它包含一个my-property属性，则检测并返回该属性，而不是任何其他PropertySource中的任何my-property属性。MutablePropertySources API暴露了许多方法，这些方法允许对属性源集合进行精确的操作。



## 1.13.3 @PropertySource的使用

`@PropertySource`注解提供了一个方便的声明式机制，**用于向Spring的Environment中添加一个PropertySource**。

给定一个名为app.properties的文件，其中包含键值对`testbean.name=myTestBean`，下面的@Configuration类使用了@PropertySource，这样调用testBean.getName()就会返回myTestBean。

```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

任何存在于@PropertySource资源位置中的${...}占位符都会被解析为已经注册到environment的属性源集，如下例所示：

```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

假设my.placeholder存在于已经注册的某个属性源中（例如系统属性或环境变量），则placeholder会被解析为相应的值。如果没有，则使用default/path作为默认值。如果没有指定默认值，且无法解析属性，则会抛出IllegalArgumentException。

根据Java 8惯例，@PropertySource注解是可以重复的。但是，所有这样的@PropertySource注解都需要在同一层次上声明，可以直接在配置类上声明，也可以在同一个自定义注解中作为元注解声明。不推荐混合使用直接注解和元注解，因为直接注解有效地覆盖了元注解。如：

```java
@Getter
@Setter
@Component
@ConfigurationProperties("data.redis")
@PropertySource(value = "classpath:data/read/test.properties", ignoreResourceNotFound = true) 
@PropertySource(value = "file://${CONFIG_HOME}/data/redis/test.properties", ignoreResourceNotFound = true)
public class DataProperties {

    private UserProps user;

    private Device connect;
    private String oauthJwtSecret;

    private List<Device> devices;

    @Getter
    @Setter
    public static class UserProps {
        private String email;
        private String countryCode;
        private String password;
    }

    @Getter
    @Setter
    public static class Device {
        private String uuid;
        private String pid;
    }
}
```

## 1.13.4 声明中的占位符解析

历史上，元素中占位符的值只能针对JVM系统属性或环境变量进行解析。现在不再是这种情况了。因为`Environment` 抽象集成在整个容器中，所以很容易通过它来解析占位符。这意味着您可以以任何喜欢的方式配置解析过程。您可以改变系统属性和环境变量搜索的优先级，或者完全删除它们。你也可以根据情况将自己的属性源添加到组合中。

具体来说，不管自定义属性是在哪里定义的，只要在Environment中可用，下面的语句就可以用：

```xml
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```



## 附录

**环境变量system environment variables**是和操作系统相关的，Windows和Linux是不同的语法

环境变量的获取通过`System.getenv()`，不提供设置环境变量的方法。

环境变量产生更多的**全局效应**，因为它们不仅对Java子进程可见，而且对于定义它们的进程的**所有子进程都是可见的**。在不同的操作系统上，它们的语义有细微的差别，比如，不区分大小写。因此环境变量更可能有意料不到的副作用。程序中尽可能使用系统属性。环境变量应该在需要全局效应的时候使用，或者在外部系统接口要求使用环境变量时使用（比如 PATH）

而**系统属性 JVM system properties** 是属于JVM的，

- 系统属性的设置: 通过JVM参数: `-D属性名=值` 或者在代码中通过`Sytem.setProperty(String key, String value)`来设置.
- 系统属性的获取: 在Java中通过`System.getProperty(String key)`获取属性值.

```java
@Configuration
// @ConditionalOnExpression("!'pro'.equals(environment['env'])")
public class SwaggerEnableConfig implements EnvironmentAware {

    @Override
    public void setEnvironment(Environment environment) {
        // 获取环境变量
        Map<String, String> env = System.getenv();

        // 获取系统属性
        Properties properties = System.getProperties();

        System.out.println(properties);
    }
}
```

[环境变量和系统属性的区别](https://fangshixiang.blog.csdn.net/article/details/90694330?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2.control)

![aspectJ](images/环境变量.png)

# 1.14 注册`LoadTimeWeaver`

LoadTimeWeaver被Spring用于在类被加载到Java虚拟机（JVM）中时进行动态转换。

开启加载时织入(load-time weaving)，需要开启配置：

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

一旦为ApplicationContext配置，该ApplicationContext中的任何bean都可以实现`LoadTimeWeaverAware`，从而接收对加载时织入实例的引用。在结合Spring的JPA支持时，这一点特别有用，因为在JPA类转换中可能需要加载时织入。更多细节请查阅 [`LocalContainerEntityManagerFactoryBean`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/orm/jpa/LocalContainerEntityManagerFactoryBean.html)和 [Load-time Weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw).

# 1.15 ApplicationContext的其他能力

正如在章节介绍中所讨论的那样，`org.springframework.beans.factory`包提供了管理和操作Bean的基本功能，包括以编程的方式。`org.springframework.context`包增加了ApplicationContext接口，它继承BeanFactory接口，另外继承了其他接口以更加面向应用框架的方式提供额外的功能。很多人在使用ApplicationContext时，完全采用声明式的方式，甚至不以编程的方式创建它，而是依靠ContextLoader等支持类来自动实例化ApplicationContext，作为Java EE Web应用正常启动过程的一部分。

为了以更加面向框架的风格增强 `BeanFactory` 接口的功能,  context 包还提供了以下功能：

- 通过 `MessageSource` 接口，访问i18n风格的消息
- 通过 `ResourceLoader` 接口，访问resource，如URL和file
- 通过 `ApplicationEventPublisher` 接口，向`ApplicationListener`的实现事件发布
- 通过`HierarchicalBeanFactory`接口，加载多个（分层）上下文，让每个上下文都集中在一个特定的层上，如应用程序的web层。

![applicationContext](images/applicationContext.png)

## 1.15.1 使用MessageSource实现国际化

ApplicationContext接口继承了名为MessageSource的接口，因此，它提供了国际化（"i18n"）功能。Spring还提供了HierarchicalMessageSource接口，它可以对消息进行分级解析。这些接口共同提供了Spring实现消息解析的基础。这些接口上定义的方法包括：

- `String getMessage(String code, Object[] args, String default, Locale loc)`：用于从MessageSource中检索消息的基本方法。当没有找到指定locale的消息时，将使用默认消息。任何传递进来的参数都会成为替换值，使用标准库提供的MessageFormat功能。

- `String getMessage(String code, Object[] args, Locale loc)`:基本上与前一个方法相同，但有一点不同。不能指定默认消息。如果找不到消息，将抛出NoSuchMessageException。

- `String getMessage(MessageSourceResolvable resolvable, Locale locale)`：前面方法中使用的所有属性都被封装在一个名为MessageSourceResolvable的类中，你可以用这个方法来使用它。



当加载ApplicationContext时，它会自动搜索上下文中定义的MessageSource Bean。这个Bean的名字必须是messageSource。如果找到了这样一个bean，所有对前面方法的调用都会委托给这个message source。如果没有找到message source，ApplicationContext 试图找到一个包含相同名称的 bean 的父类。如果找到了，它将使用该bean作为message source。如果ApplicationContext找不到任何message source，为了能够接受对上面定义的方法的调用，就会实例化一个空的DelegatingMessageSource。



Spring提供了两个MessageSource实现，即`ResourceBundleMessageSource`和`StaticMessageSource`。两者都实现了HierarchicalMessageSource，以便进行嵌套消息传递。StaticMessageSource很少使用，一般不用于生产环境，但提供了向消息源添加消息的程序化方法。下面的例子展示了ResourceBundleMessageSource:

```xml
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```

本例假设在classpath中定义了三个resource bundles资源包，分别为format,exceptions和windows。任何解析消息的请求都是以JDK标准的方式通过ResourceBundle对象解析消息来处理的。在本例中，假设上述两个资源捆绑文件的内容如下：

```properties
# in format.properties
 message=Alligators rock!
```

```properties
# in exceptions.properties
argument.required=The {0} argument is required.
```

下一个例子显示了一个运行MessageSource功能的程序。请记住，所有的ApplicationContext实现也是MessageSource实现，因此可以转为MessageSource接口：

```java
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", Locale.ENGLISH);
    System.out.println(message);
}
```

这段代码的输出为`Alligators rock!`

总结一下，MessageSource定义在一个名为beans.xml的文件中，它存在于classpath的根目录下。messageSource bean定义通过它的basenames属性来引用一些资源捆绑。列表中传递给basenames属性的三个文件作为文件存在于你的classpath的根路径，分别称为format.properties、exceptions.properties和windows.properties。

下一个例子显示了传递给消息查找的参数。这些参数被转换为String对象，并插入到查找消息的占位符中：

```java
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", Locale.ENGLISH);
        System.out.println(message);
    }
}
```

这段代码输出为：

```shell
The userDao argument is required.
```

关于国际化（"i18n"），Spring的各种MessageSource实现遵循与标准JDK `ResourceBundle`相同的locale解析和回退规则。简而言之，继续前面定义的messageSource示例，如果你想解决针对英国（en-GB）本地化的消息，你将创建分别称为format_en_GB.properties、exceptions_en_GB.properties和windows_en_GB.properties的文件。

**按照规定properties文件里的内容按照ISO-8859-1编码的**，所以出现乱码是正常的！

通常情况下，locale解析是由应用程序的周围环境管理的。在下面的例子中，(英国)消息所针对的locale是手动指定的。

```properties
# in exceptions_en_GB.properties
argument.required=Ebagum lad, the ''{0}'' argument is required, I say, required.
```

```java
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```

输出为：

```
Ebagum lad, the 'userDao' argument is required, I say, required.
```



您也可以使用`MessageSourceAware`接口来获取对任何已定义的MessageSource的引用。任何在实现MessageSourceAware接口的ApplicationContext中定义的Bean都会在创建和配置Bean时被注入应用上下文的MessageSource。

作为`ResourceBundleMessageSource`的替代，Spring提供了一个`ReloadableResourceBundleMessageSource`类。这个变体支持相同的捆绑文件格式，但比基于标准JDK的`ResourceBundleMessageSource`实现更加灵活。特别是，它允许从任何Spring资源位置读取文件（不仅仅是从classpath），并支持捆绑属性文件的热重载（同时有效地缓存它们）。详情请参见 [`ReloadableResourceBundleMessageSource`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/context/support/ReloadableResourceBundleMessageSource.html) 。

## 1.15.2 标准和自定义的Events

ApplicationContext中的event处理是通过ApplicationEvent类和ApplicationListener接口提供的。如果将实现ApplicationListener接口的Bean部署到上下文中，每次ApplicationEvent被发布到ApplicationContext时，该Bean都会得到通知。本质上，这就是标准的Observer观察者设计模式。

从Spring 4.2开始，event基础架构得到了显著的改进，提供了一个基于注解的模型，以及发布任何任意事件的能力（即不一定从ApplicationEvent扩展的对象）。当这样的object被发布时，我们会将其包裹在一个event中。

下面展示Spring提供的几个Event：

| Event                        | Explanation                                                  |
| :--------------------------- | :----------------------------------------------------------- |
| `ContextRefreshedEvent`      | 在 `ApplicationContext` is initialized or refreshed (for example, by using the `refresh()` method on the `ConfigurableApplicationContext` interface)发布. Here, “initialized” means that all beans are loaded, post-processor beans are detected and activated, singletons are pre-instantiated, and the `ApplicationContext` object is ready for use. <br />As long as the context has not been closed, a refresh can be triggered multiple times, provided that the chosen `ApplicationContext` actually supports such “hot” refreshes. For example, `XmlWebApplicationContext` supports hot refreshes, but `GenericApplicationContext` does not. |
| `ContextStartedEvent`        | Published when the `ApplicationContext` is started by using the `start()` method on the `ConfigurableApplicationContext` interface. Here, “started” means that all `Lifecycle` beans receive an explicit start signal. Typically, this signal is used to restart beans after an explicit stop, but it may also be used to start components that have not been configured for autostart (for example, components that have not already started on initialization).<br />当使用ConfigurableApplicationContext接口上的start()方法启动ApplicationContext时发布。在这里，"started "意味着所有的Lifecycle Bean都收到一个显式的启动信号。通常情况下，这个信号用于在显式停止后重新启动bean，但它也可以用于启动没有被配置为自动启动的组件（例如，在初始化时尚未启动的组件）。 |
| `ContextStoppedEvent`        | Published when the `ApplicationContext` is stopped by using the `stop()` method on the `ConfigurableApplicationContext` interface. Here, “stopped” means that all `Lifecycle` beans receive an explicit stop signal. A stopped context may be restarted through a `start()` call. |
| `ContextClosedEvent`         | Published when the `ApplicationContext` is being closed by using the `close()` method on the `ConfigurableApplicationContext` interface or via a JVM shutdown hook. Here, "closed" means that all singleton beans will be destroyed. Once the context is closed, it reaches its end of life and cannot be refreshed or restarted.<br />当 "ApplicationContext "通过使用 "ConfigurableApplicationContext "接口上的 "close() "方法或通过JVM关闭钩子关闭时发布。在这里，"关闭 "意味着所有的单例Bean将被销毁。一旦上下文被关闭，它的生命就结束了，不能被刷新或重新启动。 |
| `RequestHandledEvent`        | A web-specific event telling all beans that an HTTP request has been serviced. This event is published after the request is complete. This event is only applicable to web applications that use Spring’s `DispatcherServlet`.<br />一个Web特有的事件，告诉所有Bean一个HTTP请求已经被服务。该事件会在请求完成后发布。该事件仅适用于使用Spring的 "DispatcherServlet "的Web应用程序。 |
| `ServletRequestHandledEvent` | A subclass of `RequestHandledEvent` that adds Servlet-specific context information. 增加了servlet特有的上下文信息 |

你也可以创建和发布自定义的event，

```java
public class BlockedListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlockedListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // accessor and other methods...
}
```

要发布一个自定义的ApplicationEvent，请在ApplicationEventPublisher上调用publishEvent()方法。通常情况下，通过创建一个实现ApplicationEventPublisherAware的类并将其注册为Spring bean来实现。

```java
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blockedList;
    private ApplicationEventPublisher publisher;

    public void setBlockedList(List<String> blockedList) {
        this.blockedList = blockedList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blockedList.contains(address)) {
            publisher.publishEvent(new BlockedListEvent(this, address, content));
            return;
        }
        // send email...
    }
}
```

```java
public class BlockedListNotifier implements ApplicationListener<BlockedListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlockedListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

但注意，默认情况下，事件监听器同步接收事件。这意味着publishEvent()方法会阻塞，直到所有的监听器处理完事件。这种同步和单线程方法的一个优点是，当监听器接收到一个事件时，如果有事务上下文可用，它就会在发布者的事务上下文中操作。

spring的事件机制是为同一应用上下文中Spring Bean之间的简单通信而设计的。然而，对于更复杂的企业集成需求，单独维护的Spring Integration项目为构建轻量级的、面向模式的、事件驱动的架构提供了完整的支持，这些架构建立在著名的Spring编程模型之上。

**基于注解的Event Listeners**

在spring4.2中，可以通过`@EventListener`注解在Bean的任意public方法上来注册一个事件监听器。

```java
public class BlockedListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlockedListEvent(BlockedListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

方法签名再次声明它所监听的事件类型，但是，这次用了一个灵活的名字，而且没有实现特定的监听接口。只要实际的事件类型在其实现层次中能解析为你的通用参数，事件类型也可以通过通用来缩小。

如果您的方法应该监听多个事件，或者您想在定义方法时不使用任何参数，也可以在注解本身指定事件类型。下面的示例展示了如何做到这一点。

```java
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    // ...
}
```

也可以通过使用SpEL表达式来添加额外的过滤，下面的示例显示了如何重写我们的通知器，使其仅在事件的内容属性等于my-event时才被调用。

```java
@EventListener(condition = "#blEvent.content == 'my-event'")
public void processBlockedListEvent(BlockedListEvent blockedListEvent) {
    // notify appropriate parties via notificationAddress...
}
```

每个SpEL表达式都是针对一个专用上下文进行评估的。下表列出了上下文可用的项目，以便您可以将它们用于条件事件处理。

| Name            | Location           | Description                                                  | Example                                                      |
| :-------------- | :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Event           | root object        | The actual `ApplicationEvent`.                               | `#root.event` or `event`                                     |
| Arguments array | root object        | The arguments (as an object array) used to invoke the method. | `#root.args` or `args`; `args[0]` to access the first argument, etc. |
| *Argument name* | evaluation context | The name of any of the method arguments. If, for some reason, the names are not available (for example, because there is no debug information in the compiled byte code), individual arguments are also available using the `#a<#arg>` syntax where `<#arg>` stands for the argument index (starting from 0). | `#blEvent` or `#a0` (you can also use `#p0` or `#p<#arg>` parameter notation as an alias) |

如果你需要发布一个事件作为处理另一个事件的结果，你可以改变方法签名来返回应该发布的事件，如下例所示:

```java
@EventListener
public ListUpdateEvent handleBlockedListEvent(BlockedListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```

这个特性在异步监听时不支持

这个新方法为上面方法处理的每个BlockedListEvent发布一个新的ListUpdateEvent。如果你需要发布多个事件，你可以返回一个事件集合来代替。

在使用异步事件时要注意以下限制。

- 如果异步事件监听器抛出了一个Exception，它不会传播给调用者，请参见AsyncUncaughtExceptionHandler了解更多细节。更多细节请参见AsyncUncaughtExceptionHandler。

- 异步事件监听器方法不能通过返回一个值来发布后续事件。如果你需要发布另一个事件作为处理结果，注入一个ApplicationEventPublisher来手动发布事件。

### 异步的监听

如果你想异步地处理事件，可以使用@Async

```java
@EventListener
@Async
public void processBlockedListEvent(BlockedListEvent event) {
    // BlockedListEvent is processed in a separate thread
}
```

### 有序的监听

如果你想在另一个监听器之前先监听到事件，可以使用@Order

```java
@EventListener
@Order(42)
public void processBlockedListEvent(BlockedListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```

### 泛型event

> You can also use generics to further define the structure of your event. Consider using an `EntityCreatedEvent<T>` where `T` is the type of the actual entity that got created. For example, you can create the following listener definition to receive only `EntityCreatedEvent` for a `Person`:

你也可以使用泛型来进一步定义你的事件结构。考虑使用EntityCreatedEvent<T>，其中T是实际创建的实体的类型。例如，您可以创建以下监听器定义，以便只接收Person的EntityCreatedEvent。

```java
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    // ...
}
```

> Due to type erasure, this works only if the event that is fired resolves the generic parameters on which the event listener filters (that is, something like `class PersonCreatedEvent extends EntityCreatedEvent<Person> { … }`).
>
> In certain circumstances, this may become quite tedious if all events follow the same structure (as should be the case for the event in the preceding example). In such a case, you can implement `ResolvableTypeProvider` to guide the framework beyond what the runtime environment provides. The following event shows how to do so:

由于泛型擦除，只有当被触发的事件解析了事件监听器过滤的通用参数时，这个方法才会起作用（也就是说，类似于class PersonCreatedEvent extends EntityCreatedEvent<Person> { ... }）。

在某些情况下，如果所有的事件都遵循相同的结构，这可能会变得相当乏味（就像前面例子中的事件一样）。在这种情况下，你可以实现ResolvableTypeProvider来引导框架超越运行时环境提供的内容。下面的事件展示了如何做到这一点。

```
public class EntityCreatedEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(), ResolvableType.forInstance(getSource()));
    }
}
```

## 1.15.3 方便地获取Low-level Resources

为了最佳的使用和理解应用上下文，您应该熟悉Spring的 `Resource` 抽象，如[Resources](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources)中所述。

一个应用上下文就是一个ResourceLoader，它可以用来加载Resource对象。Resource本质上是JDK java.net.URL类的一个功能更丰富的版本。事实上，Resource的实现在适当的地方包裹了java.net.URL的实例。Resource可以以透明的方式从几乎任何位置获取低级资源，包括从classpath、文件系统位置、任何可以用标准URL描述的地方以及其他一些变化。如果资源位置字符串是一个简单的路径，没有任何特殊的前缀，那么这些资源的来源是特定的，适合于实际的应用程序上下文类型。

你可以配置一个部署到应用上下文中的Bean来实现特殊的回调接口ResourceLoaderAware，以在初始化时自动回调应用上下文本身传递的ResourceLoader。你也可以暴露 `Resource`类型的属性，用于访问静态资源。它们像其他属性一样被注入其中。您可以将这些Resource属性指定为简单的字符串路径，并在Bean部署时依靠自动从这些文本字符串转换为实际的Resource对象。

提供给ApplicationContext构造函数的位置路径或路径实际上是资源字符串，以简单的形式，根据具体的上下文实现进行适当处理。例如ClassPathXmlApplicationContext将一个简单的位置路径视为classpath。你也可以使用带有特殊前缀的位置路径（资源字符串）来强制加载来自classpath或URL的定义，而不管实际的上下文类型如何。

## 1.15.4 跟踪应用启动

ApplicationContext管理Spring应用程序的生命周期，并围绕组件提供丰富的编程模型。因此，复杂的应用程序可以有同样复杂的组件图和启动阶段。

通过特定的指标来跟踪应用程序的启动步骤，可以帮助了解在启动阶段的时间花费在哪里，但也可以作为一种方法来更好地理解整个上下文生命周期。

AbstractApplicationContext(及其子类)的工具是ApplicationStartup，它收集各种启动阶段的StartupStep数据。

应用上下文生命周期（基础包扫描，配置类管理）。

beans生命周期（实例化、智能初始化、后处理）。

应用events 处理

```java
// create a startup step and start recording
StartupStep scanPackages = this.getApplicationStartup().start("spring.context.base-packages.scan");
// 在当前阶段添加标记
scanPackages.tag("packages", () -> Arrays.toString(basePackages));
// 执行我们要测量的实际步骤
this.scanner.scan(basePackages);
// end the current step
scanPackages.end();
```

应用程序的上下文已经有了多个步骤的工具。一旦被记录下来，这些启动步骤就可以通过特定的工具进行收集、显示和分析。对于现有启动步骤的完整列表，您可以查看专门的附录部分。

默认的ApplicationStartup实现是一个无操作的变量，以最小的开销。这意味着在应用程序启动期间，默认情况下不会收集任何指标。Spring Framework提供了一个用Java Flight Recorder跟踪启动步骤的实现。FlightRecorderApplicationStartup。要使用这个变量，您必须在创建ApplicationContext后立即为其配置一个实例。

如果开发者要提供自己的AbstractApplicationContext子类，或者希望收集更精确的数据，他们也可以使用ApplicationStartup基础架构。

ApplicationStartup的目的是只在应用程序启动期间和核心容器中使用；这绝不是Java剖析器或指标库（如Micrometer）的替代品。
要开始收集自定义的StartupStep，组件可以直接从应用上下文中获取ApplicationStartup实例，使其组件实现ApplicationStartupAware，或者在任何注入点上询问ApplicationStartup类型。

开发人员在创建自定义启动步骤时，不应使用 "spring.*"命名空间。这个命名空间是为Spring内部使用而保留的，可能会发生变化。

#  1.16 关于`BeanFactory`

> The `BeanFactory` API provides the underlying basis for Spring’s IoC functionality. Its specific contracts are mostly used in integration with other parts of Spring and related third-party frameworks, and its `DefaultListableBeanFactory` implementation is a key delegate within the higher-level `GenericApplicationContext` container.

>  `BeanFactory` and related interfaces (such as `BeanFactoryAware`, `InitializingBean`, `DisposableBean`) are important integration points for other framework components. By not requiring any annotations or even reflection, they allow for very efficient interaction between the container and its components. Application-level beans may use the same callback interfaces but typically prefer declarative dependency injection instead, either through annotations or through programmatic configuration.

>  Note that the core `BeanFactory` API level and its `DefaultListableBeanFactory` implementation do not make assumptions about the configuration format or any component annotations to be used. All of these flavors come in through extensions (such as `XmlBeanDefinitionReader` and `AutowiredAnnotationBeanPostProcessor`) and operate on shared `BeanDefinition` objects as a core metadata representation. This is the essence of what makes Spring’s container so flexible and extensible.

BeanFactory API为SpringIOC功能提供了底层基础。它具体规定主要用于Spring其他部分和第三方框架的集成，其DefaultListableBeanFactory实现是上层GenericApplicationContext容器的关键代理。

BeanFactory和相关接口（如BeanFactoryAware、InitializingBean、DisposableBean）是其他框架组件的重要集成点。通过不需要任何注解甚至反射，他们允许容器和其他组件之间进行非常高效的交互。应用级的Bean可以使用相同的回调接口，但通常更倾向于通过注解或代码来声明注入依赖。

请注意，核心BeanFactory API层及其实现DefaultListableBeanFactory并没有对配置格式或任何要使用的组件注解做出假设。所有这些格式都是通过扩展（如XmlBeanDefinitionReader和AutowiredAnnotationBeanPostProcessor）进来的，并在共享的BeanDefinition对象上操作，作为核心元数据表示。这就是Spring的容器如此灵活和可扩展的本质。

## 1.16.1  `BeanFactory` 还是`ApplicationContext`?

下面介绍BeanFactory和ApplicationContext容器级别之间的差异以及对引导的影响。

你应该使用ApplicationContext，除非你有很好的理由不这样做，GenericApplicationContext及其子类AnnotationConfigApplicationContext是自定义引导的常用实现。这些是Spring核心容器的主要入口点，用于所有常见的目的：加载配置文件、触发classpath扫描、编程注册bean定义和注解类，以及（从5.0开始）注册功能bean定义。

因为ApplicationContext包含了BeanFactory的所有功能，所以一般推荐使用ApplicationContext，而不是普通的BeanFactory，除非是需要完全控制Bean处理的场景。在ApplicationContext中（如GenericApplicationContext实现），有几种Bean是通过惯例（即通过Bean名称或Bean类型--特别是后处理器）来检测的，而普通的DefaultListableBeanFactory对任何特殊的Bean都是不可知的。

对于容器的许多扩展功能，如注解处理和AOP代理，BeanPostProcessor扩展是必不可少的。如果你只使用一个普通的DefaultListableBeanFactory，像后处理器默认不会被检测和激活。这种情况可能会让人感到困惑，因为实际上你的bean配置没有任何问题。相反，在这样的情况下，容器需要通过额外的设置进行完全的引导。

下面表格展示 `BeanFactory` 和 `ApplicationContext` 提供的功能列表：

| Feature                                                 | `BeanFactory` | `ApplicationContext` |
| :------------------------------------------------------ | :------------ | :------------------- |
| Bean instantiation/wiring                               | Yes           | Yes                  |
| Integrated lifecycle management                         | No            | Yes                  |
| Automatic `BeanPostProcessor` registration              | No            | Yes                  |
| Automatic `BeanFactoryPostProcessor` registration       | No            | Yes                  |
| Convenient `MessageSource` access (for internalization) | No            | Yes                  |
| Built-in `ApplicationEvent` publication mechanism       | No            | Yes                  |

一个AnnotationConfigApplicationContext已经注册了所有常见的注解post-processors，并且可以通过配置注解（如@EnableTransactionManagement）引入额外的处理器。在Spring基于注解的配置模型的抽象层次上，bean后处理器的概念变成了一个单纯的内部容器细节。
