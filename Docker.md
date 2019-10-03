# Docker

## 什么是 docker

​	虚拟化技术：是虚拟出一套硬件， 在硬件上安装完整的操作系统，在系统中运行需要的应用进程。

​	![虚拟化技术](D:\Java\笔记\虚拟化技术.png)

​	

容器：容器内的应用进程直接运行在宿主的内核，容器没有自己的内核，也没有进行硬件虚拟。比传统虚拟机更加轻巧。

![虚拟化技术](D:\Java\笔记\虚拟化技术.png)

Docker 对容器进行了进一步的封装， 从文件系统、网络隔离到进程等，极大的简化了容器的创建和维护。使得docker 成为一个轻量化的技术。

## 为什么使用 docker

传统开发使用了传统的全虚拟化技术，其占用空间大，运行速度慢，维护率高，用户体验略差。

为了使得开发者在开发时能够快速的搭建部署服务器，降低维护开销，以及稳定的平台迁移，使用 docker 技术可以极大的优化传统的全虚拟化技术的各种缺点。

| 特性       | 容器               | 虚拟机      |
| :--------- | :----------------- | :---------- |
| 启动       | 秒级               | 分钟级      |
| 硬盘使用   | 一般为 `MB`        | 一般为 `GB` |
| 性能       | 接近原生           | 弱于        |
| 系统支持量 | 单机支持上千个容器 | 一般几十个  |

## 如何使用 docker

docker 使用 C/S 架构，提供一个 docker 命令集以及全套的 RESTfull API。

![docker结构](D:\Java\笔记\docker结构.jpg)

docker 包括三个组件

**镜像：**docker 镜像是一个微型的文件系统，像 linux 镜像一样，特点就是体积小而便捷，易于迁移。镜像的实际是由一组文件系统组成，这组文件系统层层构建，前一层是后一层的基础，每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。

**容器：**是镜像运行时的实体。容器可以创建、启动、停止、删除、暂停等操作。其实质是进程，但与宿主执行的进程不同，容器相当于一个运行在系统中的小系统，拥有自己的 root 文件系统、网络配置、进程空间、用户空间等。

**仓库：**docker registry 用来集中存储、分发镜像的服务。

一个 **Docker Registry** 中可以包含多个 **仓库**（`Repository`）；每个仓库可以包含多个 **标签**（`Tag`）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。



### 通过yum安装

​		[官方安装文档](https://docs.docker.com/install/linux/docker-ce/centos/)

​	1、先删除旧版本

```txt
$ sudo yum remove docker \
				  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

​	   2、安装依赖包

```txt
$ sudo yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2
```

 	3、修改 yum 源

```txt
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

​	4、安装 docker

```txt
$ yum install docker
$ docker -v
```

​	5、测试 docker

```txt
#启动docker
systemctl start docker

#关闭docker
systemctl stop docker

#重启docker
systemctl restart docker

#查看docker状态
systemctl status docker

#配置开机启动docker
systemctl enable docker

#查看docker概要信息
docker info

#查看docker帮助文档
docker help

#测试
$ docker run hello-world
```

注：测试报错，可能是系统版本问题，使用 yum update 更新操作系统

```txt
$ yum install
```



### 镜像操作

```
#列出本地所有镜像
docker images

#在Docker Hub搜索镜像
docker search 镜像名称

#从Docker Hub拉取镜像
docker pull 镜像名称:Tag(默认为最新的)

#删除某个镜像
docker rmi 镜像ID

#删除所有镜像
docker rmi `docker images -q`
```

注：由于注册中心在国外的服务器中，下载速度会比较慢，可以配置国内代理服务器使用。

```t&#39;x&#39;t
修改 /etc/docker/daemon.json
{
	"registry-mirrors": [
		"https://docker.mirrors.ustc.edu.cn",
	    "https://dockerhub.azk8s.cn",
    	"https://reg-mirror.qiniu.com"
    ]
}
重新启动 docker 服务
systemctl docker restart

```



### 容器操作

```txt
#查看正在运行的容器
docker ps

#查询所有的容器
docker ps -a

#查看最后一次运行的容器
docker ps -l

#查看停止的容器
docker ps -f status=exited

#创建并启动容器(交互式)
docker run -it --name=自定义容器名称  镜像名称:tag  /bin/bash

#创建并启动容器(守护式)
docker run -id --name=自定义容器名称  镜像名称:tag  

#登录守护容器
docker exec -it 容器名称/容器ID /bin/bash

#停止容器
docker stop 容器ID/名称

#启动容器
docker start 容器ID/名称
```

### 文件拷贝

```txt
#宿主机--->容器
docker cp 本地文件/目录	容器ID/名称:容器目录 
#容器--->宿主机
docker cp 容器ID/名称:容器目录/文件 	本地目录
```

### 目录挂载（共享）

```txt
docker run -id -v 宿主机目录:容器目录 --privileged=true --name=容器名称 镜像名称:tag 
#eg:docker run -id -v /home:/home --privileged=true --name=mycentos centos:7 
```

## 容器属性数据

```
#查看全部属性数据
docker inspect 容器名称
#查看某个数据,例如IP地址
docker inspect --format='{过滤条件}'  容器ID
#例如查看centos容器的IP
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器ID
```

## 删除容器

```
#删除某个容器
docker rm 容器ID		#不能删除正在运行的容器,必须先停止,再删除
#删除所有容器
docker rm `docker ps -a -q`
```

# 常见容器部署

##  mysql容器

```
#拉取镜像
docker pull mysql

#创建MySQL容器	
##-p 代表端口映射,宿主机映射端口:容器运行端口
##-e 代表添加环境变量 MYSQL_ROOT_PASSWORD 是 root 用户的登陆密码
docker run -id --name pinyougou_mysql -p 33306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql

#进入Mysql容器
docker exec -it pinyougou_mysql /bin/bash

#本地登录Mysql测试
mysql -u root -p

#远程可以通过宿主机的IP:映射端口登录
```

## tomcat容器

```
#拉取镜像
docker pull tomcat:7-jre7

#创建容器,并设置共享目录
docker run -id --name=pinyougou_tomcat -p 9000:8080 -v /usr/local/myhtml:/usr/local/tomcat/webapps --privileged=true tomcat:7-jre7

#进入Tomcat容器查看文件
docker exec -it 容器ID /bin/bash
```

## Nginx部署

```
#拉取镜像
docker pull nginx

#创建容器
docker run -id --name=pinyougou_nginx -p 80:80 nginx

#拷贝容器中的配置文件到宿主机
docker cp pinyougou_nginx:/etc/nginx/nginx.conf nginx.conf

#编辑nginx.conf 添加反向代理
server {
	server_name passport.pinyougou.com;
    listen 80;
    
    location / {
        proxy_pass http://tomcat-cas;
        index index.html index.htm;
    }
}

upstream tomcat-cas {
	server 172.17.0.7:8080;
}
```

## Redis部署

```
#拉取镜像
docker pull redis

#创建容器
docker run -di --name=pinyougou_redis -p 6379:6379 redis

#本地连接测试
redis-cli -h 192.168.247.135
```

# 容器备份和迁移

## 容器保存

```
docker commit 容器名称 镜像名称
```

## 镜像备份

```
docker save -o 包名.tar 镜像
```

## 镜像恢复

```
docker load -i 包名.tar
```

![docker-all](D:\Java\笔记\docker-all.jpg)



