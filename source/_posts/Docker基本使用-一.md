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

### **镜像是什么**

```
镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含
运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。
```

### **Docker镜像加载原理**

+ UnionFS (联合文件系统)
  + Docker 镜像的基础
  + Union文件系统(UnionFS)是一种分层、轻量级并且高性能的文件系统， 它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系 统下
+ Docker镜像加载原理
  + docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

### **分层理解**

`我们可以去下载一个镜像，注意观察下载的日志输出，可以看到是一层一层的在下载!`

{% asset_img 4.png Docker结构图  %}

`Docker镜像采用这种分层结构，最大的好处莫过于资源共享了`

```shell
# 查看镜像分层的方式可以通过 docker image inspect ID
```

> **重点**

​		所有的 Docker 镜像都起始于一个基础镜像层，当进行修改或增加新的内容时，就会在当前镜像层之 上，创建新的镜像层。

​		举一个简单的例子，假如基于 Ubuntu Linux 16.04 创建一个新的镜像，这就是新镜像的第一层;如果 在该镜像中添加 Python包，就会在基础镜像层之上创建第二个镜像层;如果继续添加一个安全补丁，就 会创建第三个镜像层。

​		该镜像当前已经包含 3 个镜像层，如下图所示。

{% asset_img 5.png Docker结构图  %}

> 注意

​		Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部! 这一层就是我们通常说的容器层，容器之下的都叫镜像层!容器层是可写的。

### **镜像Commit**

**docker commit** **从容器创建一个新的镜像。**

```shell
docker commit 提交容器副本使之成为一个新的镜像!
# 语法
docker commit -m="提交的描述信息" -a="作者" 容器id 要创建的目标镜像名:[标签名]
```

**测试**

比如我们自己需要制作一个镜像，以ubuntu16.04 为基础镜像，在该镜像上安装深度学习库，比如要安装python 和 pytorch

我们就可以按上述说的方式进行操作

1. 拉去ubuntu 16.04基础镜像
2. 启动镜像，生成一个容器，进入该容器
3. 安装python，pytorch 安装包
4. 退出
5. docker commit 

## **容器数据卷**

### **什么是容器数据卷**

**docker的理念回顾:**

​		将应用和运行的环境打包形成容器运行，运行可以伴随着容器，但是我们对于数据的要求，是希望能够持久化的! 就好比，你安装一个MySQL，结果你把容器删了，就相当于删库跑路了，这TM也太扯了吧!

​		所以我们希望容器之间有可能可以共享数据，Docker容器产生的数据，如果不通过docker commit 生成 新的镜像，使得数据作为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了!这样是行 不通的!

​		为了能保存数据在Docker中我们就可以使用卷!让数据挂载到我们本地!这样数据就不会因为容器删除 而丢失了!

**作用:**

​		卷就是目录或者文件，存在一个或者多个容器中，由docker挂载到容器，但不属于联合文件系统，因此 能够绕过 Union File System ， 提供一些用于持续存储或共享数据的特性:

​		卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂 载的数据卷。

**特点:**

1. 数据卷可在容器之间共享或重用数据
2. 卷中的更改可以直接生效 
3. 数据卷中的更改不会包含在镜像的更新中 
4. 数据卷的生命周期一直持续到没有容器使用它为止

**所以:总结一句话: 就是容器的持久化，以及容器间的继承和数据共享!**

### **使用数据卷**

> 方式一：容器中直接使用命令来添加

```shell
# 命令
docker run -it -v 宿主机绝对路径目录:容器内目录 镜像名
# 测试
[root@wl ~]# docker run -it -v /home/ceshi:/home centos /bin/bash
```

查看数据卷是否挂载成功 `docker inspect 容器id`

{% asset_img 6.png Docker结构图  %}

**测试容器和宿主机之间数据共享:可以发现，在容器中创建的文件会在宿主机对应目录上看到!**

```shell
测试容器停止退出后，主机修改数据是否会同步!
```

1. 停止容器
2. 在宿主机上修改文件，增加些内容
3.  启动刚才停止的容器
4.  然后查看对应的文件，发现数据依旧同步!ok

> 方式二：通过Docker File 来添加(了解)

测试:

```shell
# 1、我们在宿主机 /home 目录下新建一个 docker-test-volume文件夹 
[root@wl home]# mkdir docker-test-volume

# 说明:在编写DockerFile文件中使用 VOLUME 指令来给镜像添加一个或多个数据卷 VOLUME["/dataVolumeContainer1","/dataVolumeContainer2","/dataVolumeContainer 3"]
# 出于可移植和分享的考虑，我们之前使用的 -v 主机目录:容器目录 这种方式不能够直接在 DockerFile中实现。
# 由于宿主机目录是依赖于特定宿主机的，并不能够保证在所有宿主机上都存在这样的特定目录.

# 2、编写DockerFile文件
[root@wl docker-test-volume]# pwd /home/docker-test-volume
[root@wl docker-test-volume]# vim dockerfile1 
[root@wl docker-test-volume]# cat dockerfile1
# volume test
FROM centos
VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"] CMD echo "-------end------"
CMD /bin/bash

# 3、build后生成镜像，获得一个新镜像 kuangshen/centos
docker build -f /home/docker-test-volume/dockerfile1 -t kuangshen/centos . # 注意最后有个.

# 4、启动容器
[root@wl docker-test-volume]# docker run -it 0e97e1891a3d /bin/bash # 启动容器
[root@f5824970eefc /]# ls -l  # 可以看到对应的两个数据卷目录

# 问题:通过上述步骤，容器内的卷目录地址就已经知道了，但是对应的主机目录地址在哪里呢?

# 5、我们在数据卷中新建一个文件
[root@f5824970eefc dataVolumeContainer1]# pwd /dataVolumeContainer1
[root@f5824970eefc dataVolumeContainer1]# touch container.txt

# 6、查看下这个容器的信息
[root@wl ~]# docker inspect 0e97e1891a3d

# 查看输出的Volumes
"Volumes": {
    "/dataVolumeContainer1": {},
    "/dataVolumeContainer2": {}
},
```

{% asset_img 6.png Docker结构图  %}

### 练习

> 通过Docker 来 安装 mysql

**思考:mysql 数据持久化的问题!**

```shell
# 1、搜索镜像
[root@wl ~]# docker search mysql

# 2、拉取镜像
[root@wl ~]# docker pull mysql:5.7

# 3、启动容器 -e 环境变量!
# 注意: mysql的数据应该不丢失!先体验下 -v 挂载卷! 参考官方文档
[root@wl home]# docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

# 4、使用本地的mysql可视化工具连接测试一下 端口号 3310

# 5、查看本地的 /home/mysql 目录
# 可以看到我们刚刚建立的mysql数据库在本地存储着

# 6、删除mysql容器
[root@wl data]# docker rm -f mysql01 # 删除容器，然后发现远程连接失败!

# 7、在看对我们宿主机上对应目录的文件，依然是存在的，但我们用上述启动容器命令再一次启动的时候，
#    连接数据库，发现该有的数据依然存在
```

### **匿名和具名挂载**

```shell
# 匿名挂载
-v 容器内路径
docker run -d -P --name nginx01 -v /etc/nginx nginx
# 匿名挂载的缺点，就是不好维护，通常使用命令 docker volume维护 docker volume ls

# 具名挂载
-v 卷名:/容器内路径
docker run -d -P --name nginx02 -v nginxconfig:/etc/nginx nginx
# 查看挂载的路径
[root@wl ~]# docker volume inspect nginxconfig [
    {
        "CreatedAt": "2020-05-13T17:23:00+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/nginxconfig/_data",
        "Name": "nginxconfig",
        "Options": null,
        "Scope": "local"
} ]


# 改变文件的读写权限
# ro: readonly
# rw: readwrite
# 指定容器对我们挂载出来的内容的读写权限
docker run -d -P --name nginx02 -v nginxconfig:/etc/nginx:ro nginx docker run -d -P --name nginx02 -v nginxconfig:/etc/nginx:rw nginx
```

### **数据卷容器**

> 如何让多个容器数据共享

​		命名的容器挂载数据卷，其他容器通过挂载这个(父容器)实现数据共享，挂载数据卷的容器，称之为
数据卷容器。

​		我们使用上一步的镜像:wl/centos 为模板，运行容器 docker01，docker02，docker03，他们都会具有容器卷

1、先启动一个父容器docker01，然后在dataVolumeContainer2新增文件

{% asset_img 7.png Docker结构图  %}

2、创建docker02，docker03 让他们继承docker01 --volumes-from

```shell
[root@wl docker-test-volume]# docker run -it --name docker02 --volumes-from docker01 wl/centos
[root@95164598b306 dataVolumeContainer2]# touch docker02.txt

[root@wl docker-test-volume]# docker run -it --name docker03 --volumes-from docker01 kuangshen/centos
[root@95164598b306 dataVolumeContainer2]# touch docker03.txt
```

3、回到docker01发现可以看到 02 和 03 添加的共享文件

```shell
[root@wl docker-test-volume]# docker attach docker01
[root@799b6ea5db7c dataVolumeContainer2]# ls -l
total 0
-rw-r--r-- 1 root root 0 May 11 13:20 docker01.txt
-rw-r--r-- 1 root root 0 May 11 13:22 docker02.txt
-rw-r--r-- 1 root root 0 May 11 13:24 docker03.txt
```

4、删除docker01，docker02 修改后docker03还能不能访问

```shell
[root@wl docker-test-volume]# docker rm -f docker01
docker01

[root@wl docker-test-volume]# docker attach docker02
[root@ea4c82779077 dataVolumeContainer2]# ls -l
total 0
-rw-r--r-- 1 root root 0 May 11 13:20 docker01.txt
-rw-r--r-- 1 root root 0 May 11 13:22 docker02.txt
-rw-r--r-- 1 root root 0 May 11 13:24 docker03.txt 

[root@ea4c82779077 dataVolumeContainer2]# touch docker02-update.txt 
[root@ea4c82779077 dataVolumeContainer2]# ls -a
. .. docker01.txt docker02.txt docker02-update.txt docker03.txt 

[root@ea4c82779077 dataVolumeContainer2]# Ctrl+P+Q 退出容器
[root@kuangshen docker-test-volume]# docker attach docker03
[root@95164598b306 dataVolumeContainer2]# ls -l
total 0
-rw-r--r-- 1 root root 0 May 11 13:20 docker01.txt
-rw-r--r-- 1 root root 0 May 11 13:22 docker02.txt
-rw-r--r-- 1 root root 0 May 11 13:29 docker02-update.txt
-rw-r--r-- 1 root root 0 May 11 13:24 docker03.txt
```

5、删除docker02 ，docker03还能不能访问

```shell
[root@wl docker-test-volume]# docker ps
CONTAINER ID
95164598b306
ea4c82779077
[root@wl docker-test-volume]# docker rm -f docker02
docker02
[root@wl docker-test-volume]# docker attach docker03
[root@95164598b306 dataVolumeContainer2]# ls -l
total 0
-rw-r--r-- 1 root root 0 May 11 13:20 docker01.txt
-rw-r--r-- 1 root root 0 May 11 13:22 docker02.txt
-rw-r--r-- 1 root root 0 May 11 13:29 docker02-update.txt
-rw-r--r-- 1 root root 0 May 11 13:24 docker03.txt
[root@95164598b306 dataVolumeContainer2]# touch docker03-update.txt
```

6、新建docker04继承docker03，然后再删除docker03，看下是否可以访问!

{% asset_img 8.png Docker结构图  %}

```shell

```



**结论:**

> 容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止。
> 存储在本机的文件则会一直保留!

## **什么是DockerFile**

> 到目前为止，我们已经知道了一种生成镜像的方式了，就是上述的 Commit 命令
>
> 但是我们在很多项目中看到都有一个DockerFile的东西，接下来就介绍一下这个东西吧。

**DockerFile 是用来构建Docker镜像的构建文件，是由一些列命令和参数构成的脚本。**



## **发布镜像**



## **Docker网络讲解**



## **实战:部署一个Redis集群**