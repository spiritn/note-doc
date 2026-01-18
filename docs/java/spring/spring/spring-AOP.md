Aspect-oriented Programming (AOP) 是面向对象Object-oriented Programming (OOP) 的补充，它提供了另一种思考程序结构的方式。在OOP中，模块化的关键是class，在AOP中模块化的单位是Aspect切面。切面可以跨越多种类型和对象（如事务管理）。

spring的关键组件是AOP。虽然IOC容器不依赖AOP（这意味着你如果不想使用AOP,可以去除），但AOP对IOC做了补充，提供了非常有力的中间解决方案。

spring提供了基于XML和@Aspect J的方式来自定义切面，仍然使用AOP进行织入。

spring AOP在spring框架中用来：

- 提供声明式的企业服务，最典型的莫过于声明式事务
- 让用户可以自定义切面，利用AOP完善OOP编程

# 5.1 AOP概念

我们先来定义AOP的一些概念和术语。这些术语不是spring特有的。不幸的是，AOP术语不是特别直观，然而如果spring使用自己的术语会更加令人困惑。

> - Aspect: A modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented by using regular classes (the [schema-based approach](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-schema)) or regular classes annotated with the `@Aspect` annotation (the [@AspectJ style](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-ataspectj)).
> - Join point: A point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.
> - Advice: Action taken by an aspect at a particular join point. Different types of advice include “around”, “before” and “after” advice. (Advice types are discussed later.) Many AOP frameworks, including Spring, model an advice as an interceptor and maintain a chain of interceptors around the join point.
> - Pointcut: A predicate that matches join points. Advice is associated with a pointcut expression and runs at any join point matched by the pointcut (for example, the execution of a method with a certain name). The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default.
> - Introduction: Declaring additional methods or fields on behalf of a type. Spring AOP lets you introduce new interfaces (and a corresponding implementation) to any advised object. For example, you could use an introduction to make a bean implement an `IsModified` interface, to simplify caching. (An introduction is known as an inter-type declaration in the AspectJ community.)
> - Target object: An object being advised by one or more aspects. Also referred to as the “advised object”. Since Spring AOP is implemented by using runtime proxies, this object is always a proxied object.
> - AOP proxy: An object created by the AOP framework in order to implement the aspect contracts (advise method executions and so on). In the Spring Framework, an AOP proxy is a JDK dynamic proxy or a CGLIB proxy.
> - Weaving: linking aspects with other application types or objects to create an advised object. This can be done at compile time (using the AspectJ compiler, for example), load time, or at runtime. Spring AOP, like other pure Java AOP frameworks, performs weaving at runtime.

- Aspect : 横跨多个类的切面。事务管理是最好的例子。在spring AOP中，aspect可以通过xml或者@Aspect的形式指定一些特定的类。
- Join point：程序执行的一个点，例如一个方法的执行或者一个异常的捕获。在spring AOP中，join point一般表示一个方法的执行。
- Advice: 在特定连接点执行的动作。有多种不同的建议，如around，before，after(稍后讨论advice)。许多AOP框架，包括spring，将建议建模为一个拦截器，并围绕joint point保留一个拦截器链。
- Pointcut: 表示匹配所有Join point的断言。advice总是关联pointcut表达式，并且在所有匹配到的joint point处执行（例如执行名字方法的执行）。被pointcut匹配到的joint points是AOP的核心概念，spring默认使用Aspect J的pointcut语法表达式
- Introduction: 在类型生命一个附加的方法或者字段。spring AOP允许你向任何被代理的对象引入一个新的接口（和对应的实现）。例如，你可以用Introduction实现一个实现IsModifued接口的bean，来简化缓存。（在Aspect J社区，Introduction被称为inter-type声明）
- Target object: 被一个或多个切面代理的对象。也被称为advised object。**因为spring AOP是通过运行时代理来实现的，所以这个对象总是一个被代理的对象。**
- AOP proxy：被AOP框架生成的对象，在spring中，AOP proxy是一个JDK动态代理或者CGLIB代理
- Weaving织入: 连接aspects和其他应用或者被创建的对象。这个可以在编译时（例如Aspect J编译器），加载时，或者运行时。spring AOP是在运行时进行织入。

Spring AOP 包含以下类型的Advice:

- Before advice: Advice that runs before a join point but that does not have the ability to prevent execution flow proceeding to the join point (unless it throws an exception).
- After returning advice: Advice to be run after a join point completes normally (for example, if a method returns without throwing an exception).
- After throwing advice: Advice to be run if a method exits by throwing an exception.
- After (finally) advice: Advice to be run regardless of the means by which a join point exits (normal or exceptional return).
- Around advice: Advice that surrounds a join point such as a method invocation. This is the most powerful kind of advice. Around advice can perform custom behavior before and after the method invocation. It is also responsible for choosing whether to proceed to the join point or to shortcut the advised method execution by returning its own return value or throwing an exception.

around advice是最通用的一种建议。由于spring AOP和Aspect J一样，提供了完整的advice类型，我们建议尽量使用最小权限的advice类型。例如，如果你只需要通过方法的返回值来更新缓存，你最好使用after returning advice而不是around advice，尽管around advice也可以完成同样的事情。使用特别指定的advice可以提供一个更简单的编程模型，出错的机会更小。例如，你不需要在around advice中调用`proceed()`方法，因此你不能不调用它。

# 5.2 spring AOP的能力和目标

spring AOP是纯java实现的，不需要特别的编译过程。spring AOP不需要控制类的加载，因此特别适用于servlet容器或应用服务。

spring AOP目前只支持方法级别（建议在spring bean中的方法）。字段的拦截没有实现，尽管可以实现。如果你需要字段的advice和更新joint points，可以考虑使用AspectJ。

spring实现AOP的方式不同于其他大多数AOP框架。其目的不是提供最完整的AOP实现（尽管spring AOP挺强的）,而是为了提供AOP和IOC容器之间的紧密结合，以解决企业开发中的常见问题。



因此，spring AOP功能通常和IOC容器结合使用。通过使用普通的bean定义语法(尽管允许强大的自动代理)来配置AspectJ，这是与其他AOP框架的关键区别。**你不能使用spring AOP轻松或高效的针对非常细粒度的对象（如常见的DTO对象）实现advice**。在这种情况下ASpectJ是最好的选择。然而，我们的经验是，spring AOP针对企业开发的大多数问题都提供了一个很好的解决方案。

spring AOP从来没有努力和AspectJ竞争，以提供一个全面的AOP解决方案。我们相信spring AOP的基于代理的框架和完整的AOP框架（如AspectJ)都是有价值的。他们是互补的，而不是竞争的。spring将AOP和IOC容器，AspectJ进行了无缝集成，可以在spring应用中实现AOP的所有功能，这种集成不会影响spring AOP或者其他AOP框架。spring AOP仍然向后兼容。

**静态AOP和动态AOP**

静态AOP通过特定的编译器将Aspect编译织入到系统类中，也就是在编译时期，就以字节码的形式织入到目标java类中，织入后的类就是普通Java类，虚拟机可以像通常一样加载运行，相比较于下面的动态代理不会有任何的性能损失。缺点就是不够灵活，如果要改变join point，需要重新代码编译。

动态AOP不是预先编译到目标类中，织入过程在系统运行之后，也就是在类加载或者运行时进行织入。这样可以比较方便更改切入点和织入逻辑，缺点就是存在性能损失，但随着反射和字节码操作技术的发展，这种性能损失完全可以接受。

**Spring AOP和AspectJ的区别**？

AspectJ是非常强大的AOP框架，隶属于Eclipse基金会，官网地址是`http://www.eclipse.org/aspectj/`。可以单独使用，也可以集成到其他框架中。它支持编译时织如，编译后织如（但是需要使用ajc编辑器），加载时织如。注意它是织入，它是将切面代码嵌入到被代理的业务类，这是和spring重要的区别。

AspectJ支持字段，方法，构造函数等等的织入，而spring仅支持方法的织入。

常用的这些注解就是aspectJ中的:

![image-20201116175642839](images/aspectJ.png)

spring AOP也是一个AOP框架，它是基于动态代理的，有基于JDK和基于cgLib两种方式，它需要依赖spring容器来管理，通过代理类来作用。基本满足开发中80%的需要，但性能不如Aspect J。

spring也集成了Aspect J，如上图中的用法。但是注意spring还是通过代理类的方式，所以也造成同类中调用AOP失效的问题。

[spring AOP和AspectJ的区别](https://segmentfault.com/a/1190000022019122)

[AspectJ的方法](https://blog.mythsman.com/post/5d301cf2976abc05b34546be/)

# 5.3 AOP 代理

spring AOP默认使用JDK动态代理。这用于接口的代理

spring AOP也可以使用CGLIB代理，这用于没有接口的代理类。默认的，如果一个业务对象没有实现接口就使用CGLIB代理。如果一个类实现了多个接口，可以强制使用CGLIB。在这种情况（希望是罕见的）。

**掌握spring AOP是基于代理的非常重要**。更多细节参考 [Understanding AOP Proxies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-understanding-aop-proxies)。

# 5.4 @AspectJ support

@AspectJ指的是通过注解来声明切面的java风格。@AspectJ是由AspectJ项目引入的，作为AspectJ 5的一部分，spring继承了相同的注解，使用AspectJ的依赖支持进行pointcut的解析和匹配。**但是AOP仍然是纯的spring AOP，对AspectJ的 compiler or weaver没有任何依赖。**

使用AspectJ的compiler or weaver可以使用其完整的功能，查看5.10。

## 5.4.1. 开启@AspectJ支持

要在spring中使用@AspectJ，你需要开启spring的支持来配置基于@AspectJ和自动代理的spring AOP。所谓自动代理，指的是如果spring确定一个bean被一个或多个aspects切面，它会自动为该bean生成一个代理类来拦截方法调用，确保在运行时被advice代理。

可以通过xml或者java配置的方式来启用@AspectJ的支持。不管哪种，你都需要确保AspectJ的aspectjweaver.jar依赖在你的classpath中。

如何开启@AspectJ支持。（spring -boot-starter-aop集成了这个）

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

## 5.4.2 声明切面

上面开启@Aspectj支持后，你应用上下文中任何被@Aspect注解的bean都会被spring探测到，用来配置spring AOP。如下代码所示：

```
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {

}
```

被@Aspect标注的类，和其他类一样可以拥有方法和字段。它也可以包含pointcut，advice和introduction(inter-type)的声明。

> Autodetecting aspects through component scanning
>
> You can register aspect classes as regular beans in your Spring XML configuration or autodetect them through classpath scanning — the same as any other Spring-managed bean. However, note that the `@Aspect` annotation is not sufficient for autodetection in the classpath. For that purpose, you need to add a separate `@Component` annotation (or, alternatively, a custom stereotype annotation that qualifies, as per the rules of Spring’s component scanner).

注意这个bean只被@Aspect注解还不足以被spring探测到，还需要添加@Component注解



在spring AOP中，aspects切面不能再被其他切面代理。标注了@Aspect的类不会被自动代理。

## 5.4.3. 声明Pointcut

pointcuts决定哪些是关心的join points，因此允许我们控制何时advice执行。spring AOP只支持bean的方法级别的join points，所以你可以看作pointcut是用来匹配spring beans的方法。一个pointcut声明有两部分：一个包含name和任何参数的方法签名，和用来精确决定哪些方法执行要被代理的pointcut 表达式。在AOP的@AspectJ注解风格，pointcut签名是由一个普通方法提供（这个方法必须返回必须是void），而pointcut表达式使用@pointcut注解。

下面用一个例子来说明point  signature 和point signature 的区别：

```java
@Pointcut("execution(* transfer(..))") // the pointcut expression
private void anyOldTransfer() {} // the pointcut signature
```

上面@Pointcut注解中的pointcut表达式属于@AspectJ 5的语法，更具体的可以看 [AspectJ Programming Guide](https://www.eclipse.org/aspectj/doc/released/progguide/index.html)， [AspectJ 5 Developer’s Notebook](https://www.eclipse.org/aspectj/doc/released/adk15notebook/index.html)。

### Supported Pointcut Designators

> Spring AOP supports the following AspectJ pointcut designators (PCD) for use in pointcut expressions:
>
> - `execution`: For matching method execution join points. This is the primary pointcut designator to use when working with Spring AOP.
> - `within`: Limits matching to join points within certain types (the execution of a method declared within a matching type when using Spring AOP).
> - `this`: Limits matching to join points (the execution of methods when using Spring AOP) where the bean reference (Spring AOP proxy) is an instance of the given type.
> - `target`: Limits matching to join points (the execution of methods when using Spring AOP) where the target object (application object being proxied) is an instance of the given type.
> - `args`: Limits matching to join points (the execution of methods when using Spring AOP) where the arguments are instances of the given types.
> - `@target`: Limits matching to join points (the execution of methods when using Spring AOP) where the class of the executing object has an annotation of the given type.
> - `@args`: Limits matching to join points (the execution of methods when using Spring AOP) where the runtime type of the actual arguments passed have annotations of the given types.
> - `@within`: Limits matching to join points within types that have the given annotation (the execution of methods declared in types with the given annotation when using Spring AOP).
> - `@annotation`: Limits matching to join points where the subject of the join point (the method being run in Spring AOP) has the given annotation.

spring AOP的pointcut表达式支持一下pointcut语法：

- `execution`用来匹配指定方法签名的方法的joint point。因为spring AOP只支持方法级别的aspect，这个是最常用的。

- `within` 匹配具有特定类型的join points。也就是指定类声明的所有方法。@Pointcut("within(common.job.BaseJob+)

- `this` AOP代理类是给定类型的实例

- target 被代理的对象是给定类型的实例

- args：方法参数是给定类型的实例

- `@target`被代理的对象必须有给定类型的注解

- `@args` 运行时参数必须有给定的注解

- `@within`有指定注解的类中的方法

- `@annotation` 指定的运行中的方法必须有给定的注解，比如@Transaction就应该使用的这个

  注意：@开头的表达式都只接受注解类型的参数。

> Other pointcut types
>
> The full AspectJ pointcut language supports additional pointcut designators that are not supported in Spring: `call`, `get`, `set`, `preinitialization`, `staticinitialization`, `initialization`, `handler`, `adviceexecution`, `withincode`, `cflow`, `cflowbelow`, `if`, `@this`, and `@withincode`. Use of these pointcut designators in pointcut expressions interpreted by Spring AOP results in an `IllegalArgumentException` being thrown.
>
> The set of pointcut designators supported by Spring AOP may be extended in future releases to support more of the AspectJ pointcut designators.

AspectJ有一些其他类型的pointcuts语法，spring AOP并不支持，会抛出`IllegalArgumentException` 异常。将来可能会增加支持的类型。



由于spring AOP支持方法级别的join points，前面关于pointcut的讨论都比AspectJ编程指南中的定义要狭小。此外，AspectJ自身是基于type-based，所以在join point 的执行时，this和target都指的是同一个对象。spring AOP是基于代理的，区分了代理对象（this），被代理的对象（target）。

由于spring AOP是**基于代理的特性，目标对象内的调用是不会被拦截**的。对于JDK动态代理，只有public方法才会被拦截，对于CGLIB，public和protect方法会被拦截（甚至package-visible）。然而想要被拦截代理的方法最好都设计成public的。

如果你的拦截需求包含目标类的方法调用甚至构造函数，请考虑使用原生的AspectJ，而不是基于代理的spring AOP。



spring AOP也支持PCD named bean。允许你定义point时,使用bean的name或者通配符。

`idOrNameOfBean` 是bean的name，通配符限制支持*，所以如果你的bean命名是有规律的，你可用PCD表达式来挑选他们。也可以使用 && || ！等。

bean PCD只在spring AOP中支持，AspectJ中并没有。他是spring AOP特有的扩展，所以你不能在@Aspect模型中使用。

PCD是基于实例而不是仅仅基于类的。基于实例的pointcut设计是基于代理的spring AOP特有的，它和spring 工厂结合是自然而直观的。

### 组合Pointcut表达式

你可以使用&& || ！来组合pointcut表达式，也可以通过方法name来引用pointcut。来看几个例子：

```java
@Pointcut("execution(public * *(..))") // 表示任何public修饰的方法 
private void anyPublicOperation() {} 

@Pointcut("within(com.xyz.myapp.trading..*)") //表示trading包下的方法
private void inTrading() {} 

@Pointcut("anyPublicOperation() && inTrading()") // 表示trading包下 public修饰的方法
private void tradingOperation() {}
```

如上，通过一些较小的命名组件来构建复杂的pointcuts是一种比较好的做法。当通过name来引用pointcuts时，方法的可见性是没有关系的，public，protected，private等都不会影响pointcut的匹配。

### 共享公共的pointcut定义

Sharing Common Pointcut Definitions

在企业开发中，开发者经常希望引用应用的模块和特定的操作中的切面。我们建议定义一个公共的pointcuts，来保存常见的pointcuts。举例如下：

```java
package com.xyz.myapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class CommonPointcuts {

    /**
     * A join point is in the web layer if the method is defined
     * in a type in the com.xyz.myapp.web package or any sub-package
     * under that.
     * web包及子包下的所有方法
     */
    @Pointcut("within(com.xyz.myapp.web..*)")
    public void inWebLayer() {}

    /**
     * A join point is in the service layer if the method is defined
     * in a type in the com.xyz.myapp.service package or any sub-package
     * under that.
     * service包及子包下的所有方法
     */
    @Pointcut("within(com.xyz.myapp.service..*)")
    public void inServiceLayer() {}

    /**
     * A join point is in the data access layer if the method is defined
     * in a type in the com.xyz.myapp.dao package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.myapp.dao..*)")
    public void inDataAccessLayer() {}

    /**
     * A business service is the execution of any method defined on a service
     * interface. This definition assumes that interfaces are placed in the
     * "service" package, and that implementation types are in sub-packages.
     *
     * If you group service interfaces by functional area (for example,
     * in packages com.xyz.myapp.abc.service and com.xyz.myapp.def.service) then
     * the pointcut expression "execution(* com.xyz.myapp..service.*.*(..))"
     * could be used instead.
     *
     * Alternatively, you can write the expression using the 'bean'
     * PCD, like so "bean(*Service)". (This assumes that you have
     * named your Spring service beans in a consistent fashion.)
     * 不同路径下的service包下的方法
     */
    @Pointcut("execution(* com.xyz.myapp..service.*.*(..))")
    public void businessService() {}

    /**
     * A data access operation is the execution of any method defined on a
     * dao interface. This definition assumes that interfaces are placed in the
     * "dao" package, and that implementation types are in sub-packages.
     * 定义在dao接口的任意方法，注意和within的区别
     */
    @Pointcut("execution(* com.xyz.myapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```

你可以在其他地方使用方法的全路径来引用这个pointcuts，

```java
pointcut="com.xyz.myapp.CommonPointcuts.businessService()"
```

### 举例

很多人都喜欢用execution语法，它的表达式语法是：

```java
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)
                throws-pattern?)
```

除了ret-type-pattern，name-pattern和param-pattern，其他部分都是可选的。returning type pattern决定哪些返回类型的方法被匹配到，\*是最常用的匹配符，它匹配任何类型的返回值，完全限定的类型名只有在方法返回给定类型时才会匹配。name pattern对应的是方法的名称，也可以使用\*来作为匹配的全部或部分，如果你指定了一个声明的模式，需要包含\.来连接。

parameters pattern稍微复杂一些，

() 匹配没有参数的方法，

(..)匹配任何数量（0个或多个）的参数,

(\*)匹配拥有一个任意类型的参数方法

*(*\*,String)匹配有两个参数的方法，第一个可以是任意类型，第二个必须是string类型。

下面举一些例子：

```java
 execution(public * *(..)) //任意public修饰的方法
     
 execution(* set*(..))  // 任意set开头的方法
     
 execution(* com.xyz.service.AccountService.*(..)) // 任意定义在AccountService接口中的方法
     
 execution(* com.xyz.service.*.*(..)) // 任意service包下的方法
     
 execution(* com.xyz.service..*.*(..)) // 任意service包及其子包下的方法
     
 within(com.xyz.service.*) // Any join point (method execution only in Spring AOP) within the service package:
     
 within(com.xyz.service..*) // within the service package or one of its sub-packages:
     
 this(com.xyz.service.AccountService) // Any join point (method execution only in Spring AOP) where the proxy implements the AccountService interface:

  target(com.xyz.service.AccountService) //  target object implements the AccountService interface
  
 args(java.io.Serializable) // Any join point (method execution only in Spring AOP) that takes a single parameter and where the argument passed at runtime is Serializable:
 注意它和execution(* *(java.io.Serializable))的不同，后者是指方法签名中的参数 ，前者是运行时
     
 @target(org.springframework.transaction.annotation.Transactional) // Any join point (method execution only in Spring AOP) where the target object has a @Transactional annotation
     
 @args(com.xyz.security.Classified) //  which takes a single parameter, and where the runtime type of the argument passed has the @Classified annotation:
     
  bean(tradeService) // 名字为tradeService的bean
     
  bean(*Service) // service开头的bean
```

### Writing Good Pointcuts

在编译过程中，AspectJ会处理点切，以优化匹配性能。检查代码并确定每个连接点是否与给定的点切匹配（静态或动态）是一个代价高昂的过程。(动态匹配意味着不能从静态分析中完全确定匹配，当代码运行时，要在代码中放置一个测试来确定是否有实际匹配)。在第一次遇到一个pointcut声明时，AspectJ会把它改写成一个最佳的形式来进行匹配过程。这意味着什么呢？基本上，点切是以DNF（Disjunctive Normal Form）重写的，并且点切的组件会被排序，这样那些评估成本较低的组件会被首先检查。这意味着您不必担心了解各种点切代号的性能，可以在点切声明中以任何顺序提供它们。

然而，AspectJ只能工作在被告知的情况下，为了优化匹配的性能，你应该思考要实现的目的，并在定义中尽可能的缩小匹配的搜索的范围。上述的匹配语法可以分为三类：

一个写得好的pointcut应该至少包括前两种类型（kinded和scoping）。你可以包括基于Contextual 匹配，或者将该上下文绑定到advice中使用。只提供一个kinded代号或只提供一个 contextual 代号是可行的，但由于额外的处理和分析，可能会影响织造性能（所用时间和内存）。Scoping代号的匹配速度非常快，使用它们的用法意味着AspectJ可以非常快速地剔除不应该进一步处理的连接点组。如果可能的话，一个好的点切应该总是包含一个。

## 5.4.4 声明 Advice

advice和pointcut表达式关联，在方法before，after，around执行。pointcut表达式可以是另一个pointcut的名字引用，也可以是pointcut表达式。

### Before Advice

使用@Before注解

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.CommonPointcuts.dataAccessOperation()") // 引用公共包里的pointcut方法
    public void doAccessCheck() {
        // ...
    }
}


import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    // 直接使用pointcut表达式
    @Before("execution(* com.xyz.myapp.dao.*.*(..))")
    public void doAccessCheck() {
        // ...
    }

}
```

### After Returning Advice

用于方法执行正常返回时，使用@AfterReturning注解。

你可以在同一个类里声明多个advice。

有时，你想在方法里获取实际的返回值，可以使用以下的方式：

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning(
        pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }

}
```



注解里的returning属性必须和方法参数严格对应。当一个方法执行返回时，返回值会经过这个advice方法。returning属性还会限制只有方法返回对应返回值的匹配（这里Object会匹配所有返回类型值）

请注意，在执行advice后，不可能返回一个完全不同的引用。

### After Throwing Advice

通常你希望只有抛出指定异常时才执行advice，或者你想要在方法里获取异常信息。你可以使用`throwing` 属性来严格匹配，和对应方法参数。举例如下：

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing(
        pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
        throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }

}
```

throwing属性的名字必须和方法参数名严格对应。当一个方法执行抛出异常时，异常信息会经过这个advice，并传给方法参数，并且只有抛出的异常是参数声明的异常时才会调用advice（这里是DataAccessException异常）

### After (Finally) Advice

After (finally) advice在方法执行推出时调用，使用@After注解，对应方法正常结束或抛出异常的情况。典型的用法可以用来释放资源等。

### Around Advice

最后一种advice类型是around。Around advice可以在方法执行前后调用，决定何时，怎么甚至方法实际是否执行。Around advice 经常用来如果你想在一个线程安全管理时，在方法执行前后共享一个状态（如开启和结束一个计时器，统计方法执行时长）。注意如果其他advice能满足需求时，请不要使用around advice。

方法里使用ProceedingJoinPoint 的proceed方法调用目标类的方法。可以拿到返回值。

around advice方法的返回值，就是调用者调用方法看到的返回值。例如，如果一个缓存功能的切面，如果缓存有就返回缓存值，没有就调用proceed方法。注意，proceed方法可以被调用一次，许多次，或者零次，这些都是合法的。

### Advice Parameters

spring提供了完全类型化的advice，这意味着你可以在方法签名中声明你需要的参数（就和之前在eturning 和throwing的advice中看到的一样），而不是一直使用Object[]工作。我们将在本节后面看到如何使参数和其他上下文值可用于建议体。首先，我们来看看如何编写通用的建议，可以找出该建议当前所建议的方法的情况。

#### **Access to the Current** `JoinPoint`

任何advice方法都可以把JoinPoint作为方法签名的第一个参数（注意around advice的ProceedingJoinPoint属于JoinPoint的子类），JoinPoint提供了一下常见的API方法：

- `getArgs()`: Returns the method arguments.
- `getThis()`: Returns the proxy object.
- `getTarget()`: Returns the target object.
- `getSignature()`: Returns a description of the method that is being advised.
- `toString()`: Prints a useful description of the method being advised.

#### 获取advice的参数**Passing Parameters to Advice**

> We have already seen how to bind the returned value or exception value (using after returning and after throwing advice). To make argument values available to the advice body, you can use the binding form of `args`. If you use a parameter name in place of a type name in an args expression, the value of the corresponding argument is passed as the parameter value when the advice is invoked. An example should make this clearer. Suppose you want to advise the execution of DAO operations that take an `Account` object as the first parameter, and you need access to the account in the advice body. You could write the following:

我们前面已经知道如何绑定返回值和异常信息（在使用after returning和after throwing的 advice）。想要在方法里使用这些参数，你可以使用args表单。如果在args表达式使用方法参数名，那么当执行advice时，相应参数的值会传递给方法参数。下面举个例子会说的更清楚些，假如你想拦截dao层方法第一个参数是Account对象的方法，并在方法体里获取这个Account对象，可以这样写：

```java
@Before("com.xyz.myapp.CommonPointcuts.dataAccessOperation() && args(account,..)")
public void validateAccount(Account account) {
    // ...
}
```

>  The `args(account,..)` part of the pointcut expression serves two purposes. First, it restricts matching to only those method executions where the method takes at least one parameter, and the argument passed to that parameter is an instance of `Account`. Second, it makes the actual `Account` object available to the advice through the `account` parameter.

pointcut表达式里的`args(account,..)`有两个目的。第一，它严格限制了只有方法有一个参数是Account实例时才会匹配，第二，可以在advice方法体里通过account参数获取Account对象。

还有另外一种实现方式，通过引用另一个pointcut方法名，举例：

```java
@Pointcut("com.xyz.myapp.CommonPointcuts.dataAccessOperation() && args(account,..)")
private void accountDataAccessOperation(Account account) {}

@Before("accountDataAccessOperation(account)")
public void validateAccount(Account account) {
    // ...
}
```

> The proxy object ( `this`), target object ( `target`), and annotations ( `@within`, `@target`, `@annotation`, and `@args`) can all be bound in a similar fashion. The next two examples show how to match the execution of methods annotated with an `@Auditable` annotation and extract the audit code:

代理对象（this），目标对象（target）,和注解（@within`, `@target`, `@annotation`, and `@args）都可以通过相同的风格绑定。下面两个例子展示了如何获取方法上的注解中的信息

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditable {
    AuditCode value();
}

@Before("com.xyz.lib.Pointcuts.anyPublicMethod() && @annotation(auditable)")
public void audit(Auditable auditable) {
    AuditCode code = auditable.value();
    // ...
}
```

#### **Advice Parameters and Generics泛型**

spring AOP可以处理类和方法中的泛型，

```java
public interface Sample<T> {
    void sampleGenericMethod(T param);
    void sampleGenericCollectionMethod(Collection<T> param);
}
```

You can restrict interception of method types to certain parameter types by typing the advice parameter to the parameter type for which you want to intercept the method:

```java
@Before("execution(* ..Sample+.sampleGenericMethod(*)) && args(param)")
public void beforeSampleMethod(MyType param) {
    // Advice implementation
}

// 但是对集合泛型是无效的，下面这个是不行的
@Before("execution(* ..Sample+.sampleGenericCollectionMethod(*)) && args(param)")
public void beforeSampleMethod(Collection<MyType> param) {
    // Advice implementation
}
```

> To make this work, we would have to inspect every element of the collection, which is not reasonable, as we also cannot decide how to treat `null` values in general. To achieve something similar to this, you have to type the parameter to `Collection<?>` and manually check the type of the elements.

为了实现集合泛型，spring必须检查集合中的每个元素，这是不合理的，而且也不知道如何处理null的情况。如果要实现这个目的，你可以使用Collection<?>，然后手动检查每个元素。

#### **Determining Argument Names**

> The parameter binding in advice invocations relies on matching names used in pointcut expressions to declared parameter names in advice and pointcut method signatures. Parameter names are not available through Java reflection, so Spring AOP uses the following strategy to determine parameter names:

advice中调用的参数绑定依赖于pointcut表达式中使用的参数名称和 pointcut方法签名和advice里的参数名字匹配。参数名字不能通过Java反射获取，所以spring AOP提供了一下下策略来决定参数名字：

- If the parameter names have been explicitly specified by the user, the specified parameter names are used. Both the advice and the pointcut annotations have an optional `argNames` attribute that you can use to specify the argument names of the annotated method. These argument names are available at runtime. The following example shows how to use the `argNames` attribute:

  ```java
  @Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
          argNames="bean,auditable")
  public void audit(Object bean, Auditable auditable) {
      AuditCode code = auditable.value();
      // ... use code and bean
  }
  ```

  

  If the first parameter is of the `JoinPoint`, `ProceedingJoinPoint`, or `JoinPoint.StaticPart` type, you can leave out the name of the parameter from the value of the `argNames` attribute. For example, if you modify the preceding advice to receive the join point object, the `argNames` attribute need not include it:

  ```java
  @Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
          argNames="bean,auditable")
  public void audit(JoinPoint jp, Object bean, Auditable auditable) {
      AuditCode code = auditable.value();
      // ... use code, bean, and jp
  }
  ```

  The special treatment given to the first parameter of the `JoinPoint`, `ProceedingJoinPoint`, and `JoinPoint.StaticPart` types is particularly convenient for advice instances that do not collect any other join point context. In such situations, you may omit the `argNames` attribute. For example, the following advice need not declare the `argNames` attribute:

  ```java
  @Before("com.xyz.lib.Pointcuts.anyPublicMethod()")
  public void audit(JoinPoint jp) {
      // ... use jp
  }
  ```

- Using the `'argNames'` attribute is a little clumsy, so if the `'argNames'` attribute has not been specified, Spring AOP looks at the debug information for the class and tries to determine the parameter names from the local variable table. This information is present as long as the classes have been compiled with debug information ( `'-g:vars'` at a minimum). The consequences of compiling with this flag on are: (1) your code is slightly easier to understand (reverse engineer), (2) the class file sizes are very slightly bigger (typically inconsequential), (3) the optimization to remove unused local variables is not applied by your compiler. In other words, you should encounter no difficulties by building with this flag on.

  If an @AspectJ aspect has been compiled by the AspectJ compiler (ajc) even without the debug information, you need not add the `argNames` attribute, as the compiler retain the needed information.

- If the code has been compiled without the necessary debug information, Spring AOP tries to deduce the pairing of binding variables to parameters (for example, if only one variable is bound in the pointcut expression, and the advice method takes only one parameter, the pairing is obvious). If the binding of variables is ambiguous given the available information, an `AmbiguousBindingException` is thrown.

- If all of the above strategies fail, an `IllegalArgumentException` is thrown.

#### **Proceeding with Arguments**

> We remarked earlier that we would describe how to write a `proceed` call with arguments that works consistently across Spring AOP and AspectJ. The solution is to ensure that the advice signature binds each of the method parameters in order. The following example shows how to do so:
>
> In many cases, you do this binding anyway (as in the preceding example).

解决方法是确保advice的签名按顺序绑定每个方法参数

```java
@Around("execution(List<Account> find*(..)) && " +
        "com.xyz.myapp.CommonPointcuts.inDataAccessLayer() && " +
        "args(accountHolderNamePattern)")
public Object preProcessQueryPattern(ProceedingJoinPoint pjp,
        String accountHolderNamePattern) throws Throwable {
    String newPattern = preProcess(accountHolderNamePattern);
    return pjp.proceed(new Object[] {newPattern});
}
```

### advice的顺序

Advice Ordering

当多个advice都想在同一个join point运行时，会发生什么？spring AOP遵循AspectJ的优先级来决定advice的执行顺序。优先级高的advice在进入方法时先运行（也就是给定两个before advice，优先级高的先执行）。在join point的On the way out，高优先级的advice后运行，（所以给定两个after advice，高优先级的后运行）。

当两个AspectJ的advice都需要运行在相同的join point时，除非你特别指定，否则执行顺序是未定义的。你可以指定优先级来控制执行顺序，可以通过@Order注解来执行。order值较低的aspect拥有较高的优先级。

# 5.5 Schema-based AOP Support

基于XML声明AOP，由于不常用，这里略去。

# 5.6 选择如何使用AOP？

一旦你决定好使用切面来实现你的需求，接着就是决定使用Spring AOP还是AspectJ，使用@Aspect注解方式还是xml方式。这取决于应用需求，开发工具和团队对AOP的熟悉程度。

## 5.6.1 Spring AOP还是Full AspectJ

> Use the simplest thing that can work. Spring AOP is simpler than using full AspectJ, as there is no requirement to introduce the AspectJ compiler / weaver into your development and build processes. If you only need to advise the execution of operations on Spring beans, Spring AOP is the right choice. If you need to advise objects not managed by the Spring container (such as domain objects, typically), you need to use AspectJ. You also need to use AspectJ if you wish to advise join points other than simple method executions (for example, field get or set join points and so on).
>
> When you use AspectJ, you have the choice of the AspectJ language syntax (also known as the “code style”) or the @AspectJ annotation style. Clearly, if you do not use Java 5+, the choice has been made for you: Use the code style. If aspects play a large role in your design, and you are able to use the [AspectJ Development Tools (AJDT)](https://www.eclipse.org/ajdt/) plugin for Eclipse, the AspectJ language syntax is the preferred option. It is cleaner and simpler because the language was purposefully designed for writing aspects. If you do not use Eclipse or have only a few aspects that do not play a major role in your application, you may want to consider using the @AspectJ style, sticking with regular Java compilation in your IDE, and adding an aspect weaving phase to your build script.

尽量使用简单的方式。Spring AOP比使用Full AspectJ更简单，因为不需要将AspectJ编译/weaver引入你的开发和构建过程。如果你只需要针对Spring的bean进行advice，Spring AOP是正确的选择。如果你想要advice的对象没有被Spring容器管理（如domain对象，DTO等），你需要使用aspectJ。如果你想基于字段或者设置join point等进行advice，也需要使用aspectJ。

当你使用AspectJ，你可以选择AspectJ语法（编程方式），或者注解风格。如果aspect在你的设计中起着很大的作用，并且你能够使用Eclipse的[AspectJ开发工具(AJDT)](https://www.eclipse.org/ajdt/)插件，AspectJ语言语法是首选。它更干净、更简单，因为该语言是专门为编写aspect而设计的。如果你不使用Eclipse，或者只有少数几个在你的应用程序中不起主要作用的方面，你可能要考虑使用@AspectJ风格，在你的IDE中坚持使用常规的Java编译，并在你的构建脚本中添加一个方面编织阶段。

## 5.6.2 @Aspect注解还是XML

肯定注解更方便了，支持更丰富点，可以组合pointcut。这里也略去。

# 5.7. Mixing Aspect Types

在同一个配置中，混合使用自动代理支持、xml定义的@AspectJ风格的切面，其他风格的代理和拦截，这些都是可以的。所有这些都通过相同的底层支持，可以毫无困难的共存。

# 5.8 代理机制

> Spring AOP uses either JDK dynamic proxies or CGLIB to create the proxy for a given target object. JDK dynamic proxies are built into the JDK, whereas CGLIB is a common open-source class definition library (repackaged into `spring-core`).
>
> If the target object to be proxied implements at least one interface, a JDK dynamic proxy is used. All of the interfaces implemented by the target type are proxied. If the target object does not implement any interfaces, a CGLIB proxy is created.

Spring AOP不仅使用JDK动态代理，也使用CGLIB来为目标对象创造代理。JDK动态代理构建于JDK，CGLIB是一个开源的类定义包。

**如果目标对象实现了至少一个接口，就会使用JDK动态代理。目标类实现的所有接口都会被代理**。如果目标类没有实现任何接口，就会创建一个CGLIB代理。

如果你想强制使用CGLIB代理（例如，代理目标类的每个方法，而不仅是实现接口的方法），这可以做到。但是你需要考虑以下问题：

- **使用CGLIB，final方法不能被代理**，因为他们不能被运行时生成的子类重写。
- 从Spring4.0开始，被代理的对象不再被调用两次，因为CGLIB代理的实例是通过Objenesis生成的。只有当JVM不允许绕过构造函数时，你才会看到Spring AOP的两次调用和debug日志。

想要强制使用CGLIB代理，设置`<aop:config>`的`proxy-target-class`属性为true，如：

```xml
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
```

当使用@AspectJ自动代理时，可以设置`<aop:aspectj-autoproxy>`的属性`proxy-target-class`为true，如

```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

>  Multiple `<aop:config/>` sections are collapsed into a single unified auto-proxy creator at runtime, which applies the *strongest* proxy settings that any of the `<aop:config/>` sections (typically from different XML bean definition files) specified. This also applies to the `<tx:annotation-driven/>` and `<aop:aspectj-autoproxy/>` elements.
>
>  To be clear, using `proxy-target-class="true"` on `<tx:annotation-driven/>`, `<aop:aspectj-autoproxy/>`, or `<aop:config/>` elements forces the use of CGLIB proxies *for all three of them*.

## 5.8.1 理解AOP代理

就是自我调用不会生效，这里不再赘述。略去

# 5.9 代码创建@AspectJ代理

> In addition to declaring aspects in your configuration by using either `<aop:config>` or `<aop:aspectj-autoproxy>`, it is also possible to programmatically create proxies that advise target objects. For the full details of Spring’s AOP API, see the [next chapter](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-api). Here, we want to focus on the ability to automatically create proxies by using @AspectJ aspects.
>
> You can use the `org.springframework.aop.aspectj.annotation.AspectJProxyFactory` class to create a proxy for a target object that is advised by one or more @AspectJ aspects. The basic usage for this class is very simple, as the following example shows:

除了使用`<aop:config>` or `<aop:aspectj-autoproxy>`来声明切面，还可以通过代码来创建代理，advice目标类。关于spring API的更多细节，请查看下一节。这里，我们只关心使用@AspectJ自动创建代理类。

你可以使用`org.springframework.aop.aspectj.annotation.AspectJProxyFactory`为目标对象创建一个或多个代理，这个类的基本使用非常简单，如下：

```java
// create a factory that can generate a proxy for the given target object
AspectJProxyFactory factory = new AspectJProxyFactory(targetObject);

// add an aspect, the class must be an @AspectJ aspect
// you can call this as many times as you need with different aspects
factory.addAspect(SecurityManager.class);

// you can also add existing aspect instances, the type of the object supplied must be an @AspectJ aspect
factory.addAspect(usageTracker);

// now get the proxy object...
MyInterfaceType proxy = factory.getProxy();
```

更多细节查看 [javadoc](https://docs.spring.io/spring-framework/docs/5.3.1/javadoc-api/org/springframework/aop/aspectj/annotation/AspectJProxyFactory.html) 。

# 6. Spring AOP APIs

上一个章节介绍了spring对@AspectJ和基于xml的AOP支持。在本章节，我们讨论底层的spring  AOP的APIs，对于普通的应用来说，我们建议使用上一章节介绍的基于AspectJ的spring AOP。