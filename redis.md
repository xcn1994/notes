# redis

## spring data redis的xml配置

```xml
    <context:property-placeholder location="*.properties" />
    <!-- redis 相关配置 -->
    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxIdle" value="${redis.maxIdle}" />
        <property name="maxWaitMillis" value="${redis.maxWait}" />
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />
    </bean>

	<!-- 创建工厂对象 -->
    <bean id="JedisConnectionFactory"
          class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
          p:host-name="${redis.host}" p:port="${redis.port}" p:password="${redis.pass}"
          p:pool-config-ref="poolConfig"/>
	
 	<!-- 通过工厂对象创建redisTemplate对象 -->
    <bean id="redisTemplate"
          class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="JedisConnectionFactory" />
    </bean>
```



## redis properties

```properties
#访问地址  
redis.host=127.0.0.1  
#访问端口  
redis.port=6379  
#注意，如果没有password，此处不设置值，但这一项要保留  
redis.password=  
  
#最大空闲数，数据库连接的最大空闲时间。超过空闲时间，数据库连接将被标记为不可用，然后被释放。设为0表示无限制。  
redis.maxIdle=300  
#连接池的最大数据库连接数。设为0表示无限制  
redis.maxActive=600  
#最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。  
redis.maxWait=1000  
#在borrow一个jedis实例时，是否提前进行alidate操作；如果为true，则得到的jedis实例均是可用的；  
redis.testOnBorrow=true
```



# redis 模型

单线程、NIO 的异步模型。

丰富的数据类型。

内存运算

支持 cluster 集群模式

## 单线程模型

redis 基于reactor 模式开发网络事件处理器，连接使用 socket 方式 

1. redis 服务器启动后其中的 server socket 保持监听状态；

2. 客户端第一次连接 redis，通过socket发送连接请求；

3. **server socket** 的 accept 方法接受到请求后，**创建事件** AE_READABLE；

4. AE_READABLE 被 **IO 多路复用器监听**，并将该事件**推进事件队列中**。事件队列：【ss】;

5. 队列中的事件提交到内存中的**文件事件分派器**中；

6. 文件事件分派器根据事件内容 ss，将事件分派到**连接应答处理器**中；

7. 连接应答处理器接受到事件后，**创建**与客户端交互的 **socket**，并将该 socket **关联到命令请求处理器**上；

   ------------------------------------------连接完成，开始处理请求---------------------------

8. 客户端发送 set key value 请求；

9. redis 中对应的 socket 获取到请求，**创建** AE_READABLE 事件；

10. AE_READABLE 事件被 **IO 多路复用器**监听到，推入**事件队列**； 事件队列：【so】

11. 队列的事件提交到内存中的**文件事件分派器**中；

12. 文件事件分派其将事件分派到关联的**命令请求处理器**上；

13. 命令请求处理器读出 key value，在内存中操作；

14. 请求处理完成后准备响应数据，将 socket 的 AE_WRITEABLE 事件**关联到命令回复处理器**上；

15. 客户端读取响应数据时，redis 的 socket 创建 AE_WRITEABLE 事件；

16. IO 多路复用器将事件推入队列；【so】

17. 队列事件提交**文件事件分派器**中；

18. 分派器根据事件内容分派到**命令回复处理器**上；

19. 命令回复处理器响应**结果写入 socket**，如：ok，供客户端读取，之后**解除 AE_WRITEABLE关联**。

20. 一次操作完成。

注：

队列结构为先进先出，当多个事件被推入队列时，【so2, so】。

从事件队列、文件事件分派器到连接应答处理器、命令请求处理器、命令回复处理器，统称为文件事件处理器。

## 支撑高并发

事件处理器： 

​	基于非阻塞的IO多路复用器将请求提交队列，

​	处理器纯内存操作，

​	单线程避免频繁的上下文切换操作（线程切换）。