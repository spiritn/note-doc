# Spring是如何解决循环依赖的？

- Spring可以解决构造函数的循环依赖吗？

  直接是报错的，但是使用@lazy可以解决。是通过cglib创建一个空对象用于依赖。

- 原型scoope的bean之间的循环依赖可以解决吗？

  不能！原型的bean是在需要的时候再创建，也就是Spring并没有进行管理，放进缓存，所以无法解决循环依赖问题！！

## 三级缓存

@Autowired方式注入属性时，通过三级缓存解决。

- 单例池

- 二级缓存：存放new出来的不完整对象，这样当单例池中A找不到依赖的属性B时，就可以先从二级缓存中获取不完整对象B，完成A对象创建。后续B可以直接从单例池中获取完整的对象A。

  如果引用的对象配置了AOP需要生成代理对象时，二级缓存就不够用了。

  二级缓存可以避免重复创建某对象的动态代理。如A和B相互依赖，A和C相互依赖，所以把A放进二级缓存，就可以避免C也去创建重复的A动态对象。

- 三级缓存：存放所有对象的代理配置信息，在发现有循环依赖时，将这个对象的动态配置信息取出来，提前生成动态代理proxy。存的是函数接口，并不是直接调用。

  ![三级缓存](images/三级缓存.jpg)

整个核心代码就在DefaultSingletonBeanRegistry：

```java
/** 一级缓存 Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** 二级缓存 Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);

/** 三级缓存 Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // Quick check for existing instance without full singleton lock
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
            synchronized (this.singletonObjects) {
                // Consistent creation of early reference within full singleton lock
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    singletonObject = this.earlySingletonObjects.get(beanName);
                    if (singletonObject == null) {
                        ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                        if (singletonFactory != null) {
                            singletonObject = singletonFactory.getObject();
                            // 添加到二级缓存，从三级缓存删除
                            this.earlySingletonObjects.put(beanName, singletonObject);
                            this.singletonFactories.remove(beanName);
                        }
                    }
                }
            }
        }
    }
    return singletonObject;
}
```

