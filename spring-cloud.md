# spring-cloud

## 1、是什么

微服务是一种服务架构风格

由单体架构  ------ 分布式 ----- SOA ------微服务  转化而来

微服务架构，一种解决方案，是框架的集合。

**提供了微服务开发所需的各种组件。**

![cloud2](D:\Java\笔记\cloud2.png)

### 一、Euraka 注册中心

管理微服务的信息：ip、端口、状态

使用注册中心进行微服务的统一管理，让客户端通过注册中心访问微服务，将微服务很好的维护在后台。

**单机模式**

**高可用模式**

```xml
<!--父工程-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring‐cloud‐dependencies</artifactId>
    <version>Finchley.SR1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

```xml
<!--子模块-->
<dependencies>
    <!‐‐ 导入Eureka服务的依赖 ‐‐>
    <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring‐cloud‐starter‐netflix‐eureka‐server</artifactId>
    </dependency>
</dependencies>
```

```yaml
server:
	port: ${PORT:50101}
eureka:
  client:
  	#服务注册，是否将自己注册到Eureka服务中,单机模式为false
    registerWithEureka: true 
    #服务发现，是否从Eureka中获取注册信息，单机模式为false
    fetchRegistry: true 
    serviceUrl: 
    #Eureka客户端与Eureka服务端的交互地址，高可用状态配置对方的地址，单机状态配置自己（如果
不配置则默认本机8761端口）
      defaultZone: ${EUREKA_SERVER:http://eureka02:50102/eureka/}
  server:
  	#是否开启自我保护模式
    enable‐self‐preservation: false 
    #服务注册表清理间隔（单位毫秒，默认是60*1000）
    eviction‐interval‐timer‐in‐ms: 60000 
  instance:
    hostname: ${EUREKA_DOMAIN:eureka01}
```

```java
//启动类
//标识这是一个Eureka服务
@EnableEurekaServer
@SpringBootApplication
```

idea 测试环境配置

```
-DPORT=50102 -DEUREKA_SERVER=http://localhost:50101/eureka/ -DEUREKA_DOMAIN=eureka02
-DPORT=50101 -DEUREKA_SERVER=http://localhost:50102/eureka/ -DEUREKA_DOMAIN=eureka01
```

**微服务注册**

```xml
<!‐‐ 导入Eureka客户端的依赖 ‐‐>
<dependency>
	<groupId>org.springframework.cloud</groupId>    
	<artifactId>spring‐cloud‐starter‐netflix‐eureka‐client</artifactId>    
</dependency>
```

```yaml
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      #注册目的地url，多个用逗号分隔
      defaultZone: ${EUREKA_SERVER:http://localhost:50101/eureka/, http://localhost:50101/eureka/}
  instance:
    #将自己的ip注册到注册中心
    prefer-ip-address: true
    ip-address: ${IP_ADDRESS:127.0.0.1}
    instance-id: ${spring.application.name}:${server.port}
```

```java
//启动类，打开客户端显示
@EnableDiscoveryClient
```



#### 1>负载均衡: ribbon 

**Ribbon**: 基于 http、tcp 的**客户端负载均衡器**。

均衡算法：轮询、加权、随机、地址哈希等。

通过负载均衡来提高系统的高可用性，集群扩容能力。



在请求服务端的客户端配置 ribbon：

添加ribbon依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring‐cloud‐starter‐ribbon</artifactId>
</dependency>
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
</dependency>
```

设置 ribbon 参数

```yaml
ribbon:
  MaxAutoRetries: 2 #最大重试次数，当Eureka中可以找到服务，但是服务连不上时将会重试
  MaxAutoRetriesNextServer: 3 #切换实例的重试次数
  OkToRetryOnAllOperations: false  #对所有操作请求都进行重试，如果是get则可以，如果是post，put等操作没有实现幂等的情况下是很危险的,所以设置为false
  ConnectTimeout: 5000  #请求连接的超时时间
  ReadTimeout: 6000 #请求处理的超时时间
```

启动 eureka 中心

启动多个微服务提供者

在客户端启动类中定义 RestTemplate ，用于发送远程请求

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate(new OkHttp3ClientHttpRequestFactory());
}
```

测试：

```java
@SpringBootTest
@RunWith(SpringRunner.class)
//负载均衡调用
@Test
public void testRibbon() {
    //服务id
    String serviceId = "XC‐SERVICE‐MANAGE‐CMS";
    for(int i=0;i<10;i++){
        //通过服务id调用
        ResponseEntity<CmsPage> forEntity = restTemplate.getForEntity("http://" + serviceId
+ "/cms/page/get/5a754adf6abb500ad05688d9", CmsPage.class);
        CmsPage cmsPage = forEntity.getBody();
        System.out.println(cmsPage);
    }
}
```

debug 下观察 RibbonLoadBalancerClient 中的地址变化， 在负载均衡的方式下，服务1和2 轮流处理请求

```java
public <T> T execute(String serviceId, ServiceInstance serviceInstance, LoadBalancerRequest<T> request) throws IOException {
        Server server = null;
        if (serviceInstance instanceof RibbonLoadBalancerClient.RibbonServer) {
            server = ((RibbonLoadBalancerClient.RibbonServer)serviceInstance).getServer();
        }

  ----> if (server == null) {  //server=30101
            throw new IllegalStateException("No instances available for " + serviceId);
        } else {
            ......
```

#### 2> feign

一种轻量级 rest 客户端，可以便捷地实现 http 客户端。 spring-cloud 引入了 feign， 并集成了ribbon实现客户端的负载均衡调用。

在 ribbon 客户端中添加依赖: 由于 feign 中集成了 okhttp 和 ribbon， 所以不需要再添加他俩的依赖

```xml
   <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>com.netflix.feign</groupId>
            <artifactId>feign-okhttp</artifactId>
   </dependency>
```

参照 swagger2 接口文档定义 client 接口， url、参数类型、返回值都必须与接口文档提供一致。

```java
@FeignClient(value = XcServiceList.XC_SERVICE_MANAGE_CMS)
public interface CmsPageClient {

    @GetMapping("/cms/page/get/{id}")
    public CmsPage findById(@PathVariable("id") String id);
}
```

```java
//根据工程结构，定义服务器的常量集合类
public class XcServiceList {
    public static final String XC_GOVERN_CENTER = "xc-govern-center";
    public static final String XC_SERVICE_PORTALVIEW = "xc-service-portalview";
    public static final String XC_SERVICE_SEARCH = "xc-service-search";
    public static final String XC_SERVICE_MANAGE_COURSE = "xc-service-manage-course";
    public static final String XC_SERVICE_MANAGE_MEDIA = "xc-service-manage-media";
    public static final String XC_SERVICE_MANAGE_CMS = "xc-service-manage-cms";
    public static final String XC_SERVICE_UCENTER = "xc-service-ucenter";
    public static final String XC_SERVICE_UCENTER_AUTH = "xc-service-ucenter-auth";
    public static final String XC_SERVICE_UCENTER_JWT = "xc-service-ucenter-jwt";
    public static final String XC_SERVICE_BASE_FILESYSTEM = "xc-service-base-filesystem";
    public static final String XC_GOVERN_GATEWAY = "xc-govern-gateway";
    public static final String XC_SERVICE_BASE_ID = "xc-service-base-id";
    public static final String XC_SERVICE_MANAGE_ORDER = "xc-service-manage-order";
    public static final String XC_SERVICE_LEARNING = "xc-service-learning";

}
```

```java
//启动类添加注解
@EnableFeignClients   //扫描创建 feignClient 的 bean 对象
@EnableDiscoveryClient  //向远程注册中心注册
```

测试：

```java
@AutoWired
CmsFeignClient cmsFeignClient

@Test 
public void test(){
	CmsPage cmsPage = cmsFeignClient.findById("adf22321a");
}
```

**注意事项：**

1、有参方法必须根据参数请求方式添加

​		@PathVariable("xxx")   [/{xxx}]风格

​		@RequestParam("xxx")  key=value风格

2、feignClient 返回值为复杂对象时其类型必须有无参构造函数。



**Feign 工作原理如下：**

1. 启动类添加@EnableFeignClients注解，Spring会扫描标记了@FeignClient注解的接口，并生成此接口的代理对象
2. @FeignClient(value = XcServiceList.XC_SERVICE_MANAGE_CMS)即指定了cms的服务名称，Feign会从注册中心获取cms服务列表，并通过负载均衡算法进行服务调用。
3. 在接口方法 中使用注解@GetMapping("/cms/page/get/{id}")，指定调用的url，Feign将根据url进行远程调用。