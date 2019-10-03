# spring-boot

## 	什么是 spring-boot

​			spring-boot 可以轻松的创建可运行的独立的，生产等级的基于 spring 的应用程序。

## 	为什么使用 spring-boot

​			传统的 spring 框架虽然是轻量级的， 但是其配置文件依然是重量级的， 而且在 maven 依赖工程中对依赖的注入也是重量级操作。 这些重量级配置在不同的项目开发中存大同有小异， 为了使得开发人员能将更多的时间精力专注于业务层开发， spring-boot 将这些大同的环境配置进行了封装， 并开放可自定义配置文件方式， 使得开发者从大量繁琐的依赖与配置文件中解脱。

## 	如何使用 spring-boot

​			只需要在 maven 项目中注入相关 spring-boot 依赖即可。每种模块都封装了开发所需的大量依赖。

​			1、继承 spring-boot-parent 父工程

​			2、依赖web服务功能模块 spring-boot-starter-web	

​			3、创建引导类，负责启动项目工程。

​		

```xml
(1)
	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-parent</artifactId>
        <version>2.0.1.RELEASE</version>
    </parent>
(2)
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```



```java
(3)
@SpringBootApplication
public class MySpringBootApplication {
    public static void main(String[] args) {
        SpringApplication.run(MySpringBootApplication.class);
    }
}
```



# IDEA 热部署方式

​	通过热部署方式可以使得我们在开发过程中更新代码后，不必再重新启动 spring-boot 引导类

​	使用 shift + ctrl + alt + / 打开维护菜单

​	修改 registry 中 `compiler.automake.allow.when.app.running` 开启。

​	修改 settings 中 compiler 的 `build project automatically` 选项开启。

​	修改 Run/Debug Configuration 中 Runniing Application 的 do nothing 。



# spring-boot 注解

 

| @SpringBootApplication   | 该注解声明该类是 spring-boot 引导类。默认自动扫描该类所在包下的，所有类， 所有子包下的其他类。 |
| ------------------------ | ------------------------------------------------------------ |
| @SpringBootConfiguration | 说明该类是配置文件类，可以被 @ComponentScan 扫描到           |
| @EnableAutoConfiguration | 该注解启用了程序的自动配置功能，通过 @Import 注解加载了  AutoConfigurationImportSelector 类文件，通过该类读取 META-INF 下的配置文件 spring.factories。 |
| @ConfigurationProperties | 注解声明了类的作用为加载 prefix 配置字段，类中的属性为配置文件中被 prefix 引用的字段下的 key， 并被赋值 key 所对应的 value 值。 |

# **自动配置

![spring-boot-自动配置原理](D:\Java\笔记\spring-boot-自动配置原理.png)



1. @SpringBootApplication ： 该注解声明该类为 spring-boot 引导类，默认扫描当前包及其子包下的所有类文件。通过该注解加载到 @SpringBootConfiguration 和 @EnableAutoConfiguration 配置文件加载注解。

2. @SpringBootConfiguration 声明了一个类为配置类，该类会被 @ComponentScan 扫描到，类中被 @Bean 修饰的方法会被作为一个 bean 对象使用。 类似 `<bean id="方法名" class="修饰类型"></bean>`

3. @EnableAutoConfiguration 启动自动配置功能，通过 @Import 导入AutoConfigurationImportSelector.class 文件， 调用类中的 getCandidateConfigurations 方法， 该方法中调用用 SpringFactoriesLoader 类的 loadFactoryNames 方法加载 classpath 下的所有的 META-INF/spring.factories 配置文件。

4. spring.factories 的加载方式使用了 SPI(Service Provider Interface) 机制，配置文件中配置了大部分应用开发时使用的框架或组件接入时需要的全限定类名 auto configure 供 spring 自动加载，这些 auto configure 中的类在达成规定条件时才会进行自动加载。

5. auto configure 被加载条件：每个 auto configure 都会包含以下部分注解，当类被实例化后即可被加载。

   - @Configuration 类似于新建一个 spring.xml 配置文件，开启配置模式， 充当 <beans> 标签。

   - @ConditionalOnClass({ Make_A.class, Make_B.class }) 当 classpath 下满足拥有所有指定的类的条件时，才会为当前类创建 bean 实例对象。

   - @AutoConfigureAfter( Make_C.class) 当 Make_C.class 实例化以后， 再对当前类进行实例化。

   - @ConditionalOnWebApplication(type = Type.SERVLET) 当应用基于 Servlet 的 web 应用时

   - @ConditionalOnMissingBean(name = "someOneBean") 当 spring 容器中没有 someOneBean 时

   - @ConditionalOnProperty(name = "spring.some.enabled", matchIfMissing = true) 检查 spring.some.enabled 是否存在，不存在则设置为 true。

     **注①：@Conditional* 为与条件，满足所有的 Conditional 后当前类才可被实例化为 bean**

     **注②：当类达到被实例化的条件后，以下注解才可生效。**

   - @EnableConfigurationProperties( Make_D_Properties.class ) 开启 Make_D_Properties.class 的内嵌支持， 自动为 Make_D_Properties 创建实例， 该类被 @ConfigurationProperties 注解声明为配置文件加载类， 加载指定的 prefix 下的值。

     **注③： 可以使用该方式自定义开发第三方 starter**

# YML(YAML)配置方式

```tiki wiki
	YML文件格式是YAML (YAML Aint Markup Language)编写的文件格式，YAML是一种直观的能够被电脑识别的的数 据数据序列化格式，并且容易被人类阅读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导 入，比如： C/C++, Ruby, Python, Java, Perl, C#, PHP等。YML文件是以数据为核心的，比传统的xml方式更加简洁。
YML文件的扩展名可以使用.yml或者.yaml。
```



```yaml
#普通数据配置
name: root

#对象数据配置两种方式
person:
    name: liuyan
    age: 31
    addr: beijing

map.person2: {name: liuyan, age: 32, addr: beijing}

#数组配置两种方式
a:
	weekdays:
      - sunday
      - monday
      - tuesday
      - wednesday
      - thursday
      - friday
      - saturday

b:weekdays2: [sunday, monday, tuesday, wednesday, thursday, friday, saturday]

#对象数组配置
peoples:
  - name: liuyan
    age: 33
  - name: xukun
    age: 13
  - name: hongbao
    age: 24
```

# 在类中使用 yml 配置属性， 属性注入

```java
#普通属性注入
@Value("${keyName}")
    
#对象属性注入
@Compenent
@ConfigurationProperties(prefix = "person")
    
#集合属性注入
@ConfigurationProperties(prefix = "a")
    private List<String> weekday
    
@ConfigurationProperties(prefix = "b")
    private String[] weekday2    
    
#对象集合注入
@ConfigurationProperties(prefix = "c")
    private List<Person> peoples   
    
#map注入
@ConfigurationProperties(prefix = "map")
    private Map<String, Object> person2   
```



# Actuator

​	跟踪监控工具，监控应用程序的运行和健康状况

​	用来监控 spring-boot 的运行状况

​	使用：

​			在 maven 中导入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

- 访问端点

actuator访问路径:http://localhost:8080/actuator

默认情况下,SpringBoot处于安全的考虑,自己开放了部分的端点.

如果需要全部开放,配置如下信息

```
management.endpoints.web.exposure.include=*
```

- 常用的端点

  1. 查看应用程序加载的Bean:[/actuator/beans](http://localhost:8080/actuator/beans)
  2. 自动配置详解:[/actuator/conditions](http://localhost:8080/actuator/conditions)
  3. 查看配置属性:[/actuator/configprops](http://localhost:8080/actuator/configprops)
  4. 查看环境配置信息:[actuator/env](http://localhost:8080/actuator/env)
  5. 映射信息:[/actuator/mappings](http://localhost:8080/actuator/mappings)
  6. 系统性能度量指标:[/actuator/metrics](http://localhost:8080/actuator/metrics)
     - 检查某个具体指标的信息:http://localhost:8080/actuator/metrics/XXX
  7. 追踪Web请求:[/actuator/httptrace](http://localhost:8080/actuator/httptrace)
  8. 线程快照:[/actuator/threaddump](http://localhost:8080/actuator/threaddump)
  9. 健康状况:[/actuator/health](http://localhost:8080/actuator/health)

  ```
  # 处于安全考虑,SpringBoot没有开放系统安全详细信息,配置如下信息
  management.endpoint.health.show-details=always
  ```

  1. 系统信息:[/actuator/info](http://localhost:8080/actuator/info)

  ```
  # 配置系统定义信息
  info.contactEmail=support@myreadinglist.com
  info.telephone=XXXXXXXX
  ```

# 部署 spring-boot-web 项目

## 1、jar包部署

​		  在工程目录下使用 mvn package 命令，将项目打包。

​		 使用 java -jar xxx.jar 启动项目

## 2、war包部署

​		1、exclusion 去除 spring 自带的 tomcat 依赖。

​		2、创建引导类 @SpringBootApplication 替换掉 main 引导。

​		3、打包 war 包

​		4、上传 tomcat 服务器