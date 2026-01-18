本节将更详细地介绍如何使用Spring Boot。它涵盖了诸如构建系统、自动配置以及如何运行您的应用程序等主题。我们还介绍了一些Spring Boot的最佳实践。虽然Spring Boot没有什么特别的地方（它只是另一个您可以依赖的库），但有一些建议，如果遵循会让您的开发过程变得更容易。

# 1. Build Systems

## 1.1 依赖管理

Spring Boot 的每个版本都提供了它所支持的依赖项的精选列表。实际上，您不需要在构建配置中为这些依赖项提供版本，因为Spring Boot为您管理这些依赖项。当您升级Spring Boot本身时，这些依赖项也会以一致的方式升级。

如果您需要的话，您仍然可以指定一个版本并覆盖Spring Boot的建议。

Spring Boot的每个版本都与Spring框架的基础版本相关联。我们强烈建议您不要指定其版本。



策划列表包含您可以与Spring Boot一起使用的所有Spring模块，以及第三方库的精简列表。该列表以标准物料清单（spring-boot-dependencies）的形式提供，可用于 Maven 和 Gradle。

## 1.2 Maven

下面的文档详细介绍了如何在Spring Boot中使用Maven 

[https://docs.spring.io/spring-boot/docs/2.4.1/maven-plugin/reference/htmlsingle/](https://docs.spring.io/spring-boot/docs/2.4.1/maven-plugin/reference/htmlsingle/)

## 1.5 Starters

Starters是一组方便的依赖关系描述，您可以将其包含在您的应用程序中。您可以获得一站式服务，以获得您所需要的所有Spring和相关技术，而无需在示例代码中寻找和复制粘贴大量的依赖关系描述符。例如如果您想开始使用Spring和JPA进行数据库访问，请在您的项目中包含spring-boot-starter-data-jpa依赖关系。

启动程序包含了很多你需要的依赖关系，这些依赖关系可以让项目快速启动和运行，并且有一套一致的、受支持的管理的转义依赖关系。



> 关于自定义starter的命名
>
> 所有官方启动程序都遵循类似的命名模式；`spring-boot-starter-*`，其中*是一种特定的应用程序类型。这种命名结构是为了在你需要寻找启动程序时提供帮助。许多IDE中的Maven集成可以让你通过名称搜索依赖关系。
>
> 正如在 "创建您自己的启动器 "一节中所解释的那样，第三方启动器不应该以spring-boot开头，因为它是为官方的Spring Boot构件保留的。相反，第三方启动程序通常以项目名称开头。例如，一个名为`thirdpartyproject`的第三方启动项目通常会被命名为`thirdpartyproject-spring-boot-starter`。

| Name                                          | Description                                                  |
| :-------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter`                         | Core starter, including auto-configuration support, logging and YAML |
| `spring-boot-starter-activemq`                | Starter for JMS messaging using Apache ActiveMQ              |
| `spring-boot-starter-amqp`                    | Starter for using Spring AMQP and Rabbit MQ                  |
| `spring-boot-starter-aop`                     | Starter for aspect-oriented programming with Spring AOP and AspectJ |
| `spring-boot-starter-cache`                   | Starter for using Spring Framework’s caching support         |
| `spring-boot-starter-data-cassandra`          | Starter for using Cassandra distributed database and Spring Data Cassandra |
| `spring-boot-starter-data-cassandra-reactive` | Starter for using Cassandra distributed database and Spring Data Cassandra Reactive |
| `spring-boot-starter-data-couchbase`          | Starter for using Couchbase document-oriented database and Spring Data Couchbase |
| `spring-boot-starter-data-elasticsearch`      | Starter for using Elasticsearch search and analytics engine and Spring Data Elasticsearch |
| `spring-boot-starter-data-jdbc`               | Starter for using Spring Data JDBC                           |
| `spring-boot-starter-data-jpa`                | Starter for using Spring Data JPA with Hibernate             |
| `spring-boot-starter-data-mongodb`            | Starter for using MongoDB document-oriented database and Spring Data MongoDB |
| `spring-boot-starter-data-r2dbc`              | Starter for using Spring Data R2DBC                          |
| `spring-boot-starter-data-redis`              | Starter for using Redis key-value data store with Spring Data Redis and the Lettuce client |
| `spring-boot-starter-data-redis-reactive`     | Starter for using Redis key-value data store with Spring Data Redis reactive and the Lettuce client |
| `spring-boot-starter-data-rest`               | Starter for exposing Spring Data repositories over REST using Spring Data REST |
| `spring-boot-starter-data-solr`               | Starter for using the Apache Solr search platform with Spring Data Solr |
| `spring-boot-starter-freemarker`              | Starter for building MVC web applications using FreeMarker views |
| `spring-boot-starter-jdbc`                    | Starter for using JDBC with the HikariCP connection pool     |
| `spring-boot-starter-json`                    | Starter for reading and writing json                         |
| `spring-boot-starter-jta-bitronix`            | Starter for JTA transactions using Bitronix. Deprecated since 2.3.0 |
| `spring-boot-starter-mail`                    | Starter for using Java Mail and Spring Framework’s email sending support |
| `spring-boot-starter-mustache`                | Starter for building web applications using Mustache views   |
| `spring-boot-starter-oauth2-client`           | Starter for using Spring Security’s OAuth2/OpenID Connect client features |
| `spring-boot-starter-oauth2-resource-server`  | Starter for using Spring Security’s OAuth2 resource server features |
| `spring-boot-starter-quartz`                  | Starter for using the Quartz scheduler                       |
| `spring-boot-starter-security`                | Starter for using Spring Security                            |
| `spring-boot-starter-test`                    | Starter for testing Spring Boot applications with libraries including JUnit Jupiter, Hamcrest and Mockito |
| `spring-boot-starter-thymeleaf`               | Starter for building MVC web applications using Thymeleaf views |
| `spring-boot-starter-validation`              | Starter for using Java Bean Validation with Hibernate Validator |
| `spring-boot-starter-web`                     | Starter for building web, including RESTful, applications using Spring MVC. Uses Tomcat as the default embedded container |
| `spring-boot-starter-webflux`                 | Starter for building WebFlux applications using Spring Framework’s Reactive Web support |
| `spring-boot-starter-websocket`               | Starter for building WebSocket applications using Spring Framework’s WebSocket support |

| Name                                | Description                                                  |
| :---------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter-jetty`         | Starter for using Jetty as the embedded servlet container. An alternative to [`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-tomcat) |
| `spring-boot-starter-log4j2`        | Starter for using Log4j2 for logging. An alternative to [`spring-boot-starter-logging`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-logging) |
| `spring-boot-starter-logging`       | Starter for logging using Logback. Default logging starter   |
| `spring-boot-starter-reactor-netty` | Starter for using Reactor Netty as the embedded reactive HTTP server. |
| `spring-boot-starter-tomcat`        | Starter for using Tomcat as the embedded servlet container. Default servlet container starter used by [`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-web) |
| `spring-boot-starter-undertow`      | Starter for using Undertow as the embedded servlet container. An alternative to [`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-tomcat) |

# 2. 组织你的代码

spring不要求任何特殊的代码目录结构，但是有一些最佳实践。

## 2.1 default package

当一个类没有包含包声明时，它被认为是在 "默认包 "中。一般不鼓励使用 "默认包"，应该避免使用。对于使用@ComponentScan、@ConfigurationPropertiesScan、@EntityScan或@SpringBootApplication注解的Spring Boot应用程序来说，它可能会引起特别的问题，因为每个jar中的每个类都会被读取。

## 2.2 定位应用main主类

> We generally recommend that you locate your main application class in a root package above other classes. The [`@SpringBootApplication` annotation](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-using-springbootapplication-annotation) is often placed on your main class, and it implicitly defines a base “search package” for certain items. For example, if you are writing a JPA application, the package of the `@SpringBootApplication` annotated class is used to search for `@Entity` items. Using a root package also allows component scan to apply only on your project.
>
> If you don’t want to use `@SpringBootApplication`, the `@EnableAutoConfiguration` and `@ComponentScan` annotations that it imports defines that behaviour so you can also use those instead.

我们一般建议你将主应用类放在一个高于其他类的root包中。@SpringBootApplication注解通常放在你的主类上，它隐含地定义了某些项的基础 "搜索包"。例如，如果你正在编写一个JPA应用程序，@SpringBootApplication注解类的包就会被用来搜索@Entity项。使用根包还可以让组件扫描只应用在你的项目上。

如果你不想使用`@SpringBootApplication`，它导入的`@EnableAutoConfiguration`和`@ComponentScan`注解定义了这种行为，**所以你也可以使用这些来代替**。

下面是一个典型的代码布局：

```
com
 +- example
     +- myapplication
         +- Application.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

# 3. 配置类

Spring Boot倾向于使用基于Java的配置。尽管可以使用SpringApplication与XML源，但我们通常建议你的主要源是一个@Configuration类。通常，定义主方法的类是一个很好的候选人，作为主要的@Configuration。

互联网上已经发布了许多使用XML配置的Spring配置示例。如果可能的话，总是尽量使用等价的基于Java的配置。搜索Enable*注解可以是一个很好的起点。

## 3.1 导入额外配置类

你不需要把所有的@Configuration放到一个类中。可以使用@Import注解来导入额外的配置类。另外，您也可以使用@ComponentScan来自动拾取所有Spring组件，包括@Configuration类。

# 3.2 导入XML配置

如果你必须使用基于XML的配置，我们建议你还是从一个@Configuration类开始。然后你可以使用@ImportResource注解来加载XML配置文件。

# 4. 自动配置

Spring Boot 自动配置试图根据您添加的 jar 依赖关系自动配置您的 Spring 应用程序。例如，如果HSQLDB在您的classpath上，并且您没有手动配置任何数据库连接Bean，那么Spring Boot会自动配置内存数据库。

您需要通过向您的一个@Configuration类添加@EnableAutoConfiguration或@SpringBootApplication注解来选择加入自动配置。

您应该只添加一个@SpringBootApplication或@EnableAutoConfiguration注解。我们通常建议您只将其中一个或另一个添加到您的主 @Configuration 类中。

## 4.1 替换自动配置

自动配置是非侵入性的。在任何时候，你都可以开始定义自己的配置来替换自动配置的特定部分。例如，如果你添加了自己的DataSource bean，默认的嵌入式数据库支持就会退缩。

如果你需要找出当前正在应用的自动配置，以及为什么，请使用`--debug`开关启动你的应用程序。这样做可以为选定的核心记录器启用调试日志，并将条件报告记录到控制台。

## 4.2 禁用自动配置

如果你发现你不想要的特定自动配置类被应用，你可以使用@SpringBootApplication的exclude属性来禁用它们，如下例所示:

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;

@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
public class MyApplication {
}
```

如果该类不在classpath上，您可以使用注解的`excludeName`属性并指定完全限定的名称。如果您喜欢使用 `@EnableAutoConfiguration` 而不是 `@SpringBootApplication`，也可以使用 `exclude` 和 `excludeName`。最后，您还可以通过使用`spring.autoconfigure.exclude`属性来控制要排除的自动配置类的列表。

# 5. spring Bean和依赖注入

您可以自由使用任何标准的Spring框架技术来定义您的Bean及其注入的依赖关系。我们经常发现，使用@ComponentScan（查找您的bean）和使用@Autowired（进行构造函数注入）效果不错。

如果您按照上面的建议来组织您的代码（将您的应用程序类定位在一个 root package中），您可以使用没有任何参数的`@ComponentScan`。您的所有应用组件（@Component、@Service、@Repository、@Controller等）都会自动注册为Spring Beans。

下面的例子展示了一个@Service Bean，它使用构造函数注入来获取一个所需的RiskAssessor Bean。

```java
@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

如果Bean只有一个构造器，可以省略`@Autowired`：

```java
@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...
}
```

请注意，使用构造函数注入时让 riskAssessor 字段被标记为 `final`，表明它不能被随后更改。

# 6. 使用@SpringBootApplication

@SpringBootApplication有三个特性：

- `@EnableAutoConfiguration`: 开启自动配置 [Spring Boot’s auto-configuration mechanism](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-auto-configuration)
- `@ComponentScan`: enable `@Component` scan on the package where the application is located 
- `@Configuration`: 允许注册其他的Bean

```java
@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

# 8. Developer Tools

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

当运行一个完全打包的应用程序时，开发者工具会被自动禁用。如果你的应用程序是从java -jar启动的，或者是从一个特殊的classloader启动的，那么它被认为是一个 "生产应用程序"。你可以通过使用spring.devtools.restart.enabled系统属性来控制这种行为。要启用devtools，无论你的应用程序是由哪个classloader启动的，都要设置-Dspring.devtools.restart.enabled=true系统属性。在生产环境中，如果运行 devtools 有安全风险，则不能这样做。要禁用 devtools，请排除该依赖关系或设置 -Dspring.devtools.restart.enabled=false 系统属性

在 Maven 中将依赖标记为可选，可以防止 devtools 被顺便应用到使用你的项目的其他模块中。



Spring Boot提供的重启技术通过使用两个classloader工作。不改变的类（例如，来自第三方 jars 的类）被加载到基础类加载器中。您正在开发的类被加载到重启类加载器中。当应用程序重启时，重启类加载器会被扔掉，然后创建一个新的类加载器。这种方法意味着应用程序的重启通常比 "冷启动 "快得多，因为基础类加载器已经可用并被填充。

# 10. What to Read Next

您现在应该了解如何使用Spring Boot和一些您应该遵循的最佳实践。现在您可以继续深入了解Spring Boot的具体功能，或者您可以跳到前面，阅读Spring Boot的 "生产准备 "方面。