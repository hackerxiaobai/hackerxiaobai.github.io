---
title: Docker基本使用(一)
date: 2021-04-02 11:20:03
tags:
	- Docker
	- Docker images
	- Docker 数据卷
	- DockerFile	
	- Docker commit
	- Docker network
---

# 内容安排

+ **Docker基础**
+ **Docker镜像讲解**
+ **容器数据卷**
+ **什么是DockerFile**
+ **发布镜像**
+ **Docker网络讲解**
+ **实战:部署一个Redis集群**

## Docker基础

### Docker 是什么

> 一种容器化技术，问度娘吧

### 为什么会出现Docker这个东西

​		我们可以想象一下，之前比如开发一个web，从开发到上线，从操作系统，到运行环境，再到应用配置。作为开发+运维之间的协作我们需要 关心很多东西，特别是各种版本的迭代之后，不同版本环 境的兼容，对运维人员是极大的考验!

<!-- more -->

​		拿一个基本的工程项目的环境来说， Java/Tomcat/MySQL/JDBC驱动包等。安装和配置这些东西有多麻烦就不说了，它还不能跨平台。假如 我们是在 Windows 上安装的这些环境，到了 Linux 又得重新装。况且就算不跨操作系统，换另一台同 样操作系统的服务器，要移植应用也是非常麻烦的。

​		为了解决上述问题，**Docker**给出了一个标准化的解决方案，

​		Docker镜像的设计，使得Docker得以打破过去「程序即应用」的观念。通过Docker镜像 ( images ) 将 应用程序所需要的系统环境，由下而上打包，达到应用程序跨平台间的无缝接轨运作。

### Docker和虚拟机的对比

> 虚拟机(virtual machine)就是带环境安装的一种解决方案。

​		它可以在一种操作系统里面运行另一种操作系统，比如在Windows 系统里面运行Linux 系统。应用程序 对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文 件，不需要了就删掉，对其他部分毫无影响。这类虚拟机完美的运行了另一套系统，能够使应用程序， 操作系统和硬件三者之间的逻辑不变。

**虚拟机的缺点:**

1. 资源占用多 
2. 冗余步骤多 
3. 启动慢

> Docker容器虚拟化技术

​		由于前面虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术:Linux 容器(Linux Containers，缩 写为 LXC)。

​		Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。有了容器，就可以将软件运行所需的 所有资源打包到一个隔离的容器中。容器与虚拟机不同，不需要捆绑一整套操作系统，只需要软件工作 所需的库资源和设置。系统因此而变得高效轻量并保证部署在任何环境中的软件都能始终如一地运行。

### Docker的基本组成

> Docker 架构图

{% asset_img 1.png Docker结构图  %}

**镜像(image):**

> Docker 镜像(Image)就是一个只读的模板。镜像可以用来创建 Docker 容器，一个镜像可以创建很 多容器。 就好似 Java 中的 类和对象，类就是镜像，容器就是对象!

**容器(container):**

> Docker 利用容器(Container)独立运行的一个或一组应用。容器是用镜像创建的运行实例。 它可以被启动、开始、停止、删除。每个容器都是相互隔离的，保证安全的平台。
>
> 可以把容器看做是一个简易版的 Linux 环境(包括root用户权限、进程空间、用户空间和网络空间等) 和运行在其中的应用程序。。
>
> 容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的`最上面那一层是可读可写
> 的`。

**仓库(repository):**

> 仓库(Repository)是集中存放镜像文件的场所。 仓库(Repository)和仓库注册服务器(Registry)是有区别的。仓库注册服务器上往往存放着多个仓
>
> 库，每个仓库中又包含了多个镜像，每个镜像有不同的标签(tag)。 仓库分为公开仓库(Public)和私有仓库(Private)两种形式。
>
> 最大的公开仓库是 Docker Hub(https://hub.docker.com/)，存放了数量庞大的镜像供用户下载。 国内的公开仓库包括阿里云 、网易云 等

### Docker的安装

> 看官网吧！！！

### Docker的简单使用

+ 我们先来第一个简单实用

+ ```shell
  docker run hello-world
  ```

这个命令一执行，到底发生了什么，整个运行逻辑就是下图

{% asset_img 2.png 运行流程图  %}

### Docker命令记录

>  `帮助命令`

```shell
docker version # 显示 Docker 版本信息。
docker info # 显示 Docker 系统信息，包括镜像和容器数。
docker --help # 帮助
```

> `镜像命令`

**docker images**

```shell
# 列出本地主机上的镜像 
[root@wl ~]# docker images 
REPOSITORY 		   TAG 	     IMAGE ID	       SIZE         CREATED
hello-world      latest    bf756fb1ae65    13.3kB       4 months ago

# 解释
REPOSITORY 镜像的仓库源 
TAG 镜像的标签 
IMAGE ID 镜像的ID 
CREATED 镜像创建时间 
SIZE 镜像大小

# 同一个仓库源可以有多个 TAG，代表这个仓库源的不同版本，我们使用REPOSITORY:TAG 定义不同 的镜像，如果你不定义镜像的标签版本，docker将默认使用 lastest 镜像!
# docker images 可带选项
-a: 列出本地所有镜像 
-q: 只显示镜像id 
--digests: 显示镜像的摘要信息
```

**docker search**

```shell
# 搜索镜像
[root@wl ~]# docker search mysql
NAME       DESCRIPTION                 STARS        OFFICIAL
mysql      MySQL is a widely used...   9484         [OK]

# docker search 某个镜像的名称 对应DockerHub仓库中的镜像
# 可选项
--filter=stars=50 : 列出收藏数不小于指定值的镜像。
```

**docker pull**

```shell
# 下载镜像
[root@wl ~]# docker pull mysql
Using default tag: latest # 不写tag，默认是latest latest: Pulling from library/mysql
54fec2fa59d0: Already exists
bcc6c6145912: Already exists
951c3d959c9d: Already exists
05de4d0e206e: Already exists
319f0394ef42: Already exists
d9185034607b: Already exists
013a9c64dadc: Already exists
42f3f7d10903: Pull complete
c4a3851d9207: Pull complete
82a1cc65c182: Pull complete
a0a6b01efa55: Pull complete
bca5ce71f9ea: Pull complete
Digest: sha256:61a2a33f4b8b4bc93b7b6b9e65e64044aaec594809f818aeffbff69a893d1944 # 签名
Status: Downloaded newer image for mysql:latest docker.io/library/mysql:latest # 真实位置
# 指定版本下载
[root@kuangshen ~]# docker pull mysql:5.7
```

**docker rmi**

```shell
# 删除镜像
docker rmi -f 镜像id # 删除单个
docker rmi -f 镜像名:tag 镜像名:tag  # 删除多个
docker rmi -f $(docker images -qa) # 删除全部
```

> `容器命令`

`说明:有镜像才能创建容器，我们这里使用 centos 的镜像来测试，就是虚拟一个 centos !`

```shell
docker pull centos
```

**新建容器并启动**

```shell
# 命令
docker run [OPTIONS] IMAGE [COMMAND][ARG...]

# 常用参数说明 
--name="Name"  # 给容器指定一个名字
-d             # 后台方式运行容器，并返回容器的id!
-i             # 以交互模式运行容器，通过和 -t 一起使用
-t             # 给容器重新分配一个终端，通常和 -i 一起使用
-P             # 随机端口映射(大写)
-p             # 指定端口映射(小写)，一般可以有四种写法
ip:hostPort:containerPort 
ip::containerPort 
hostPort:containerPort (常用) 
containerPort

# 使用centos进行用交互模式启动容器，在容器内执行/bin/bash命令!
[root@wl ~]# docker run -it centos /bin/bash 
[root@wl /]# ls # 注意地址，已经切换到容器内部了!
[root@wl /]# exit # 使用 exit 退出容器
```

**列出所有运行的容器**

```shell
# 命令
docker ps [OPTIONS]
# 常用参数说明
-a # 列出当前所有正在运行的容器 + 历史运行过的容器 
-l # 显示最近创建的容器
-n=? # 显示最近n个创建的容器
-q # 静默模式，只显示容器编号。
```

**退出容器**

```shell
exit # 容器停止退出 
ctrl+P+Q # 容器不停止退出
```

**启动停止容器**

```shell
docker start (容器id or 容器名)    # 启动容器
docker restart (容器id or 容器名)  # 重启容器
docker stop (容器id or 容器名)     # 停止容器
docker kill (容器id or 容器名)     # 强制停止容器
```

**删除容器**

```shell
docker rm 容器id # 删除指定容器 
docker rm -f $(docker ps -a -q) # 删除所有容器 
docker ps -a -q|xargs docker rm # 删除所有容器
```

**常用其他命令**

```shell
# 命令
docker run -d 容器名
# 例子
docker run -d centos # 启动centos，使用后台方式启动
# 问题: 使用docker ps 查看，发现容器已经退出了!
# 解释:Docker容器后台运行，就必须有一个前台进程，容器运行的命令如果不是那些一直挂起的命 令，就会自动退出。
# 比如，你运行了nginx服务，但是docker前台没有运行应用，这种情况下，容器启动后，会立即自 杀，因为他觉得没有程序了，所以最好的情况是，将你的应用使用前台进程的方式运行启动。
```

**查看日志**

```shell
# 命令
docker logs -f -t --tail 容器id

# 例子:我们启动 centos，并编写一段脚本来测试玩玩!最后查看日志
 
[root@wl ~]# docker run -d centos /bin/sh -c "while true;do echo kuangshen;sleep 1;done"
[root@wl ~]# docker logs -tf --tail 10 c8530dbbe3b4

# -t 显示时间戳
# -f 打印最新的日志
# --tail 数字 显示多少条!
```

**查看容器中运行的进程信息，支持** **ps** **命令参数。**

```shell
# 命令
docker top 容器id
```

**查看容器/镜像的元数据**

```shell
# 命令
docker inspect 容器id

[root@wl ~]# docker inspect c8530dbbe3b4
[
{
        "Id":
"c8530dbbe3b44a0c873f2566442df6543ed653c1319753e34b400efa05f77cf8",
        "Created": "2020-05-11T08:43:45.096892382Z",
        "Path": "/bin/sh",
        "Args": [
"-c",
            "while true;do echo kuangshen;sleep 1;done"
],
# 状态 "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 27437,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-05-11T08:43:45.324474622Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
     // ...........
]
```

**进入正在运行的容器**

```shell
# 命令1
docker exec -it 容器id bashShell
[root@wl ~]# docker exec -it c8530dbbe3b4 /bin/bash 


# 命令2
docker attach 容器id
[root@wl ~]# docker exec -it c8530dbbe3b4 /bin/bash

# 区别
# exec 是在容器中打开新的终端，并且可以启动新的进程 
# attach 直接进入容器启动命令的终端，不会启动新的进程
```

**从容器内拷贝文件到主机上**

```shell
# 命令
docker cp 容器id:容器内路径 目的主机路径

# 容器内执行，创建一个文件测试 
[root@c8530dbbe3b4 /]# cd /home 
[root@c8530dbbe3b4 home]# touch f1 
[root@c8530dbbe3b4 home]# ls
[root@c8530dbbe3b4 home]# exit

# linux复制查看，是否复制成功
[root@wl ~]# docker cp c8530dbbe3b4:/home/f1 /home 
[root@wl ~]# cd /home
[root@wl home]# ls
f1
```

### 小结

​		将上述的命令实用熟练，只能说，你知道了Docker是什么，和怎么简单实用，并且在看下面这个图，你也基本知道是个什么意思了。

{% asset_img 3.png 运行流程图  %}

### 练习

> 使用Docker 安装 nginx

```shell
# 1、搜索镜像
[root@wl ~]# docker search nginx

# 2、拉取镜像
[root@wl ~]# docker pull nginx

# 3、查看镜像
[root@wl ~]# docker images

# 4、启动容器
[root@wl ~]# docker run -d --name mynginx -p 3500:80 nginx

# 5、测试访问
打开浏览器，输入localhost:3500
如果看到 welcome to nginx 
则安装成功

# 6、进入容器查看具体的html文件放在哪里 
[root@wl ~]# docker exec -it mynginx /bin/bash
root@a95d5f2f057f:/# whereis nginx # 寻找nginx
root@a95d5f2f057f:/# cd /usr/share/nginx # nginx 的路径
root@a95d5f2f057f:/usr/share/nginx# ls
root@a95d5f2f057f:/usr/share/nginx# cd html # 首页的位置
root@a95d5f2f057f:/usr/share/nginx/html# cat index.html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
</html>
```

### 最后

> 会使用上述命令其实连入门都算不上。



## Docker镜像讲解



## **容器数据卷**



## **什么是DockerFile**



## **发布镜像**



## **Docker网络讲解**



## **实战:部署一个Redis集群**