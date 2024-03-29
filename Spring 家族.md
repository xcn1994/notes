# Spring 总结

## 一、什么是 spring 

spring 是一种基于 java 的企业应用开发所创造的框架。 使用 DI、 AOP、i18n、数据绑定、类型转换等技术，支持数据库的事务、jdbc、orm、dao 等技术，集成 RPC、jms、 tasks、缓存（redis）等组件，支持多语言如 kotlin， groovy 等开发，通过 springMVC 支持 web 应用开发，集测试模块于一体的流行框架。其轻量级，易上手、开源、非侵入式、简化开发的特点为我们程序员带来春天。

## 二、解决什么问题

传统的企业应用开发中，我们从对象的创建，到业务逻辑的实现，最后释放内存，几乎所有的过程都需要高度关注，在一个业务逻辑的实现过程中，需要先解决对象创建的问题，而每次业务的使用都会创建一个对象，如若对象没有有效地被回收，则会使得可用资源减少，导致内存溢出，又由于对象创建在业务层中，使得业务逻辑层与其他功能层强耦合在一起，增加了开发量，维护困难。spring 框架解开了对象的创建与业务逻辑的耦合，使得开发者可以专注于业务逻辑，而不需要去关注对象的创建，并且默认使用单例模式，高效利用资源，降低异常地发生几率。

## 三、家族

### 1、spring-framework

Bean 的启动过程：

![](D:\Java\笔记\springBean生命周期.png)

![](D:\Java\笔记\springBean生命周期-2.png)

​		

- 

### 2、springMVC

工作流程：

启动过程：

spring 整合：

DespacherServlet ：

### 3、spring-security



### 4、spring-data

- #### jpa

- #### redis

- #### mongoDB

- #### solr

- #### Elasticsearch

### 5、spring-boot



### 6、spring-cloud



## 四、技术

### IOC、DI

在使用 IOC、DI 技术之前，对象 A 在需要对象 B 的地方主动创建对象 B。

在 ioc 中，主角是对象 A，以对象 A 为主视角观察，我（A）需要对象 B，spring 容器创建对象 B，我（A）被spring 注入了对象 B。B对象的创建由我（A）自己主动创建注入变为由 spring 容器创建后注入给我（A)，这种由主动变被动注入对象的方式称为控制反转。

在 ioc 中的主角是对象 A，主动创建 B 对象，或是被动由 spring 容器注入 B 对象。

在 DI 中，主角为 spring 容器，以 spring 视角观察，当 A 对象需要使用 B 对象时， 我这个 spring 将会把 B 对象注入到 A 对象中， 也就是我（spring）做了依赖注入（DI）动作。

### AOP

面向对象的编程艺术，封装、继承、多态是其特色，在纵向功能（继承、多态）开发中有着十分优秀的作用，但是当子类方法中功能不同，但却有一些相同的过程时，比如日志记录，这时候，通过面向对象不能够对过程级别的代码进行封装，开发代码的冗余量增多，同时增加维护成本（时间、人力、机器损耗、电费、咖啡、感情。。。）；

面向切面的编程艺术，为了弥补面向对象的这个缺陷，取出冗余的代码过程进行封装，

将对象像香肠一样将两头横向切开（需要切开的地方@Pointcut），

端来一碗泡面（切面@Aspect），

吃一口面，吃一口香肠（前置通知@Before），

吃一口香肠，吃一口面（@After 后置通知），

面卷着香肠一口吃（@Around 环绕通知），吃

一口香肠说一句“真香”，然后再吃一口面（@AfterReturning 目标方法返回后通知），

吃一口香肠噎住了，需要喝口泡面汤顺一顺（@AfterThrowing 异常后通知）。

主要做声明式事务管理、日志、权限校验等功能。

实现使用了代理模式包括：

静态代理

动态代理：jdk动态代理，Proxy，对接口模式的动态代理

​					cglib动态代理，对继承模式的动态代理