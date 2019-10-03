# cookie & session

## http协议

​	在 web 应用服务中，数据的传输依靠 http 协议，http协议构建于 tcp/ip 协议。

​	http 协议有以下几个特性：

​	1、简单速度快：客户端向服务端请求服务只需要一个 URL 即可。http 服务器程序规模小，通讯速度快。

​	2、灵活：可以传输任意格式类型的数据对象。

​	3、无连接：每次连接只处理一个请求，服务器收到客户端应答后断开连接。可以节省传输时间。

​	4、无状态：http协议对事务处理没有记忆功能，服务器不能辨识客户端，也就是说一次请求响应过后，服务器不会记录客户端状态信息，当后续处理需要之前的信息时则必须重新传输数据，由此数据量会增大而影响传输效率。

​	5、支持 B/S, C/S 模式。



**为了解决 http 协议的无状态问题，  web 应用衍生出会话技术， 即 cookie&session 。**

## cookie&session的定义

​	1、cookie：保存在客户端的数据对象，每当客户端通过http协议向服务器发送请求时，都会携带cookie对象。使得服务器可以辨识客户端，让http可以通过cookie来记录状态。

​	2、session：保存在服务器的数据对象，存储特定用户会话所需的属性及配置信息。客户端任意切换web页面时，session保持数据不丢失，直到客户端关闭会话，或session超时间失效。



## 不同点

​	1、cookie保存在客户端，session保存在服务器中

​	2、cookie存储的数据类型固定为ascii，session可存储任意数据类型

​	3、cookie存储容量有限，单个不超过4kb，session容量不限制

​	4、cookie安全性较低，客户端容易被攻击，session服务器中安全性较高

​	5、cookie可长时间保存，session有效时间短，会话关闭或超时即失效



## 关联

​	session的实现依托于cookie：

​	客户端第一次访问服务器时，服务器会生成session保存数据，并生成一个sessionID保存在cookie中并记录所属,让响应携带该cookie信息，响应给客户端，客户端保存该cookie信息。

​	客户端第二次访问服务器时，在请求体中携带cookie信息，服务器接受到请求，先解析该cookie得到sessionid，然后查询session中是否保存该sessionid关联的数据，再进行后续操作。

​	多用来实现登录状态验证。



## 其他问题

### 1、客户端禁止使用cookie

​		session的实现依赖cookie，其根本是将seesionID通过cookie来传输，若禁止了cookie功能，相当于http回到无状态情景。

​		1、在每次请求URL中都携带sessionID参数，通过get/post传输，在服务器端记录该sessionID。

​		2、使用token机制。在客户端第一次访问服务器时，服务器产生token信息响应给客户端，客户端再次请求时携带该token实现与cookie相同的功能。token是无状态的。



### 2、分布式下session问题

​	session是服务器端共享数据时使用的对象，在分布式系统中，一个web应用的多个服务分布在不同的服务器下，每个服务器都有独立的session对象互不相通。为解决分布式服务器之间session共享问题，有两种可靠方案：

1、cookie保存共享数据

​		将共享数据保存在客户端的cookie中，客户端每次请求服务器时，携带该cookie对象。

只能用作简单数据共享，数据量小。



2、session服务器

​		独立出一个session服务器，同一管理分发session对象，当客户端第一次请求时，被请求的服务器生成对应的session对象和cookie对象，将cookie对象响应给客户端，将session对象发送给session服务器，session服务器将对象广播给分布式服务器中，实现分布式session共享。



### 3、跨域请求

 	协议、主机地址（域名）、端口其中一项不同，则为不同域。

​	若客户端 A:8080 向服务器 B:9090 发送请求，此时就会出现跨域问题。使用 CORS 标准，在服务器中允许跨域请求的方法体内对特定的响应头进行设置，开启跨域访问，或是使用spinrgMVC 4.2 以上的版本，使用注解的方式 @CrossOrigin（origins="http://localhost:9105"），使用 “*” 表示所有域都可以对当前域发送跨域请求。

```java
//服务器设置可接受跨域请求
response.setHeader("Access-Control-Allow-Origin", "http://localhost:9105"); 
//操作cookie的话必须配置允许跨域cookie的使用
response.setHeader("Access-Control-Allow-Credentials", "true"); 
```

​	由于 CORS 默认不发送 cookie 和 http 认证信息，所以在服务器端必须开启 Access-Control-Allow-Credentials。在客户端请求中打开 withCredentials 属性。

```js
//客户端发送get请求
$scope.get("http://localhost:8080/cart/addCart.do?cartId="+cartId, {'withCredentials':true}).success{...}
```

