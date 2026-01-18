# 加密
## DH秘钥交换算法

作用就是将对称加密的秘钥，通过非对称算法来进行交换。（非对称加密算法性能比较低）

1.调用端A，产生P，g,publicKey，privateKey，支持的AES加密算法列表，P，g,publicKey给B（通过Token来检验身份）。

2.被调用端通过P,g产生自己的publicKeyB，privateKeyB，然后结合publicKeyA，来生成一个秘钥（秘钥+向量两部分）。利用秘钥加密数据形成密文s(data).然后传输publicKeyB，密文，AES算法给A

3.调用端A通过publicKeyB结合自己的privateKeyA来生成秘钥（秘钥+向量两部分，同B中的秘钥一致），利用这个秘钥解密B传过来的密文，就可以拿到数据了。

注：在第二步还可以传一个ExpireTime过期时间，会话ID，这样拿这个会话ID在过期时间之前都可以进行安全的数据传递。

A直接调用B的数据，B直接在响应中返回密文。还有一种是A要设置敏感数据，就需要先和B建立连接拿到秘钥，再传递数据。这两种情况有所区别，要注意。B要主动推送数据的话，就需要提前建立会话ID，设置过期时间，在这期间可以进行主动推送数据。



## 数字摘要（Digital Digest）

数字摘要Message Digest用来保证数据的一致性和完整性

MD5一般用来做数字摘要。把原文和原文的MD5串一起发送给接收方，接收方对原文进行MD5，然后和接收到的MD5串进行比较，确认原文是否被篡改（MD5的特性，只要原文有一点点改变，MD5结果就会大不相同）。比较流行的还有sha-1，HAVAL



## 签名（Digital Signature）

签名用来用来确认发送方的身份，确实是某人发出的，不可抵赖。

RSA可以用来做数字签名。发送方将原文的摘要再用私钥加密，形成签名，接收方接受信息后将签名利用发送方的公钥解密，证实这个消息确实是发送方发出来的，并将解密的摘要进行比较，证明信息没有被篡改。

但是一般微信，支付宝接口都是用MD5加上秘钥进行数字签名。



## 对称加密和非对称加密

加密用来加密原文，解密密文得到原文，分为对称加密和非对称加密

对称加密的代表算法是AES算法。使用一个秘钥key，但是这个key的传输很难确保安全，这是他的唯一缺点。

非对称加密的算法是RSA算法，RSA算法可以用来加密，也可以用来做签名（流程有点相反）。但是RSA加密算法非常慢，性能比对称加密慢三个数量级，所以适用小数据量的传输，如可以用来传递对称加密算法的秘钥。DH秘钥交换算法也可以用来交换对称加密算法的秘钥。



## Base64编码

是用来在一些只适合文本传输协议的情况传输二进制流等不可见字符，即将不可见字符进行Base64编码转变为可见的字符（ASCII的64个字符），然后进行传输。并不是加密算法，可以很轻松的进行编码，解码。

encrypt加密 decrypt解密

## Web前后端敏感数据传输

平台在启动时，自动随机生成一对RSA公私钥，将私钥缓存于服务端，将公钥以接口形式公开。
前端首先请求服务端公钥获取接口并缓存，在之后涉及敏感数据提交前，使用公钥加密数据，并进行Base64编码后提交；后端获取加密后数据，使用Base64解码，再利用私钥解密获取明文数据


## 公司登录加密流程

Md5加密 SHA加密 挑战码 盐值 Cas认证

用户登录输入用户名密码，调用getSalt（携带用户名），返回盐值，挑战码，加密算法，然后前端加密SHA256（SHA256(password+salt）+challengeCode)，生成密文去后台验证。后台从数据库取出密码和盐值，结合挑战码生成密文，与前台生成的比对，相同则去cas单点登录获取TGT

登录时输错密码两次后要进行验证码校验，五次后要锁定ip。注意周剑提出要先对验证码进行校验（这不属于业务校验），然后再对ip校验（是否锁定，是否在限制区间），这可以防止暴力攻击，节省流量（不能高频输入验证码）。成功后记得清除登录失败信息。

公司的服务间认证的token生成和配置文件里的密码采用AES加密然后base64编码，调用的底层native的方法。共享秘钥应该是根据授权文件的东西生成传输。敏感数据传输，包括获取和发送。采用的DH秘钥交换技术先交换秘钥。

## 内部服务间调用

从物理网络上对外网是隔离的，内部的认证机制相对做到轻量，具体方式如下：

1. 为每一个内部服务分配唯一服务标识；
2. 所有内部服务都共享一个秘钥；
3. 服务调用方发起请求时，使用“被调用方应用标识” + “共享秘钥” + 当前时间（精确到小时），得到一个拼接字符串，再对该字符串进行MD5生成摘要，带入到请求头中，记为Token。
4. 服务提供方校验请求头中的Token，使用几个因子经过相同算法生成摘要，对Token进行比对，相同则认证通过；


# 要点

## maven打包

idea的maven插件可以打包

也可以使用命令行maven命令来打包，具体可以百度。

svn上有install.bat很好用，可以直接点击运行打包，非常好用。内容为

```
call mvn -Dmaven.test.skip=true clean package
```

会自动退出，在exit退出时，整个脚本进程会结束。

## tomcat下放两个war包

1.tomcat下有两个war包时，war包的名字就是上下文的路径，很方便。

2.好像也可以去serfver.xml配置，


## 出现问题的组件：


由于上述原因是导致tomca阻塞的原因之一，特提供排查方法和排查原理以备不时之需：

1、使用jstack [pid] > stack.txt 输出栈信息
2、检查是否存在死锁
3、检查startStop线程是否是睡眠状态或者阻塞状态
4、检查startStop线程在哪个方法内阻塞或者睡眠了
5、分析代码寻找原因

排查原理背景介绍：

tomcat部分知识：

tomcat启动时使用callable多线程去初始化每个servlet，多线程的名称是中包含startStop字符串，

servlet初始化成功后会打印类似于一下日志

13-Jan-2020 19:46:32.249 信息 [localhost-startStop-26] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/opt/hik/web/components/tomcat85linux64.1/webapps/fpfas] has finished in [56,183] ms

tomcat启动成功后会打印

16-Jan-2020 20:12:10.893 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in 175421 ms

当startStop线程阻塞，第二条日志不会打印。因为tomcat是通过callable线程的get方法判断servlet是否启动完成， 一旦startStop线程阻塞，会导致tomcat一直等待启动线程结束。


spring的bean加载知识：

@Scope即bean的作用域不同代理类的不同状态，以及代理类的动态加载。bean的初始化的流程。

处理意见：
按目前已发现的阻塞问题,提供如下处理意见。

1、将[@PostConstruct注解，改为实现ApplionRunner接口、或者接听项目启动事件@EventListener(classes ](https://www.yuque.com/PostConstruct注解，改为实现ApplionRunner接口、或者接听项目启动事件@EventListener(classes ) = ApplicationReadyEvent.class) 

2、不要在@Bean方法内使用远程调用


区别在于

修改前当单个bean加载成功后就执行被注解的代码，
修改后所有bean加载成功后才同样的代码，绕开多线程初始化spring的bean。

```properties
VM Flags:
-XX:CICompilerCount=2 
-XX:ConcGCThreads=2 
-XX:G1ConcRefinementThreads=2 
-XX:G1HeapRegionSize=1048576 
-XX:G1ReservePercent=10 
-XX:GCDrainStackTargetSize=64 
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=/opt/hik/web/components/tomcat85linux64.2/dump 
-XX:InitialHeapSize=1073741824 
-XX:InitiatingHeapOccupancyPercent=70 
-XX:MarkStackSize=4194304 
-XX:MaxDirectMemorySize=67108864 
-XX:MaxHeapSize=2684354560 
-XX:MaxNewSize=1610612736 
-XX:MinHeapDeltaBytes=1048576 
-XX:NonNMethodCodeHeapSize=5825164 
-XX:NonProfiledCodeHeapSize=122916538 
-XX:ParallelGCThreads=2 
-XX:ProfiledCodeHeapSize=122916538 
-XX:ReservedCodeCacheSize=251658240 
-XX:+SegmentedCodeCache 
-XX:ThreadStackSize=512 
-XX:+UseCompressedClassPointers 
-XX:+UseCompressedOops 
-XX:+UseG1GC
```

## rabbitMQ

参数配置：

```properties
spring.rabbitmq.host=localhost

spring.rabbitmq.port=5672

spring.rabbitmq.http.port=15672

spring.rabbitmq.username=iotadmin

spring.rabbitmq.password=passw0rd

spring.rabbitmq.virtual-host=iotdev

rabbitmq.simple.consumer=1

rabbitmq.simple.maxconsumer=3

rabbitmq.simple.prefetchcount=30

rabbitmq.simple.threadpool=3

rabbitmq.listener.consumer=1

rabbitmq.listener.maxconsumer=10

rabbitmq.listener.prefetchcount=100

rabbitmq.listener.threadpool=10
```

rabbitmq的发布者确认机制
confirm和returnback监听。[参考这里](http://baijiahao.baidu.com/s?id=1590386092002156339&wfr=spider&for=pc)
消息可以发送到Exchange时Ack为true，否则为false。
如果消息无法发送到指定的消息队列那么ReturnCallBack回调方法会被调用，setMandatory必须为true。不过这种增加了生产方的逻辑，一般设置备份队列

mq消费端一旦挂掉之后，队列中会大量堆积消息为被消费，消费端重启后还会去消费，但已经超时不应该再消费了。

添加一个时间戳，消费端进行判断，如果超时就不再消费，就可以解决这类问题。

rabbitmq使用线程共享技术，多个channel共享一些线程。详细查看[多个channel共享线程池](https://blog.csdn.net/weinianjie1/article/details/50611379) 和 [如何配置多个消费者线程](https://blog.csdn.net/csdnofzhc/article/details/79422858)




## 恒大项目中你认为有何不足？也就是你认为微服务架构有何缺点？

1.对运维和测试来说更加复杂，要求运维较高的技术水平，因为部署更加复杂要理清组件间的依赖，一个功能的实现涉及到多个组件时，部署时都要部署，很容易漏掉，从一开始一定要构建起自动化测试和自动化部署。多个docker部署。对测试来说，一条用例涉及到多个组件的会比较沟通困难，定位问题的时候也很麻烦。

2.拆分了服务，但是业务逻辑从代码上的依赖变成了组件间通信的依赖，接口设计要非常谨慎遵循一定的原则进行完善的管理，否则一个接口改变影响多个组件，极大影响开发效率。恒大是通过client封装httpclient接口，mq调用。

3.拆分服务后，多个组件独立部署各自进程内，构成了分布式环境，有些情况下的考虑分布式事务，网络延迟，还有异步消息通信问题就要考虑解决。

4.系统的维护复杂度大大提升，系统中出现故障的频率大大提升，虽然有高可用机制的保障，但是也需要做好良好的自动化监控，不断收集各个组件服务情况，及时发现处理问题。

## 恒大项目开发工具：

项目管理平台：开始是redmine，进入开发阶段后转为jira。 Wiki

版本管理服务器：gitlab 可视化git管理开源项目

项目计划工具：Projects 2013 架构工具 visio

平台运行环境是ubantu，一种linux开源免费系统

前端框架使用vue.js（还有react和angularJS可以用）。前后端完全分离，静态文件放在nginx下面。

## 数据库序列

serial内部其实帮助我们实现了:

创建了一个sequence,名字是表名*字段名*seq

设置id字段为NOT NULL DEFAULT nextval('此sequence')

关联此sequence到相关字段.因为这个sequence关联在表的字段上,所以删除表字段会一起删除,除非owner没有指定

serial类型有3中,smallserial,serial,bigserial,底层分别对应smallint,integer,bigint,分别占2bit,4bit,8bit.


## GPU和CPU对比
- 由于CPU需要很强的通用性来处理各种不同的数据类型，同时有需要逻辑判断又需要引入大量的分支跳转和中断的处理。这些都使得CPU的内部结构异常复杂，CPU不仅被Cache占据了大量空间，而且还有很复杂的控制逻辑和优化电路。相比之下计算能力只是CPU很小的一部分。 GPU面对的则是类型高度统一，相互无依赖的3大规模数据和不需要被打断的纯净的环境，GPU基于大的吞吐量设计，采用了数量众多的计算单元和超长的流水线，但只有简单的控制逻辑并省去了Cache,计算能力突出。综上所述，CPU擅长的是逻辑控制，串行的运算，GPU擅长的是大规模的并发运算。

9.最智能的地方就体现在各个摄像头的功能。人员徘徊 人脸识别 人体检测 重点人员 业主 访客 陌生人 。业主可以方便的开门呼梯，访客可以在获得权限后一定时间内开门呼梯，走到指路牌前，指路牌人脸识别给你指示方向。可以跟踪重点业主交物业费，快递等。可以识别一些人 车的来访次数。

10.两个线程收到联动规则然后去抓图，可以在发送抓图命令时 加锁，rennternlock然后sleep(2000)

## 人脸图片识别方案

建立好人脸库后，传入人脸图片识别有三种方案可选

1. 百度人脸识别 传入一张图片即可返回结果
2. 阿里的人脸识别只能传入两张图片，比较是否为同一人
3. face recongnition是来源的识别算法，识别度较差，也只能传入两张照片来比较

# 其他

- win10删除需要权限的文件夹：可以参考：http://www.xitongzhijia.net/xtjc/20161115/87360.html。

- 使用 gradle init --type pom 即可将maven项目转换为gradle项目

- webflux响应式变成 异步的非阻塞的响应式流 https://blog.csdn.net/get_set/article/details/79480233

- 集群多实例的任务执行为避免错误 可以在redis中

- jsonutil的转json的时候，是根据get方法来判断的，所以不能有额外的get方法

- aop实现方法日志记录，参考cas实现

- 要加紧学习tomcat nginx的基本用法，了解war包等的生成结构。

- openresty是基于nginx的快速开发web的框架，性能非常好，并发轻松上一万，可以把一些不太复杂的业务放到nginx层。

- gogs比gitlab安装简单，功能差不多。是国人写的开源产品，适合自己托管或小项目。

- 批量更新的小方法

  场景:传入通道对象集合，要根据indexCode和Version去修改status字段的值。

  方法：`update channel set status =1 where indexCode || "_" || version in <forEach/>` 循环里传入的是拼好的indexCode_version字符串。

- 记住proxy_pass 地址的最后一个斜杠是关键 访问/netdata/xxx/bbb时，如果代理地址最后3有/，那么会转发到http://localhost:19999/xxx/bbb; 访问 /netdata/xxx/bbb时，如果代理地址最后没有/，那么会转发到http://localhost:19999/netdat...

- acunetix程序平台漏洞扫描工具，百度awvs。登录可以录播，从而创建脚本来模拟登录

- wireshark抓包体会三次握手四次挥手

- 用户需求 产品需求 设计交互图，总体设计 概要设计，UCD效果图，详细开发计划

- 海康日志是在代码里硬编写逻辑，然后会放到一个长度为10000的BLOCKINGQUEUE里。这个队列会在类加载时初始化好。

- apache common的utils写的很好，如果能看一遍就厉害了。

- xml语言： extention markup language ，可扩展的标记语言。有两种解析方式，DOM解析是将文档一次性的加载进内存，形成树型结构再解析。如果文档特别大，容易导致内存溢出，但是可以很方便的对xml进行编辑。SAX解析是事件驱动的方式，一行一行的解析，不能对文档进行增删改的操作，但是当文档特别大时不会导致内存溢出。XML约束有DTD和schema约束。现在都是使用schema约束了。

- 1.DTD语法是自成一体的.Schema语法就是XML的语法.

- - 2.Schema的语法就是XML的语法所以更容易被解析器所解析.
  - 3.Schema支持名称空间.
  - 4.Schema有比DTD更加强大的语义和语法的约束.

- 常见的web服务器：Apache webSphere（IBM公司研发,收费的大型服务器软件） WebLogic（BEA公司研发,收费的大型服务器软件） Tomcat （Apache组织研发,免费的小型的服务器软件） JBoss

- **我亲自比较**了下两种代理方式的性能差距，在1.6和1.7的时候，JDK动态代理的速度要比CGLib动态代理的速度要慢，但是并没有教科书上的10倍差距，在JDK1.8的时候，JDK动态代理的速度已经比CGLib动态代理的速度快很多了，希望小伙伴在遇到这个问题的时候能够有的放矢！

- http1.1的特点是长连接，而且现在几乎都是1.1

- 为什么前段的请求都带一个时间戳？怎么解决重复提交？ 用户点击一次后只有等后台返回了结果才允许点击下一次！

- foreach循环怎么跳出去 在foreach里return的作用和continue一样

- 1.只想取出某一个对象，可转成map.jsonObject取值

  ```java
  LinkedHashMap linkedHashMap = (LinkedHashMap) triggerClientImpl.queryTriggerByEventKey(triggerDto).getData();
  TriggerDto triggerResDto = new JSONObject(linkedHashMap).toJavaObject(TriggerDto.class);
  ```

- 海康定时任务，会监听contextrefreshedEvent事件，然后去开启定时任务，利用scheduler的api来添加，修改，删除任务，还有trigger等

- get请求也可以用一个对象来接受参数，前面不用任何注解

- h_mxc

- IBM的学习里都是高质量的文章好好计划学习

- 时间同步。NTP是网络时间协议(Network Time Protocol)，它是用来同步网络中各个计算机的时间的协议

- Rest接口。必须要有服务名和版本号 资源编号IndexCode。如https：[//127.0.0.1/userService/v1.0/getUSer/1242](https://127.0.0.1/userService/v1.0/getUSer/1242)

- 参数校验。一般来说，不涉及dao层的校验都放到controller层。 可以拦截到达controller方法的请求，利用@validated注解来进行参数的校验，不合法可以直接抛出异常，在统一异常处理器中处理，返回需要的信息。spring validate就是实现了这个功能的框架。

- 连接远程主机可以用serity CRT,也可以用XSheel。

- 卸载服务。在命令行通过 sc delete 服务名就可以卸载电脑上的服务。服务名可以通过services.msc，右键点击相应的服务可以查看服务名

- 如何在使用andLike（“name","%"+name+"%"）防止sql注入的问题,需要将name字段进行工具类进行过滤*和%。工具类中进行sql.replace("*","/_").replace("%","/%")过滤。然后andLike（”name","%"+name+"%"）就可以防止sql注入了。

- 海康图片服务器用的是自己开发的pss，后来变成pstore图片服务器。开源的有fastDFS。

- easyexcel在poi的基础上做了二次优化，解决了poi的内存占用问题，解决了poi的并发造成的报错问题。在大数据量的情况下导入导出性能较强排查出一个组件概率性死锁问题，导致的tomcat启动阻塞。

- 语言国际化本地化要考虑长度，西方语会显示不下。而且，有些语言是从右到左的，需要镜面显示

# career

*2017.9.13入职。*

*12.4到广州参与恒大项目，6.9结束。*

*11.22日 架构设计完成*

*12.08日 技术验证初步完成 1*

*2.28日 POC1正式退出测试* 

*02.10日 POC2正式退出测试* 

*03.22日 POC3正式退出测试*

*6.15参与定制项目 重庆百货商场 和陈号29。要求6.25部署到项目现场*

*7.18前完成立体防控模块的开发，7月底Ar设备立体防控模块要随着产品卖出去*

*8月9月完成区域入侵的开发，从产品需求文档，总体设计，概要设计，交互，效果图都参与了其中，（前期分配了三个人）但由于人员紧张独立完成代码的编写*
