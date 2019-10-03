# spring security 配置

## web.xml

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring/springSecurity.xml</param-value></context-param><listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class></filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

​		注：filter-name为springSecurityFilterChain不可变，该框架对其固定扫描加载。



## spring-security.xml

```xml
<!--配置拦截规则,不拦截以下路径-->
<http pattern="/login.html" security="none"></http>
<http pattern="/plugins/**" security="none"></http>
<http pattern="/css/**" security="none"></http>
<http pattern="/img/**" security="none"></http>
<http pattern="/js/**" security="none"></http>
```



```xml
<!--配置拦截重定向访问路径-->
<http use-expressions="false">
    <!--拦截路径，及可访问要求-->
    <intercept-url pattern="/*" access="ROLE_ADMIN"></intercept-url>    
    <form-login username-parameter="username"
                password-parameter="password"
                authentication-failure-url="login.html"
                authentication-success-forward-url="/admin/index.html"
                default-target-url="/admin/index.html"  
                login-processing-url="login.do"
                always-use-default-target="true">
    </form-login>
    
<!--关闭跨站请求伪造拦截功能-->
    <csrf disabled="true"></csrf>
<!--如果使用了框架页面，设置框架页的策略为SAMEORIGIN-->
	<headers>
        <frame-options policy="SAMEORIGIN"></frame-options>
    </headers>
</http>
```

​		 注：框架页面：页面中使用<iframe>标签加载了其他页面，policy在 “协议：//地址：端口号” 一致时为符合 SAMEORIGIN规则，允许加载。配置为DENY时，拦截所有框架页。 配置为ALLOW-FROM为允许指定页面加载访问。



```xml
<authentication-manager>
    <authentication-provider>
        <!--<password-encoder></password-encoder>-->
        <user-service>
            <!--测试时使用标签-->
            <user name="admin" password="{noop}123" authorities="ROLE_ADMIN"></user>        	  </user-service>
</authentication-provider></authentication-manager>
```

​		 注：security会自动扫描加密解密规则，测试用户时，密码没有对应的加密解密规则，加入{noop}提示无密码规则



## 前端配置

​			在页面中配置action=“/login”，此请求url为security框架自动生成路径，也可在xml中自定义url，使用login-prosessing-url = “url”。

