> There are pros and cons for considering validation as business logic, and Spring offers a design for validation (and data binding) that does not exclude either one of them. Specifically, validation should not be tied to the web tier and should be easy to localize, and it should be possible to plug in any available validator. Considering these concerns, Spring provides a `Validator` contract that is both basic and eminently usable in every layer of an application.
>
> Data binding is useful for letting user input be dynamically bound to the domain model of an application (or whatever objects you use to process user input). Spring provides the aptly named `DataBinder` to do exactly that. The `Validator` and the `DataBinder` make up the `validation` package, which is primarily used in but not limited to the web layer.

将数据校验视为业务逻辑是有利有弊的，spring提供的数据校验和数据绑定设计，不排斥其中任何一种。具体来说，校验不应该绑定到web层，应该易于本地化，而且应该插入任何可用的验证器。考虑到这些问题，spring提供了Validator规范，属于基础规范，可以在应用的任何一层使用。

数据绑定对于让用户的输入动态绑定到应用的domain中非常有用。spring提供了一个恰如其分的名字DataBinder来实现这个功能。Validator和DataBinder组成了validation包，它主要用于但是不限于web层。

> The `BeanWrapper` is a fundamental concept in the Spring Framework and is used in a lot of places. However, you probably do not need to use the `BeanWrapper` directly. Because this is reference documentation, however, we felt that some explanation might be in order. We explain the `BeanWrapper` in this chapter, since, if you are going to use it at all, you are most likely do so when trying to bind data to objects.
>
> Spring’s `DataBinder` and the lower-level `BeanWrapper` both use `PropertyEditorSupport` implementations to parse and format property values. The `PropertyEditor` and `PropertyEditorSupport` types are part of the JavaBeans specification and are also explained in this chapter. Spring 3 introduced a `core.convert` package that provides a general type conversion facility, as well as a higher-level “format” package for formatting UI field values. You can use these packages as simpler alternatives to `PropertyEditorSupport` implementations. They are also discussed in this chapter.

BeanWrapper属于spring的基本概念，在很多地方都会用到。然而,你可能不需要直接使用BeanWrapper。然而由于这是参考文档，我们认为可能需要一些解释。我们在这一章介绍BeanWrapper，因为如果你想要使用它，你可能是在尝试将数据绑定到对象。

spring的DataBinder和底层的BeanWrapper都使用`java.beans.PropertyEditorSupport`实现来解析和格式化属性值。`PropertyEditor` 和`PropertyEditorSupport` 都是javaBean规范的一部分，本章节会进行介绍。spring 3 引入了`core.convert`包提供了一个通用的类型转换工具，和顶层的“format"包用于格式化UI字段。你可以使用这些包作为`PropertyEditorSupport` 的简单替代品，本章也会进行介绍。

> Spring supports Java Bean Validation through setup infrastructure and an adaptor to Spring’s own `Validator` contract. Applications can enable Bean Validation once globally, as described in [Java Bean Validation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-beanvalidation), and use it exclusively for all validation needs. In the web layer, applications can further register controller-local Spring `Validator` instances per `DataBinder`, as described in [Configuring a `DataBinder`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-binder), which can be useful for plugging in custom validation logic.

Spring通过设置基础架构和spring自己的`Validator` 适配器来支持java的Bean Validation 。应用程序可以像[Java Bean Validation](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-beanvalidation)描述的一样，在全局范围内一次启用Bean Validation，用于所有的validation需求。在web层，应用可以更进一步为每个DataBinder注册本地controller的Validator，这对于插入自定义的校验逻辑很有用。

# 3.1 使用spring的校验Validator接口

# 3.7 Java Bean Validation

spring框架提供了对 [Java Bean Validation](https://beanvalidation.org/)的支持。

## 3.7.1 Bean Validation总览

Bean Validation为java应用提供了一种通过约束声明和元数据进行验证的通用方式。要使用它，可以使用在model字段上注解，然后运行时强制执行。已经提供了一些内置约束，也可以定义自己的约束。

Bean Validation允许声明约束

```java
public class PersonForm {

    @NotNull
    @Size(max=64)
    private String name;

    @Min(0)
    private int age;
}
```

Bean Validation validator会基于声明的注解约束校验这个类的实例. 更多信息查看 [Bean Validation](https://beanvalidation.org/) 和[Hibernate Validator](https://hibernate.org/validator/) 文档。Bean Validation是用于数据校验的一套规范，
而Hibernate Validator是此规范的参考实现且对其进行了扩展。

如果想要如何把bean validation provider注册为spring bean，继续往下阅读。 

## 3.7.2 配置Bean Validation Provider

spring为Bean Validation API 提供了全面的支持，包括将Bean Validation Provider注册为spring bean。这允许你可以在需要校验的地方注入`javax.validation.ValidatorFactory` 或者`javax.validation.Validator`。

你可以使用LocalValidatorFactoryBean配置默认的validator为spring bean，如：

```java
import org.springframework.validation.beanvalidation.LocalValidatorFactoryBean;

@Configuration
public class AppConfig {

    @Bean
    public LocalValidatorFactoryBean validator() {
        return new LocalValidatorFactoryBean();
    }
}
```

前面例子中基本配置是使用默认的引导机制触发bean的验证初始化。像Hibernate validator这样的Bean Validation provider，希望在classpath中被自动检测到。

`LocalValidatorFactoryBean` 同时实现了 `javax.validation.ValidatorFactory` 和`javax.validation.Validator`, 也实现了 Spring的 `org.springframework.validation.Validator`。你可以将这些接口的引用注入到需要调用验证逻辑的bean中。

如果你更愿意使用Validator的API，可以直接注入java或spring的Validator使用。

```java
import javax.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;
}
```

```java
import org.springframework.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;
}
```

### 配置自定义的验证

每个bean validation验证包含两部分：

-  `@Constraint` 注解，用来声明约束和可配置的属性
-  `javax.validation.ConstraintValidator` 接口的实现类，具体的验证行为。

> To associate a declaration with an implementation, each `@Constraint` annotation references a corresponding `ConstraintValidator` implementation class. At runtime, a `ConstraintValidatorFactory` instantiates the referenced implementation when the constraint annotation is encountered in your domain model.
>
> By default, the `LocalValidatorFactoryBean` configures a `SpringConstraintValidatorFactory` that uses Spring to create `ConstraintValidator` instances. This lets your custom `ConstraintValidators` benefit from dependency injection like any other Spring bean.

为了将声明和实现关联起来，每个@Constraint注解都会引用一个对应的`ConstraintValidator` 实现类。在运行时，当在model对象中遇到你定义的约束注解时，`ConstraintValidatorFactory` 就会实例化这个validator。

默认情况下， `LocalValidatorFactoryBean` 配置了 `SpringConstraintValidatorFactory` ，使用Spring来创建`ConstraintValidator`实例。这允许你自定义的`ConstraintValidators`像其他spring bean一样可以进行依赖注入。

下面的例子展示了一个自定义的@Constraint注解，它包含一个相关的ConstraintValidator实现，它可以使用Spring进行依赖注入。

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MyConstraintValidator.class) // 引用具体的验证实现
public @interface MyConstraint {
}


import javax.validation.ConstraintValidator;
public class MyConstraintValidator implements ConstraintValidator {
	
    @Autowired; // 可以注入其他spring bean，进行业务逻辑方面的验证
    private Foo aDependency;

    // ...
}
```

下面列举两个开发中使用的案例：

特殊字符的校验：

```java
/**
 * 特殊字符注解
 *
 */
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.PARAMETER, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Constraint(validatedBy = {SpecialCharacterCustomerValidator.class})
public @interface SpecialCharacter {

    /**
     * 默认提示为 input.string.special.charactor， 中文可定位为“参数中包含了特殊字符”
     */
    String message() default "input.string.special.character";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    String pattern() default "[\\\\/:*?\"<|'>]";
}


/**
 * 特殊字符校验
 *
 */
public class SpecialCharacterCustomerValidator implements ConstraintValidator<SpecialCharacter, String> {
    private String pattern = "";

    @Override
    public void initialize(SpecialCharacter constraintAnnotation) {
        pattern = constraintAnnotation.pattern();
    }


    /**
     * @param value
     * @param context
     * @return true 不含特殊字符  false 含特殊字符
     */
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        boolean result = true;
        if (StringUtils.isNotEmpty(value)) {
            Pattern p = Pattern.compile(pattern);
            Matcher m = p.matcher(value);
            result = !m.find();
        }
        return result;
    }
}
```

枚举类的校验：

```java
import com.jun.sail.sailvalidate.validater.EnumCheckValidator;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = EnumCheckValidator.class)
public @interface EnumCheck {

        /**
         * 是否必填 默认是必填的
         */
        boolean required() default true;

        /**
         * 验证失败的消息
         */
        String message() default "枚举验证失败";

        /**
         * 分组的内容
         */
        Class<?>[] groups() default {};
    
        /**
         * 错误验证的级别
         */
        Class<? extends Payload>[] payload() default {};
    
        /**
         * 枚举的Class
         */
        Class<? extends Enum<?>> enumClass();
    
        /**
         * 枚举中的验证方法，也就是枚举类中必须存在validation()方法，也可以自定义
         */
        String enumMethod() default "validation";
}


/**
 * 自定义的枚举验证器，
 *
 */
public class EnumCheckValidator implements ConstraintValidator<EnumCheck, Object> {

    private static final Logger logger = LoggerFactory.getLogger(EnumCheckValidator.class);

    private EnumCheck enumCheck;

    @Override
    public void initialize(EnumCheck enumCheck) {
        this.enumCheck =enumCheck;
    }

    @Override
    public boolean isValid(Object value, ConstraintValidatorContext constraintValidatorContext) {
        // 注解表明为必选项 则不允许为空，否则可以为空
        if (value == null) {
            return !this.enumCheck.required();
        }

        Boolean result = Boolean.FALSE;
        Class<?> valueClass = value.getClass();
        try {
            //通过反射执行枚举类中的校验方法
            Method method = this.enumCheck.enumClass().getMethod(this.enumCheck.enumMethod(), valueClass);
            result = (Boolean)method.invoke(null, value);
            if(result == null){
                return false;
            }
        } catch (Exception e) {
            logger.error("custom EnumCheckValidator error", e);
        }
        return result;
    }
}
```



### Spring-driven Method Validation

>  You can integrate the method validation feature supported by Bean Validation 1.1 (and, as a custom extension, also by Hibernate Validator 4.3) into a Spring context through a `MethodValidationPostProcessor` bean definition:

您可以通过MethodValidationPostProcessor Bean定义将Bean Validation 1.1支持的方法验证功能集成到Spring上下文中（作为自定义扩展，也可以通过Hibernate Validator 4.3）。

```java
import org.springframework.validation.beanvalidation.MethodValidationPostProcessor;

@Configuration
public class AppConfig {

    @Bean
    public MethodValidationPostProcessor validationPostProcessor() {
        return new MethodValidationPostProcessor();
    }
}
```

> To be eligible for Spring-driven method validation, all target classes need to be annotated with Spring’s `@Validated` annotation, which can optionally also declare the validation groups to use. See [`MethodValidationPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/validation/beanvalidation/MethodValidationPostProcessor.html) for setup details with the Hibernate Validator and Bean Validation 1.1 providers.
>
>  Method validation relies on [AOP Proxies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-introduction-proxies) around the target classes, either JDK dynamic proxies for methods on interfaces or CGLIB proxies. There are certain limitations with the use of proxies, some of which are described in [Understanding AOP Proxies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-understanding-aop-proxies). In addition remember to always use methods and accessors on proxied classes; direct field access will not work.

为了符合Spring驱动的方法验证的条件，所有的目标类都需要用Spring的@Validated注解，它还可以选择性地声明要使用的验证组。请参阅[`MethodValidationPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/validation/beanvalidation/MethodValidationPostProcessor.html)，了解与Hibernate Validator和Bean Validation 1.1 providers的设置细节。
方法验证依赖于目标类的AOP代理，可以是接口中方法的JDK动态代理，也可以是CGLIB代理。使用代理有一定的限制，其中一些限制在理解AOP代理中有所描述。另外记得一定要使用代理类的方法和访问器，直接访问字段是不行的。

### 其他配置选项

> The default `LocalValidatorFactoryBean` configuration suffices for most cases. There are a number of configuration options for various Bean Validation constructs, from message interpolation to traversal resolution. See the [`LocalValidatorFactoryBean`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/validation/beanvalidation/LocalValidatorFactoryBean.html) javadoc for more information on these options.

默认的 "LocalValidatorFactoryBean "配置足以满足大多数情况，更多配置信息查看 [`LocalValidatorFactoryBean`](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/validation/beanvalidation/LocalValidatorFactoryBean.html) 。



## 3.7.3 配置`DataBinder`

从Spring 3 开始，可以为`DataBinder`配置Validator。配置后，就可以调用 `binder.validate()`.进行校验。任何校验错误Errors会自动添加到BindingResult。

下面展示如何使用`DataBinder`编程在绑定目标对象后进行校验逻辑

```java
Foo target = new Foo();
DataBinder binder = new DataBinder(target);
binder.setValidator(new FooValidator());

// bind to the target object
binder.bind(propertyValues);

// validate the target object
binder.validate();

// get BindingResult that includes any validation errors
BindingResult results = binder.getBindingResult();
```

也可以为`DataBinder`配置多个 `Validator` 实例，通过 `dataBinder.addValidators` and `dataBinder.replaceValidators`。This is useful when combining globally configured bean validation with a Spring `Validator` configured locally on a DataBinder instance. See [Spring MVC Validation Configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-validation).

## 3.7.4 spring-mvc 3 Validation

See [Validation](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-validation) in the Spring MVC chapter.
