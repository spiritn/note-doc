# 7. Null-safety

虽然java不允许使用它的类型系统来表达null-safety，但是spring框架现在在org.springframework.lang包下提供了以下注解，来允许你声明APIs和字段的null-safety。

- [`@Nullable`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/lang/Nullable.html): Annotation to indicate that a specific parameter, return value, or field can be `null`. 表示参数，返回值，字段可以为null的注解
- [`@NonNull`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/lang/NonNull.html): Annotation to indicate that a specific parameter, return value, or field cannot be `null` (not needed on parameters / return values and fields where `@NonNullApi` and `@NonNullFields` apply, respectively).这个注解表示指定的参数，返回值，字段不能为null（不需要在参数/返回值和字段上分别的使用`@NonNullApi`和`@NonNullFields`
- [`@NonNullApi`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/lang/NonNullApi.html): Annotation at the package level that declares non-null as the default semantics for parameters and return values.用在包级别的注解，默认声明参数和返回值不能为空
- [`@NonNullFields`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/lang/NonNullFields.html): Annotation at the package level that declares non-null as the default semantics for fields.用在包级别的注解，声明参数默认不能为空



spring框架本身使用了这些注解，但是他们可以在任何基于spring的java项目中使用，来声明null-safety的APIs,和可选的null-safe字段。泛型，varargs和数组还不支持，但应该很快会在未来版本中支持，最新信息查看[SPR-15942](https://jira.spring.io/browse/SPR-15942)。将来的版本中，会做一些微调，方法体中的类型null-safe不在本次版本功能中。

其他常见的库如reacter和spring Data也提供了类似的空安全声明，为spring开发者提供了一致的整体体验。

除了为spring项目提供明确的声明外，这些注解还可以被IDE开发工具用来提供与null-safety相关的警告，避免在运行时出现NullPointerException。

> Spring annotations are meta-annotated with [JSR 305](https://jcp.org/en/jsr/detail?id=305) annotations (a dormant but wide-spread JSR). JSR-305 meta-annotations let tooling vendors like IDEA or Kotlin provide null-safety support in a generic way, without having to hard-code support for Spring annotations.
>
> It is not necessary nor recommended to add a JSR-305 dependency to the project classpath to take advantage of Spring null-safe API. Only projects such as Spring-based libraries that use null-safety annotations in their codebase should add `com.google.code.findbugs:jsr305:3.0.2` with `compileOnly` Gradle configuration or Maven `provided` scope to avoid compile warnings.

spring注解使用了 [JSR 305](https://jcp.org/en/jsr/detail?id=305)  的元注解（一个暂停但是被广泛使用的JSR）。JSR-305 元注解让IDEA等以通用方式提供null-safety支持，无需为spring注解硬编码。也就是下面代码块中的@Nonnull(when = When.MAYBE)

```java
/**
 * A common Spring annotation to declare that annotated elements can be {@code null} under
 * some circumstance.
 *
 * <p>Leverages JSR-305 meta-annotations to indicate nullability in Java to common
 * tools with JSR-305 support and used by Kotlin to infer nullability of Spring API.
 *
 * <p>Should be used at parameter, return value, and field level. Methods override should
 * repeat parent {@code @Nullable} annotations unless they behave differently.
 *
 * <p>Can be used in association with {@code @NonNullApi} or {@code @NonNullFields} to
 * override the default non-nullable semantic to nullable.
 *
 * @author Sebastien Deleuze
 * @author Juergen Hoeller
 * @since 5.0
 * @see NonNullApi
 * @see NonNullFields
 * @see NonNull
 */
@Target({ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Nonnull(when = When.MAYBE)
@TypeQualifierNickname
public @interface Nullable {
}
```

不需要也不建议在项目中增加JSR-305的依赖，以利用spring的null-safe API。只有基于spring的项目在代码里使用null-safety注解，才应该添加`com.google.code.findbugs:jsr305:3.0.2`，并且配置`compileOnly`为`provided`来避免编译警告。

# 8. Data Buffers

> Java NIO provides `ByteBuffer` but many libraries build their own byte buffer API on top, especially for network operations where reusing buffers and/or using direct buffers is beneficial for performance. For example Netty has the `ByteBuf` hierarchy, Undertow uses XNIO, Jetty uses pooled byte buffers with a callback to be released, and so on. The `spring-core` module provides a set of abstractions to work with various byte buffer APIs as follows:

虽然Java NIO提供了ByteBuffer，但许多框架也构建了他们自己的byte buffer顶层API，特别对于网络操作，会重用buffers或者直接使用buffer对性能帮助很大。例如Netty的ByteBuf等级，Undertow使用XNIO，Jetty使用池化的带有回调的byte buffers等等。Spring-core模块提供了抽象的byte bufferAPIs，如下：

- [`DataBufferFactory`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#databuffers-factory) 创建data buffer.
- [`DataBuffer`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#databuffers-buffer) represents a byte buffer, which may be [pooled](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#databuffers-buffer-pooled).
- [`DataBufferUtils`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#databuffers-utils)  提供了data buffers一些实用的方法.
- [Codecs](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#codecs) decode or encode streams data buffer streams into higher level objects.

# 9. Logging

将Log4j 2.X或者Logback添加到classpath，不需要任何额外配置，Spring框架会自动适配你的选择。更多信息查看[Spring Boot Logging Reference Documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-logging).

```java
public class MyBean {
    private final Log log = LogFactory.getLog(getClass());
    // ...
}
```

# 10. 附录

下面展示核心容器的已存在的StartupSteps，

每个startup步骤的名字和详细信息并不是确定的合约，会随着具体的实现而发生改变。

核心容器中的应用启动阶段定义：

| Name                                           | Description                                                  | Tags                                                         |
| :--------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring.beans.instantiate`                     | Instantiation of a bean and its dependencies.                | `beanName` the name of the bean, `beanType` the type required at the injection point. |
| `spring.beans.smart-initialize`                | Initialization of `SmartInitializingSingleton` beans.        | `beanName` the name of the bean.                             |
| `spring.context.annotated-bean-reader.create`  | Creation of the `AnnotatedBeanDefinitionReader`.             |                                                              |
| `spring.context.base-packages.scan`            | Scanning of base packages.                                   | `packages` array of base packages for scanning.              |
| `spring.context.beans.post-process`            | Beans post-processing phase.                                 |                                                              |
| `spring.context.bean-factory.post-process`     | 调用实现了 `BeanFactoryPostProcessor` 的beans.               | `postProcessor` the current post-processor.                  |
| `spring.context.beandef-registry.post-process` | 调用实现了`BeanDefinitionRegistryPostProcessor`接口的 beans. | `postProcessor` the current post-processor.                  |
| `spring.context.component-classes.register`    | Registration of component classes through `AnnotationConfigApplicationContext#register`. | `classes` array of given classes for registration.           |
| `spring.context.config-classes.enhance`        | Enhancement of configuration classes with CGLIB proxies.     | `classCount` count of enhanced classes.                      |
| `spring.context.config-classes.parse`          | Configuration classes parsing phase with the `ConfigurationClassPostProcessor`. | `classCount` count of processed classes.                     |
| `spring.context.refresh`                       | Application context refresh phase.应用刷新阶段               |                                                              |
| `spring.event.invoke-listener`                 | Invocation of event listeners, if done in the main thread.调用事件监听器，如果在主线程中完成 | `event` the current application event, `eventType` its type and `listener` the listener processing this event. |


