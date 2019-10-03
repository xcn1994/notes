# RESTFUL 

## 一、what

restful 是一种设计风格， 对该风格的实现提供了设计原则和一些约束。

## 二、作用：

看到 URL 就知道需要什么资源

看到 http method 就知道在要干什么

看到 http status code 就知道结果如何

## 三、http method

```
GET  /查询
```

GET方法请求一个指定资源的表示形式. 使用GET的请求应该只被用于获取数据.

```
HEAD
```

HEAD方法请求一个与GET请求的响应相同的响应，但没有响应体.

```
POST /添加
```

POST方法用于将实体提交到指定的资源，通常导致在服务器上的状态变化或副作用. 

```
PUT	  /更新
```

PUT方法用请求有效载荷替换目标资源的所有当前表示。

```
DELETE /删除
```

DELETE方法删除指定的资源。

```
CONNECT
```

CONNECT方法建立一个到由目标资源标识的服务器的隧道。

```
OPTIONS
```

OPTIONS方法用于描述目标资源的通信选项。

```
TRACE
```

TRACE方法沿着到目标资源的路径执行一个消息环回测试。

```
PATCH
```

PATCH方法用于对资源应用部分修改。