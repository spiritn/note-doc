# 3.  Profiles

Spring Profiles提供了一种隔离应用程序配置部分的方法，使其仅在特定环境中可用。任何@Component、@Configuration或@ConfigurationProperties都可以用@Profile来标记，以限制它被加载的时间，如下例所示：

```java
@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

如果@ConfigurationProperties Bean是通过@EnableConfigurationProperties而不是自动扫描注册的，则需要在具有@EnableConfigurationProperties注解的@Configuration类上指定@Profile注解。在扫描@ConfigurationProperties的情况下，可以在@ConfigurationProperties类本身上指定@Profile。



你可以使用spring.profile.active环境属性来指定哪些配置文件是激活的。你可以用本章前面描述的任何一种方式来指定该属性。例如，您可以将其包含在您的application.properties中，如下例所示：

```properties
spring.profiles.active=dev,hsqldb
```

也可以在命令行中添加`--spring.profiles.active=dev,hsqldb`

## 3.1 增加激活的Profiles

> The spring.profiles.active property follows the same ordering rules as other properties: The highest PropertySource wins. This means that you can specify active profiles in application.properties and then replace them by using the command line switch.
>
> Sometimes, it is useful to have properties that add to the active profiles rather than replace them. The SpringApplication entry point has a Java API for setting additional profiles (that is, on top of those activated by the spring.profiles.active property). See the setAdditionalProfiles() method in SpringApplication. Profile groups, which are described in the next section can also be used to add active profiles if a given profile is active.

`spring.profile.active`属性与其他属性遵循相同的排序规则。最高的PropertySource获胜。这意味着你可以在application.properties中指定活动配置文件，然后在命令行里替换它们。

有时，添加属性到激活的配置文件而不是替换它们是有用的。SpringApplication入口点有一个Java API用于设置额外的配置文件（即在那些由spring.profile.active属性激活的配置文件之上）。参见SpringApplication中的`setAdditionalProfiles()`方法。如果一个给定的配置文件是活跃的，那么下一节中描述的Profile groups也可以用来添加活跃的配置文件。

## 3.2 Profile 组

有时，您在应用程序中定义和使用的配置文件过于细化，使用起来会变得很麻烦。例如，您可能会有 proddb 和 prodmq的profiles ，用于独立启用数据库和消息功能。

为了帮助解决这个问题，Spring Boot允许您定义配置文件组。配置文件组允许您为相关的配置文件组定义一个逻辑名称。

例如，我们可以创建一个由 proddb 和 prodmq 配置文件组成的生产组:

```properties
spring.profiles.group.production[0]=proddb
spring.profiles.group.production[1]=prodmq
```

现在可以使用`--spring.profile.active=production`启动我们的应用程序，一次性激活production、proddb和prodmq配置文件。

## 3.3 编程设置Profiles

您可以在应用程序运行前调用SpringApplication.setAdditionalProfiles(...)来编程设置活动的配置文件。也可以通过使用Spring的ConfigurableEnvironment接口来激活配置文件。

## 3.4 Profile-specific 配置文件

>  Profile-specific variants of both application.properties (or application.yml) and files referenced through @ConfigurationProperties are considered as files and loaded. See "Profile Specific Files" for details.

application.properties（或application.yml）和通过`@ConfigurationProperties`引用的配置文件，都被视为files并被加载。详情请参见 "配置文件特定文件"。

# 4. logging

Spring Boot对所有内部日志记录使用 [Commons Logging](https://commons.apache.org/logging) ，但开放底层日志实现。为 Java Util Logging、Log4J2 和 [Logback](https://logback.qos.ch/) 提供了默认配置。在每种情况下，日志记录器都被预先配置为使用控制台输出，也可选择文件输出。

默认情况下，如果您使用 "Starters"，将使用Logback。还包括适当的Logback路由，以确保使用Java Util Logging、Commons Logging、Log4J或SLF4J的依赖库都能正常工作。

Java有很多可用的日志框架。如果上面的列表看起来很混乱，不要担心。一般来说，你不需要改变你的日志依赖关系，Spring Boot的默认值也能正常工作。
当您将应用程序部署到 servlet 容器或应用程序服务器时，通过 Java Util Logging API 执行的日志不会被路由到应用程序的日志中。这就防止了由容器或已部署到容器的其他应用程序执行的日志记录出现在您应用程序的日志中。

## 4.1 日志格式

springboot默认的日志输出如下图：

```tex
2019-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2019-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2019-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

按照以下输出

- Date and Time：毫秒级别排序
- Log level：`ERROR`, `WARN`, `INFO`, `DEBUG`, 或者 `TRACE`
- 进程ID
- `--` 分隔符，来区分实际的日志输出
- 线程名称，使用中括号括起
- Logger name:通常是source class名称，一般会缩写
- 实际的日志信息



## 4.2 Console 输出

默认的日志配置在消息写入时将其回传到控制台。默认情况下，ERROR级别、WARN级别和INFO级别的消息会被记录下来。你也可以通过使用--debug标志启动你的应用程序来启用 "调试 "模式：

```
$ java -jar myapp.jar --debug
```

也可以在`application.properties`文件里debug=true开启

当启用调试模式时，核心日志器（嵌入式容器、Hibernate和Spring Boot）被配置为输出更多信息。启用调试模式不会将您的应用程序配置为记录所有具有DEBUG级别的消息。

或者，你可以通过使用--trace标志（或在application.properties中`trace=true`）来启用 "跟踪 "模式。这样做可以使选定的核心日志记录器（嵌入式容器、Hibernate模式生成和整个Spring组合）进行trace logging。

### 4.2.1彩色输出

如果你的终端支持ANSI，则会使用彩色输出来帮助阅读。你可以将`spring.output.ansi.enabled`设置为一个[supported value](https://docs.spring.io/spring-boot/docs/2.4.3/api/org/springframework/boot/ansi/AnsiOutput.Enabled.html)如always来覆盖自动检测。

颜色编码是通过使用`%clr`转换字来配置的。在其最简单的形式中，转换器根据日志级别对输出进行着色，如下例所示:`%clr(%5p)`

| Level   | Color  |
| :------ | :----- |
| `FATAL` | Red    |
| `ERROR` | Red    |
| `WARN`  | Yellow |
| `INFO`  | Green  |
| `DEBUG` | Green  |
| `TRACE` | Green  |

另外，您也可以通过为转换提供一个选项来指定应该使用的颜色或样式。例如，要使文本为黄色，请使用以下设置:`%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}`,支持以下颜色和格式：

- `blue`
- `cyan`
- `faint`
- `green`
- `magenta`
- `red`
- `yellow`

## 4.3 文件输出

springboot日志默认只会输出到控制台，不会记录到文件。如果想输出到文件需要设置`logging.file.name` 或`logging.file.path`属性（如application.properties文件）

| `logging.file.name` | `logging.file.path` | Example    | Description                                                  |
| :------------------ | :------------------ | :--------- | :----------------------------------------------------------- |
| *(none)*            | *(none)*            |            | 只输出到控制台.                                              |
| Specific file       | *(none)*            | `my.log`   | 写入指定的日志文件。名称可以是确切的位置，也可以是当前目录的相对位置。 |
| *(none)*            | Specific directory  | `/var/log` | 将 "spring.log "写入指定目录。名称可以是确切的位置，也可以是相对于当前目录的位置。 |

日志文件达到 10 MB 时会轮换，与控制台输出一样，默认情况下会记录 ERROR 级、WARN 级和 INFO 级消息。

> 日志属性独立于实际的日志实现。因此，spring Boot 不管理特定的配置键（如Logback的 Logback.configurationFile 属性 ）。

## 4.4 日志滚动

如果你使用的是Logback，那么可以使用你的application.properties或者application.yaml文件对日志轮换设置进行微调。对于其他日志系统，你需要自己直接配置轮换设置（例如，如果你使用Log4J2，那么你可以添加一个log4j.xml文件）。

支持以下轮换策略属性：

| Name                                                   | Description                                                  |
| :----------------------------------------------------- | :----------------------------------------------------------- |
| `logging.logback.rollingpolicy.file-name-pattern`      | 用来归档的文件名称pattern                                    |
| `logging.logback.rollingpolicy.clean-history-on-start` | 是否在应用程序启动时进行日志存档清理？                       |
| `logging.logback.rollingpolicy.max-file-size`          | The maximum size of log file 在归档前                        |
| `logging.logback.rollingpolicy.total-size-cap`         | The maximum amount of size log archives can take before being deleted.日志存档在被删除前所能承受的最大数量。 |
| `logging.logback.rollingpolicy.max-history`            | 归档的天数 (默认7天)                                         |

## 4.5 日志级别

所有支持的日志系统都可以通过使用`logging.level.<logger-name>=<level>`在Spring  Environment中设置日志器级别（例如在application.properties中），其中级别可以为TRACE、DEBUG、INFO、WARN、ERROR、FATAL或OFF。可以通过使用logging.level.root配置root。

下面的示例显示了application.properties中潜在的日志记录设置：

```properties
logging.level.root=warn
logging.level.org.springframework.web=debug
logging.level.org.hibernate=error
```

也可以使用环境变量来设置日志级别。例如，`LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB=DEBUG`将把org.springframework.web设置为DEBUG。

这种方式挺有用的，比如像单独调试追踪某个包下的问题，就可以**只开启某个包的日志级别为debug**，方便追踪问题！

> 以上方法只适用于package 包级日志。由于松弛绑定总是将环境变量转换为小写，所以不可能用这种方式为单个类配置日志记录。如果需要为一个类配置日志记录，可以使用SPRING_APPLICATION_JSON变量。

## 4.6 日志组

能够将相关的日志记录器分组在一起，以便在同一时间对它们进行配置，这通常很有用。例如，您可能通常会更改所有 Tomcat 相关日志程序的日志级别，但您无法轻松记住顶级包。

为了帮助解决这个问题，Spring Boot允许您在Spring环境中定义日志组。例如，您可以通过将其添加到您的 application.properties 中来定义一个 "Tomcat "组：

```properties
logging.group.tomcat=org.apache.catalina,org.apache.coyote,org.apache.tomcat
```

定义后，您可以通过一行改变组中所有包的级别：

```properties
logging.level.tomcat=trace
```

springboot包含以下预先定义好的日志组，开箱即用：

| Name | Loggers                                                      |
| :--- | :----------------------------------------------------------- |
| web  | `org.springframework.core.codec`, `org.springframework.http`, `org.springframework.web`, `org.springframework.boot.actuate.endpoint.web`, `org.springframework.boot.web.servlet.ServletContextInitializerBeans` |
| sql  | `org.springframework.jdbc.core`, `org.hibernate.SQL`, `org.jooq.tools.LoggerListener` |

## 4.7 使用日志关闭钩子

为了释放日志资源，通常在应用程序终止时停止日志系统是一个好主意。不幸的是，没有一种方法可以适用于所有应用程序类型。如果您的应用程序具有复杂的上下文层次结构或作为war文件部署，您将需要研究由底层日志系统直接提供的选项。例如，Logback 提供了上下文选择器，允许在自己的上下文中创建每个 Logger。

对于部署在自己JVM中的简单的 "单jar "应用程序，你可以使用logging.register-shutdown-hook属性。将`logging.register-shutdown-hook`设置为true将注册一个关机钩子，该钩子将在JVM退出时触发日志系统清理。

您可以在application.properties或application.yaml文件中设置该属性:

```properties
logging.register-shutdown-hook=true
```

## 4.8 自定义日志配置

可以通过在classpath上包含适当的库来激活各种日志系统，并可通过在classpath的根目录下或在以下Spring环境属性指定的位置提供合适的配置文件来进一步定制：logging.config。

您可以通过使用 o`rg.springframework.boot.logging.LoggingSystem` system 属性强制 Spring Boot 使用特定的日志系统。该值应该是 LoggingSystem 实现的完全限定类名。您也可以通过使用 none 的值完全禁用 Spring Boot 的日志配置。

> 由于日志记录是在ApplicationContext创建之前初始化的，所以无法通过Spring @Configuration文件中的@PropertySources来控制日志记录。唯一的方法是通过系统属性来改变日志系统或完全禁用它。

根据不同的日志系统实现，会加载不同的配置文件，如下表

| Logging System          | Customization                                                |
| :---------------------- | :----------------------------------------------------------- |
| **Logback**             | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

> When possible, we recommend that you use the `-spring` variants for your logging configuration (for example, `logback-spring.xml` rather than `logback.xml`). If you use standard configuration locations, Spring cannot completely control log initialization.

在可能的情况下，我们建议您使用-spring变体来进行日志配置（例如，**logback-spring.xml而不是logback.xml**）。如果您使用标准配置位置，Spring无法完全控制日志初始化。



为了帮助定制，一些其他属性从Spring Environment转移到系统属性，如下表所述:

| Spring Environment                  | System Property                 | Comments                                                     |
| :---------------------------------- | :------------------------------ | :----------------------------------------------------------- |
| `logging.exception-conversion-word` | `LOG_EXCEPTION_CONVERSION_WORD` | The conversion word used when logging exceptions.            |
| `logging.file.name`                 | `LOG_FILE`                      | 如果定义了，则在默认的日志配置中使用。                       |
| `logging.file.path`                 | `LOG_PATH`                      | If defined, it is used in the default log configuration.     |
| `logging.pattern.console`           | `CONSOLE_LOG_PATTERN`           | The log pattern to use on the console (stdout).              |
| `logging.pattern.dateformat`        | `LOG_DATEFORMAT_PATTERN`        | Appender pattern for log date format.                        |
| `logging.charset.console`           | `CONSOLE_LOG_CHARSET`           | The charset to use for console logging.                      |
| `logging.pattern.file`              | `FILE_LOG_PATTERN`              | The log pattern to use in a file (if `LOG_FILE` is enabled). |
| `logging.charset.file`              | `FILE_LOG_CHARSET`              | The charset to use for file logging (if `LOG_FILE` is enabled). |
| `logging.pattern.level`             | `LOG_LEVEL_PATTERN`             | The format to use when rendering the log level (default `%5p`). |
| `PID`                               | `PID`                           | The current process ID (discovered if possible and when not already defined as an OS environment variable). |

如果使用的是logback，还支持以下

| Spring Environment                                     | System Property                                | Comments                                                     |
| :----------------------------------------------------- | :--------------------------------------------- | :----------------------------------------------------------- |
| `logging.logback.rollingpolicy.file-name-pattern`      | `LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN`      | Pattern for rolled-over log file names (default `${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz`). |
| `logging.logback.rollingpolicy.clean-history-on-start` | `LOGBACK_ROLLINGPOLICY_CLEAN_HISTORY_ON_START` | Whether to clean the archive log files on startup.           |
| `logging.logback.rollingpolicy.max-file-size`          | `LOGBACK_ROLLINGPOLICY_MAX_FILE_SIZE`          | Maximum log file size.                                       |
| `logging.logback.rollingpolicy.total-size-cap`         | `LOGBACK_ROLLINGPOLICY_TOTAL_SIZE_CAP`         | Total size of log backups to be kept.                        |
| `logging.logback.rollingpolicy.max-history`            | `LOGBACK_ROLLINGPOLICY_MAX_HISTORY`            | Maximum number of archive log files to keep.                 |

所有支持的日志系统在解析其配置文件时都可以参考系统属性。请参阅spring-boot.jar中的默认配置示例：

- [Logback](https://github.com/spring-projects/spring-boot/tree/v2.4.3/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)
- [Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.4.3/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)
- [Java Util logging](https://github.com/spring-projects/spring-boot/tree/v2.4.3/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

> 如果您想在日志属性中使用占位符，您应该使用 Spring Boot 的语法，而不是底层框架的语法。值得注意的是，如果您使用 Logback，您应该使用 : 作为属性名与其默认值之间的分隔符，而不是使用 `:-`

> 您可以只通过覆盖LOG_LEVEL_PATTERN（或使用Logback的logging.pattern.level）来向日志行添加MDC和其他临时内容。例如，如果你使用`logging.pattern.level=user:%X{user} %5p`，那么默认的日志格式就包含了 "user "的MDC条目，如果它存在的话，如下例所示
>
> ```
> 2019-08-30 12:30:04.031 user:someone INFO 22174 --- [  nio-8080-exec-0] demo.Controller
> Handling authenticated request
> ```

## 4.9 logback扩展

Spring Boot 包含了许多有助于高级配置的 Logback 扩展。您可以在您的 logback-spring.xml 配置文件中使用这些扩展。

> 因为标准的logback.xml配置文件加载得太早，所以你不能在其中使用扩展。你需要使用logback-spring.xml或者定义一个logging.config属性。

这些扩展名不能与Logback的配置扫描一起使用。如果您试图这样做，对配置文件进行更改会导致类似于以下的错误记录。

### 4.9.2 Environment Properties

  <springProperty>标签让您可以从Spring Environment中公开属性，以便在Logback中使用。如果你想在Logback配置中访问application.property文件中的值，这样做很有用。该标签的工作方式与 Logback 的标准 <property> 标签类似。然而，你不是直接指定一个值，而是指定属性的来源（来自Environment）。如果你需要将属性存储在本地范围以外的地方，你可以使用 scope 属性。如果你需要一个后备值（在环境中没有设置属性的情况下），你可以使用defaultValue属性。下面的示例展示了如何在 Logback 中公开属性。

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
        defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```

下面是实际参考案例

```xml
<define name="appId" class="com.zhangmen.arch.logger.component.common.AppIdDefiner"/>

    <springProperty scope="context" name="springAppName" source="spring.application.name"/>

    <springProperty scope="context" name="logSwitch" source="port.http.server"/>

    <!-- log maxFileSize -->
    <springProperty scope="context" name="MAX_FILE_SIZE" source="spring.logback.max-file-size" defaultValue="100MB"/>

    <!-- log maxIndex -->
    <springProperty scope="context" name="MAX_INDEX" source="spring.logback.max-index" defaultValue="5"/>

    <springProperty scope="context" name="MASK_FIELDS" source="log.mask.fields" defaultValue="message,input_param,output_param,header_param"/>

    <springProperty scope="context" name="MASK_REGEX_TYPES" source="log.mask.regex.types" defaultValue="mobile"/>

    <springProperty scope="context" name="MASK_REGEX_MATCH_MAX_DEPTH" source="log.mask.regex.match.max.depth" defaultValue="12"/>

    <springProperty scope="context" name="MASK_REGEX_MATCH_MAX_LENGTH" source="log.mask.regex.match.max.length" defaultValue="2048"/>

```

source可以来自配置文件和系统属性

# 5. 国际化

Spring boot支持本地化信息，所以你的程序可以满足不同语言的用户。默认Spring Boot会在classpath寻找resource bundle的messages。

> 自动配置适用于已配置的 resource bundle的默认属性文件可用时（即默认的 messages.properties）。如果您的 resource bundle只包含特定语言的属性文件，您需要添加默认的。如果没有找到与任何配置的基础名称相匹配的属性文件，则不会有自动配置的MessageSource。

resource bundle资源包的basename 以及其他一些属性可以使用spring.messages命名空间进行配置，如下例所示:

```properties
spring.messages.basename=messages,config.i18n.messages
spring.messages.fallback-to-system-locale=false
```

`spring.messages.basename`支持以逗号分隔的，可以是包限定符，也可以是从classpath根目录解析的资源。

更多细节查看 [`MessageSourceProperties`](https://github.com/spring-projects/spring-boot/tree/v2.4.3/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/context/MessageSourceProperties.java)

# 6. JSON

spring提供了三种JSON库的集成，包括Gson，jackson, JSON-B，

Jackson是首选和默认的库

## 6.1 Jackson

Jackson 的自动配置已提供，是`spring-boot-starter-json`的一部分。当Jackson 在类路径时，ObjectMapper会被自动配置，配置项参看 [customizing the configuration of the `ObjectMapper`](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-customize-the-jackson-objectmapper)。

## 6.2 Gson

Gson自动配置已提供，这里不再赘述。

## 6.3  JSON-B

JSON-B自动配置已提供，这里不再赘述。



# 8. 优雅的shutdown服务

所有四种嵌入式Web服务器（Jetty、Reactor Netty、Tomcat和Undertow）以及反应式和基于Servlet的Web应用程序都支持优雅关闭。它作为关闭应用程序上下文的一部分发生，并在停止SmartLifecycle beans的最早阶段执行。这个停止处理使用了一个超时，它提供了一个宽限期，在这个宽限期内，现有的请求将被允许完成，但不允许有新的请求。不允许新请求的具体方式根据使用的Web服务器而不同。Jetty、Reactor Netty和Tomcat将在网络层停止接受请求。Undertow将接受请求，但会立即以服务不可用（503）响应。

Graceful shutdown 要求Tomcat 版本在9.0.33+。

开启优雅的停止服务，需要配置

```properties
server.shutdown=graceful
```

配置过期时间如：

```properties
spring.lifecycle.timeout-per-shutdown-phase=20s
```

注意：Using graceful shutdown with your IDE may not work properly if it does not send a proper `SIGTERM` signal.

# 13. Caching

Spring框架提供了对应用程序透明地添加缓存的支持。其核心是将缓存应用于方法，从而基于缓存中的可用信息减少方法的执行次数。缓存逻辑是透明的，对调用者没有任何干扰。只要通过@EnableCaching注解启用了缓存，Spring Boot就会自动配置缓存基础架构。

更多细节参见Spring的[relevant section](https://docs.spring.io/spring/docs/5.3.3/reference/html/integration.html#cache) 。

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Component;

@Component
public class MathService {

    @Cacheable("piDecimals")
    public int computePiDecimal(int i) {
        // ...
    }

}
```

这个例子演示了在一个潜在的耗时操作上使用缓存。在调用computePiDecimal方法之前，抽象在piDecimals缓存中寻找与i参数匹配的条目。如果找到了一个条目，缓存中的内容会立即返回给调用者，该方法不会被调用。否则，该方法将被调用，并在返回之前更新缓存。

> 您也可以透明地使用标准的JSR-107（JCache）注解（如@CacheResult）。然而，我们强烈建议您不要混合和匹配Spring Cache和JCache注解。

如果您没有添加任何特定的缓存库，Spring Boot 会自动配置一个基于内存的concurrent map的简单提供者。当需要缓存时（如前面例子中的piDecimals），该提供者会为您创建缓存。这个简单的提供者并不推荐用于生产用途，但它对于入门和确保你了解其功能是非常好的。当您决定使用缓存提供者时，请务必阅读它的文档，以了解如何配置您的应用程序使用的缓存。几乎所有的提供者都要求你明确配置你使用的每个缓存。有些提供了一种方法来定制由spring.cache.cache-name属性定义的默认缓存。

>   当然也提供了缓存的 [update](https://docs.spring.io/spring/docs/5.3.3/reference/html/integration.html#cache-annotations-put) 或 [evict](https://docs.spring.io/spring/docs/5.3.3/reference/html/integration.html#cache-annotations-evict) 。

## 13.1 缓存提供者的支持

缓存抽象并不提供实际的存储，而是依赖于 `org.springframework.cache.Cache` 和 `org.springframework.cache.CacheManager` 接口所实现的抽象。

如果您没有定义`CacheManager`类型的bean或名为`cacheResolver`的CacheResolver（参见 [`CachingConfigurer`](https://docs.spring.io/spring/docs/5.3.3/javadoc-api/org/springframework/cache/annotation/CachingConfigurer.html)），Spring Boot会尝试检测以下提供者（按指定顺序）:

1. [Generic](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-generic)
2. [JCache (JSR-107)](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-jcache) (EhCache 3, Hazelcast, Infinispan, and others)
3. [EhCache 2.x](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-ehcache2)
4. [Hazelcast](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-hazelcast)
5. [Infinispan](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-infinispan)
6. [Couchbase](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-couchbase)
7. [Redis](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-redis)
8. [Caffeine](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-caffeine)
9. [Simple](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-caching-provider-simple)

> 也可以通过设置`spring.cache.type`属性来强制使用特定的缓存提供者。如果你需要在特定的环境中（比如测试）完全禁止缓存，请使用这个属性。



>  使用spring-boot-starter-cache, "starter "会快速添加基本的缓存依赖。引入spring-context-support。如果不使用starter即手动添加依赖关系，你必须包含spring-context-support才能使用JCache、EhCache 2.x或Caffeine支持。

如果CacheManager是由Spring Boot自动配置的，您可以在它完全初始化之前，通过实现`CacheManagerCustomizer`接口的Bean来进一步调整它的配置。下面的例子设置了一个标志，表示null值应该被传递给底层map:

```java
@Bean
public CacheManagerCustomizer<ConcurrentMapCacheManager> cacheManagerCustomizer() {
    return new CacheManagerCustomizer<ConcurrentMapCacheManager>() {
        @Override
        public void customize(ConcurrentMapCacheManager cacheManager) {
            cacheManager.setAllowNullValues(false);
        }
    };
}
```

在上面的例子中，我们希望有一个自动配置的 ConcurrentMapCacheManager。如果不是这样的话(你提供了自己的配置或者是自动配置了其他的缓存提供者)，定制器根本不会被调用。你可以拥有任意多的自定义器，你也可以通过使用@Order或Ordered来指定顺序。

### 13.1.1 Generic

如果上下文定义了至少一个`org.springframework.cache.Cache` bean，则使用通用缓存。会创建一个包裹该类型所有bean的CacheManager。

### 13.1.7 Redis

> If [Redis](https://redis.io/) is available and configured, a `RedisCacheManager` is auto-configured. It is possible to create additional caches on startup by setting the `spring.cache.cache-names` property and cache defaults can be configured by using `spring.cache.redis.*` properties. For instance, the following configuration creates `cache1` and `cache2` caches with a *time to live* of 10 minutes:

如果Redis可用并配置了，则会自动配置RedisCacheManager。可以在启动时通过设置`spring.cache.cache-name`属性来创建额外的缓存，缓存默认值可以通过`spring.cache.redis.*`属性来配置。例如，下面的配置会创建cache1和cache2缓存，生存时间为10分钟:

```properties
spring.cache.cache-names=cache1,cache2
spring.cache.redis.time-to-live=10m
```

默认情况下，会添加一个键前缀，这样，即使两个独立的缓存使用相同的键，Redis也不会有重叠的键，不会返回无效的值。如果你创建了自己的RedisCacheManager，我们强烈建议保持这个设置。

你可以通过`RedisCacheConfiguration` 的@Bean来完全控制默认配置。如果你想定制默认的序列化策略，这很有用。

如果你需要控制更多的配置，可以考虑注册一个`RedisCacheManagerBuilderCustomizer` bean。下面的例子显示了一个自定义器，它为cache1和cache2配置了一个特定的生存时间。

```java
@Bean
public RedisCacheManagerBuilderCustomizer myRedisCacheManagerBuilderCustomizer() {
    return (builder) -> builder
            .withCacheConfiguration("cache1",                   RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofSeconds(10)))
            .withCacheConfiguration("cache2",
              RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(1)));

}
```

### 13.1.8 Caffeine

Caffeine是Java 8对Guava缓存的重写，取代了对Guava的支持。如果存在Caffeine，则会自动配置一个`CaffeineCacheManager`（由`spring-boot-starter-cache` "Starter "提供）。缓存可以在启动时通过设置`spring.cache.cache-names`属性来创建，并且可以通过以下之一来定制（按指示顺序）:

1.  `spring.cache.caffeine.spec`定义的缓存
2.  `com.github.benmanes.caffeine.cache.CaffeineSpec` bean所定义的
3.  `com.github.benmanes.caffeine.cache.Caffeine` bean所定义的

例如，以下配置创建了cache1和cache2缓存，最大大小为500，生存时间为10分钟。

```java
spring.cache.cache-names=cache1,cache2
spring.cache.caffeine.spec=maximumSize=500,expireAfterAccess=600s
```

如果定义了`com.github.benmanes.caffeine.cache.CacheLoader` bean，它将自动关联到CaffeineCacheManager。由于CacheLoader将与缓存管理器管理的所有缓存相关联，所以它必须被定义为CacheLoader<Object, Object>。自动配置会忽略任何其他通用类型。

[参考文章](https://www.cnblogs.com/rickiyang/p/11074158.html)

### 13.1.9 Simple

如果找不到其他的提供者，则使用ConcurrentHashMap作为缓存存储。如果您的应用程序中没有缓存库，这是默认的。默认情况下，缓存是根据需要创建的，但您可以通过设置cache-name属性来限制可用缓存的列表。例如，如果你只想要cache1和cache2缓存，则设置cache-name属性如下。

```properties
spring.cache.cache-names=cache1,cache2
```

如果您这样做，并且您的应用程序使用了未列出的缓存，那么当需要缓存时，它会在运行时失败，但在启动时不会。这与 "真正的 "缓存提供者在使用未声明的缓存时的行为方式类似。

### 13.1.10 None

当你的配置中存在@EnableCaching时，也希望有合适的缓存配置。如果你需要在某些环境中完全禁止缓存，可以强制缓存类型为none，使用无操作的实现，如下例所示。

```properties
spring.cache.type=none
```



# 17. Validation

> The method validation feature supported by Bean Validation 1.1 is automatically enabled as long as a JSR-303 implementation (such as Hibernate validator) is on the classpath. This lets bean methods be annotated with `javax.validation` constraints on their parameters and/or on their return value. Target classes with such annotated methods need to be annotated with the `@Validated` annotation at the type level for their methods to be searched for inline constraint annotations.

只要classpath上有JSR-303d的实现（如Hibernate validator），Bean Validation 1.1支持的方法验证功能就会自动启用。这使得bean方法可以在其参数和/或返回值上使用javax.validation约束进行注解。具有这种注解方法的目标类需要在类上添加@Validated注解，以便搜索其方法的内联约束注解。

下面的代码会触发第一个参数的校验，确保其大小在8到10之间：

```java
@Service
@Validated
public class MyBean {

    public Archive findByCodeAndAuthor(@Size(min = 8, max = 10) String code,
            Author author) {
        ...
    }

}
```

# 29. 创建自定义的自动配置

如果你在一个开发共享库的公司工作，或者你在一个开源或商业库上工作，你可能想开发自己的自动配置。自动配置类可以捆绑在外部jars中，但仍然可以被Spring Boot接收。

自动配置可以关联到一个 "starter"，该starter提供自动配置代码以及您将使用的依赖库。我们首先介绍您需要知道的构建您自己的自动配置，然后我们继续介绍创建自定义启动器所需的典型步骤。

这里有一个参考示例：[demo project](https://github.com/snicoll-demos/spring-boot-master-auto-configuration)

## 29.1 理解 Auto-configured的Bean

在外壳下，自动配置是通过标准的`@Configuration`类实现的。额外的@Conditional注解用于限制自动配置何时应用。通常，自动配置类使用 @ConditionalOnClass 和 @ConditionalOnMissingBean 注解。这确保了自动配置只在找到相关的类并且你没有声明自己的@Configuration时才会应用。

你可以浏览spring-boot-autoconfigure的源代码来查看Spring提供的@Configuration类（参见[`META-INF/spring.factories`](https://github.com/spring-projects/spring-boot/tree/v2.4.2/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories)文件）。

## 29.2 查找自动配置的候选人

springboot会在发布的jar包中寻找`META-INF/spring.factories`文件，这个文件会列出EnableAutoConfiguration这个key对应value，

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

自动配置必须只能以这种方式加载。确保它们被定义在一个特定的包空间中，并且它们绝不是组件扫描的目标。此外，自动配置类不应该启用组件扫描来寻找额外的组件。应该使用特定的@Imports来代替。

> You can use the @AutoConfigureAfter or @AutoConfigureBefore annotations if your configuration needs to be applied in a specific order. For example, if you provide web-specific configuration, your class may need to be applied after WebMvcAutoConfiguration.
>
> If you want to order certain auto-configurations that should not have any direct knowledge of each other, you can also use @AutoConfigureOrder. That annotation has the same semantic as the regular @Order annotation but provides a dedicated order for auto-configuration classes.
>
> As with standard @Configuration classes, the order in which auto-configuration classes are applied only affects the order in which their beans are defined. The order in which those beans are subsequently created is unaffected and is determined by each bean’s dependencies and any @DependsOn relationships.

如果您的配置需要以特定的顺序应用，您可以使用 `@AutoConfigureAfter` 或 @AutoConfigureBefore 注解。例如，如果您提供特定于 Web 的配置，您的类可能需要在 WebMvcAutoConfiguration 之后应用。

如果您想对某些不应该有任何直接知识的自动配置进行排序，您也可以使用@AutoConfigureOrder。该注解与常规的 @Order 注解具有相同的语义，但为自动配置类提供了一个专用的顺序。

与标准的 @Configuration 类一样，应用自动配置类的顺序只影响其 bean 的定义顺序。这些Bean随后被创建的顺序不受影响，它由每个Bean的依赖关系和任何@DependsOn关系决定。

## 29.3 条件注解Condition 

你几乎总是希望在你的自动配置类中包含一个或多个@Conditional注解。`@ConditionalOnMissingBean` 注解是一个常见的例子，用于允许开发人员在对您的默认配置不满意时进行覆盖。

Spring Boot包含了许多@Conditional注解，您可以在自己的代码中应用于@Configuration类或单个@Bean方法来重用这些注解。这些注解包括：

- [Class Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-class-conditions)
- [Bean Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-bean-conditions)
- [Property Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-property-conditions)
- [Resource Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-resource-conditions)
- [Web Application Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-web-application-conditions)
- [SpEL Expression Conditions](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spel-conditions)

### 29.3.1 Class 条件注解

@ConditionalOnClass和@ConditionalOnMissingClass注解让@Configuration类基于特定类的存在或缺失而加载。由于注解元数据是通过使用 ASM 来解析的，因此您可以使用 value 属性来引用真正的类，即使该类实际上可能不会出现在正在运行的应用程序 classpath 上。如果你喜欢使用String值来指定类名，也可以使用name属性。

这种机制并不适用@Bean方法，通常返回类型是条件的目标：在方法上的条件应用之前，JVM将已经加载了类，并可能处理方法引用，如果类不存在，这些方法将失败。

为了处理这种情况，可以使用一个单独的@Configuration类来隔离条件，如下例所示：

```java
@Configuration(proxyBeanMethods = false)
// Some conditions
public class MyAutoConfiguration {

    // Auto-configured beans
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(EmbeddedAcmeService.class)
    static class EmbeddedConfiguration {

        @Bean
        @ConditionalOnMissingBean
        public EmbeddedAcmeService embeddedAcmeService() { ... }

    }

}
```

如果你使用@ConditionalOnClass或@ConditionalOnMissingClass作为元注解的一部分来组成你自己的注解，你必须使用name作为指代类，在这种情况下是不处理的。

### 29.3.2 Bean条件注解

@ConditionalOnBean和@ConditionalOnMissingBean注解让一个bean基于特定bean的存在或缺失而被包含。您可以使用 value 属性按类型指定 bean，或使用 name 按名称指定 bean。搜索属性可以让您限制搜索bean时应考虑的ApplicationContext层次结构。

当放在@Bean方法上时，目标类型默认为该方法的返回类型，如下例所示。

```java
@Configuration(proxyBeanMethods = false)
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService() { ... }

}
```

在这个例子中，如果应用ApplicationContext不包含一个MyService的Bean，则这个myService这个Bean就会创建。

你需要注意添加的bean定义顺序，因为这些条件是基于到目前为止所处理的内容来评估的。出于这个原因，我们建议在自动配置类上只使用@ConditionalOnBean和@ConditionalOnMissingBean注解（因为这些注解保证在任何用户定义的bean定义被添加后加载）。

@ConditionalOnBean和@ConditionalOnMissingBean不会阻止@Configuration类的创建。在类上使用这些条件和用注解标记每个包含的@Bean方法之间的唯一区别是，如果条件不匹配，前者会阻止@Configuration类注册为bean。



在声明一个@Bean方法时，在方法的返回类型中提供尽可能多的类型信息。例如，如果你的Bean的具体类实现了一个接口，那么Bean方法的返回类型应该是具体类而不是接口。在@Bean方法中提供尽可能多的类型信息在使用bean条件时尤为重要，因为它们的评估只能依赖于方法签名中可用的类型信息。

### 29.3.3 Property 条件注解

@ConditionalOnProperty注解允许基于Spring环境属性的配置被加载。使用前缀和名称属性来指定应检查的属性。默认情况下，任何存在且不等于false的属性都会被匹配。您还可以通过使用havingValue和matchIfMissing属性创建更高级的检查。

### 29.3.4 Resource 条件注解

通过@ConditionalOnResource注解，可以只在存在特定资源时才包含配置。可以使用通常的Spring约定来指定资源，如下例所示：`file:/home/user/test.dat`。

### 29.3.5 Web应用条件注解

`@ConditionalOnWebApplication`和`@ConditionalOnNotWebApplication`注解允许根据应用程序是否是 "Web应用程序 "来包含配置。基于servlet的Web应用程序是任何使用Spring WebApplicationContext、定义session范围或具有`ConfigurableWebEnvironment`。reactive Web应用是指任何使用ReactiveWebApplicationContext或拥有ConfigurableReactiveWebEnvironment。

@ConditionalOnWarDeployment 注解允许根据应用程序是否是部署到容器中的传统 WAR 应用程序来包含配置。对于使用嵌入式服务器运行的应用程序，这个条件将不匹配。

### 29.3.6 SpEL表达式

基于SpEL表达式的结果来加载配置

## 29.4 测试你的自动配置

一个自动配置可能会受到许多因素的影响：用户配置（@Bean定义和自定义的Environment ），条件评估（特定库的存在）等等。具体来说，每个测试都应该创建一个定义良好的ApplicationContext，代表这些自定义的组合。ApplicationContextRunner提供了一个很好的方法来实现这个目标。

`ApplicationContextRunner`通常被定义为测试类的一个字段，用来收集基础的、常用的配置。下面的例子确保`UserServiceAutoConfiguration`总是被调用:

```java
private final ApplicationContextRunner contextRunner = new ApplicationContextRunner()
        .withConfiguration(AutoConfigurations.of(UserServiceAutoConfiguration.class));
```

​	如果必须定义多个自动配置，则无需对它们的声明进行排序，因为它们的调用顺序与运行应用程序时完全相同。

每个测试都可以使用runner 来代表一个特定的用例。例如，下面的示例调用了一个用户配置（UserConfiguration），并检查自动配置是否正确回退。调用run提供了一个回调上下文，可以和AssertJ一起使用。

```java
@Test
void defaultServiceBacksOff() {
    this.contextRunner.withUserConfiguration(UserConfiguration.class).run((context) -> {
        assertThat(context).hasSingleBean(UserService.class);
    assertThat(context).getBean("myUserService").isSameAs(context.getBean(UserService.class));
    });
}

@Configuration(proxyBeanMethods = false)
static class UserConfiguration {

    @Bean
    UserService myUserService() {
        return new UserService("mine");
    }

}
```

## 29.5 springboot的自动配置原理

1. 启动入口main方法启动，里面调用SpringApplication.的静态*run()，最重要的一个方法*`this.refreshContext(context);`，容器还会在启动各个阶段会发布相应的event。
2. 在refreshCcontext()方法中加载对应的配置类等，最关键的@SpringBootApplication注解包含三个关键的注解，最重要的是**@**EnableAutoConfiguration

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```

1. 有一个AutoConfigurationImportSelector，非常非常关键，它的作用是去加载相应的自动配置类

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
}
```

1. AutoConfigurationImportSelector里利用一个类加载器SpringFactoriesLoader去加载META-INF/spring.factories配置文件下所有的自动配置类，
2. 自动配置类里基本都有一个ConditionalOnClass注解，也就是只有引入相应的依赖，才会去加载当前配置类。并且会替你生成默认的RedisTemplate，当然用户可以覆盖它。并且注意EnableConfigurationProperties，这就是为什么有默认的配置项

```java
@Configuration(proxyBeanMethods  = false) 
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {
    
@Bean
@ConditionalOnMissingBean(name = "redisTemplate")
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
        throws UnknownHostException {
    RedisTemplate<Object, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}

@Bean
@ConditionalOnMissingBean
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
        throws UnknownHostException {
    StringRedisTemplate template = new StringRedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}
}
```

# 学习如何编写自己的starter

1. 首先是编写AutoConfiguration自动配置类

```java
@Configuration(proxyBeanMethods = false) // 指定这是一个配置类
@ConditionalOnClass(KafkaTemplate.class) // 在指定类存在时自动配置类生效
@EnableConfigurationProperties(KafkaProperties.class) // 生效相关的Properties，方便引入使用
@Import({ KafkaAnnotationDrivenConfiguration.class, KafkaStreamsAnnotationDrivenConfiguration.class }) // 导入其他配置类
public class KafkaAutoConfiguration {

  private final KafkaProperties properties;

  public KafkaAutoConfiguration(KafkaProperties properties) {
    this.properties = properties;
  }
  
  	@Bean
    @ConditionalOnMissingBean(KafkaTemplate.class) // 提供默认的配置，使用者可以覆盖
    public KafkaTemplate<?, ?> kafkaTemplate(ProducerFactory<Object, Object> kafkaProducerFactory,
            ProducerListener<Object, Object> kafkaProducerListener,
            ObjectProvider<RecordMessageConverter> messageConverter) {
        KafkaTemplate<Object, Object> kafkaTemplate = new KafkaTemplate<>(kafkaProducerFactory);
        messageConverter.ifUnique(kafkaTemplate::setMessageConverter);
        kafkaTemplate.setProducerListener(kafkaProducerListener);
        kafkaTemplate.setDefaultTopic(this.properties.getTemplate().getDefaultTopic());
        return kafkaTemplate;
    }
  
  	@Bean
    @ConditionalOnProperty(name = "spring.kafka.producer.transaction-id-prefix")
    @ConditionalOnMissingBean
    public KafkaTransactionManager<?, ?> kafkaTransactionManager(ProducerFactory<?, ?> producerFactory) {
        return new KafkaTransactionManager<>(producerFactory);
    }

}
```

2. 要能加载到上面我们编写的自动配置类，将上面的自动配置类的全路径放在META-INF/spring.factority中，springboot启动时才能加载到我们自定义的配置类

```
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
```



1. starter规范
   starter模块是一个空的jar文件，仅提供辅助性的依赖管理，和保护依赖

   一般专门写一个自动配置模块，是真正实现功能的地方，starter依赖这个。别人使用只要依赖starter即可。

2. 命令规范：
   官方的starter命名都是spring-boot-starter-{name}，如spring-boot-starter-web，spring-boot-starter-test。
   自定义的starter一般建议{name}-spring-boot-starter，如mybatis-plus-boot-starter

   使用者只需要导入starter的依赖，然后添加配置文件（会提供默认值），然后就可以注入service使用了