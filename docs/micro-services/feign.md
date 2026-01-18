feign是声明式的web客户端，使用起来很方便，类似于 [Retrofit](https://github.com/square/retrofit)。**Spring Cloud OpenFeign**是springboot通过自动配置集成了openFeign，并增加了springMVC的注解支持，使用了同样的`HttpMessageConverters` ，同时和ribbon/spring cloud loadBalance，0000000000.际会生成一个的load-balanced http client。

[spring cloud openFeign的官方文档说明](https://docs.spring.io/spring-cloud-openfeign/docs/2.2.5.RELEASE/reference/html/#spring-cloud-feign-inheritance)



因为Spring Cloud Netflix Ribbon处于维护阶段，所以建议使用springcloud loadbalance代替，只需要设置spring.cloud.loadbalancer.ribbon.enabled=false。实际两个都支持



如果整合了Eureka等注册中心后，会自动根据name属性去注册中心查找服务实例信息，如果不想使用可以通过[`SimpleDiscoveryClient`](https://cloud.spring.io/spring-cloud-static/spring-cloud-commons/current/reference/html/#simplediscoveryclient)配置。



```java
/**
 * Annotation for interfaces declaring that a REST client with that interface should be
 * created (e.g. for autowiring into another component). If ribbon is available it will be
 * used to load balance the backend requests, and the load balancer can be configured
 * using a <code>@RibbonClient</code> with the same name (i.e. value) as the feign client.
 */
public @interface FeignClient {

    /**
     * The name of the service with optional protocol prefix. 和name相同作用.
     * 无论url是否指定，必须配置name
     * 可以使用placeHolder的形式，如${propertyKey}.
     */
    @AliasFor("name")
    String value() default "";

    /**
     * The service id with optional protocol prefix. Synonym for {@link #value() value}.
     *
     * @deprecated use {@link #name() name} instead
     */
    @Deprecated
    String serviceId() default "";

    /**
     * The service id with optional protocol prefix. Synonym for {@link #value() value}.
     */
    @AliasFor("value")
    String name() default "";
    
    /**
     * Sets the <code>@Qualifier</code> value for the feign client.
     * 设置当前client bean的名字
     */
    String qualifier() default "";

    /**
     * An absolute URL or resolvable hostname (the protocol is optional).
     * 绝对URL或主机地址，可用于本机联调时暂时使用，
     * 可以使用placeHolder的形式，如${propertyKey}.
     */
    String url() default "";

    /**
     * Whether 404s should be decoded instead of throwing FeignExceptions
     */
    boolean decode404() default false;

    /**
     * A custom <code>@Configuration</code> for the feign client. Can contain override
     * <code>@Bean</code> definition for the pieces that make up the client, for instance
     * {@link feign.codec.Decoder}, {@link feign.codec.Encoder}, {@link feign.Contract}.
     * 可以为当前FeignClient自定义Configuration，会覆盖FeignClientsConfiguration的默认的Decoder，Encoder，Contract
     * 注意自定义的配置不需要添加@Component，否则会在所有FeignClient的生效
     * @see FeignClientsConfiguration for the defaults
     */
    Class<?>[] configuration() default {};

    /**
     * Fallback class for the specified Feign client interface. The fallback class must
     * implement the interface annotated by this annotation and be a valid spring bean.
     */
    Class<?> fallback() default void.class;

    /**
     * Define a fallback factory for the specified Feign client interface. The fallback
     * factory must produce instances of fallback classes that implement the interface
     * annotated by {@link FeignClient}. The fallback factory must be a valid spring
     * bean.
     *
     * @see feign.hystrix.FallbackFactory for details.
     */
    Class<?> fallbackFactory() default void.class;

    /**
     * 所有方法上的统一路径前缀
     */
    String path() default "";

    /**
     * Whether to mark the feign proxy as a primary bean. Defaults to true.
     */
    boolean primary() default true;

}
```

# `FeignClientsConfiguration，Feign的相关配置`

默认提供的bean有 DeCoder，Encoder，Logger，Contract，Feign.Builder，Client

没有默认提供，但只要提供了会自动加载的有Logger.Level，Retryer，ErrorDecoder Request.Options QueryMapEncoder  SetterFactory  Collection<RequestInterceptor>



为单独的FeignClient中添加下面配置，将会覆盖默认的bean。也可以使用配置文件（优先）。

添加@Configuration作用于所有的FeignClient

```java
// @Configuration 添加后所有的FeignClient生效
public class FooConfiguration {
    @Bean
    public Contract feignContract() {
        return new feign.Contract.Default();
    }

    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("user", "password");
    }
}
```



- Contract规定方法上哪些注解是valid的，springcloud默认的contract支持springmvc的注解而不是原生的Feign注解。
- 通过feign.okhttp.enabled和feign.httpclient.enabled可以控制使用OkHttpClient 还是 ApacheHttpClient
- Retry bean的类型为NEVER_RETRY时，将不会重试。默认的会在IOException，或者DeCoder抛出可重试的异常时会重试





- 添加Hystrix的依赖，并且配置`feign.hystrix.enabled=true`，会在所有的方法请求时，包装一个循环的breaker。



- 实际开发中基本要指定fallback ，如果想要获得异常的原因，可以使用`fallbackFactory` 。这样看肯定`fallbackFactory` 合适点，可以打印下异常



- 可以为Request和response开启压缩，

feign.compression.request.enabled=**true**

feign.compression.response.enabled=**true**

feign.compression.request.mime-types=text/xml,application/xml,application/json                 feign.compression.request.min-request-size=2048



- 默认将会使用FeignClient的接口创建一个Logger，Feign只响应Debug级别的日志。

Logger.Level有

NONE，默认，不打印

BASIC 只打印request method and URL and the response status code and execution time

HEADERS 在BASIC基础上再打印request和response的的header

FULL 为所有的request和response打印headers, body, and metadata

通过下列方式将为单独的FeignClient指定日志级别。

```java
@Configuration
public class FooConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```



- 使用@SpringQueryMap Params params可以用一个pojo来封装多个查询参数