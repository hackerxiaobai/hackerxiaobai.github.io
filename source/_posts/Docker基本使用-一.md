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
[root@2119f4f23a92 /]# cd dataVolumeContainer2
[root@2119f4f23a92 dataVolumeContainer2]# ls -l
total 0
-rw-r--r-- 1 root root 0 May 11 13:20 docker01.txt
-rw-r--r-- 1 root root 0 May 11 13:22 docker02.txt
-rw-r--r-- 1 root root 0 May 11 13:29 docker02-update.txt
-rw-r--r-- 1 root root 0 May 11 13:32 docker03-update.txt
-rw-r--r-- 1 root root 0 May 11 13:24 docker03.txt
# 查看当前运行的容器
[root@wl docker-test-volume]# docker ps
CONTAINER ID
2119f4f23a92
95164598b306
IMAGE
wl/centos
wl/centos
NAMES
docker04
docker03
# 继续删除docker03
[root@wl docker-test-volume]# docker rm -f docker03 docker03
[root@wl docker-test-volume]# docker attach docker04 [root@2119f4f23a92 dataVolumeContainer2]# ls -l
total 0
-rw-r--r-- 1 root root 0 May 11 13:20 docker01.txt -rw-r--r-- 1 root root 0 May 11 13:22 docker02.txt -rw-r--r-- 1 root root 0 May 11 13:29 docker02-update.txt -rw-r--r-- 1 root root 0 May 11 13:32 docker03-update.txt -rw-r--r-- 1 root root 0 May 11 13:24 docker03.txt
```



**结论:**

> 容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止。
> 存储在本机的文件则会一直保留!

## **什么是DockerFile**

> 到目前为止，我们已经知道了一种生成镜像的方式了，就是上述的 Commit 命令
>
> 但是我们在很多项目中看到都有一个DockerFile的东西，接下来就介绍一下这个东西吧。

**DockerFile 是用来构建Docker镜像的构建文件，是由一些列命令和参数构成的脚本。**

**流程:**

+ 开发应用=>DockerFile=>打包为镜像=>上传到仓库(私有仓库，公有仓库)=> 下载镜像 => 启动 运行。

**构建步骤:**

1. 编写DockerFile文件
2. docker build 构建镜像
3. docker run

### **DockerFile构建过程**

**基础知识:**

1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数 
2. 指令按照从上到下，顺序执行
3. "#" 表示注释 
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交 

**流程:**

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器做出修改
3. 执行类似 docker commit 的操作提交一个新的镜像层 
4. Docker再基于刚提交的镜像运行一个新容器 
5. 执行dockerfile中的下一条指令直到所有指令都执行完成!

**说明:**

从应用软件的角度来看，DockerFile，docker镜像与docker容器分别代表软件的三个不同阶段。

+ DockerFile 是软件的原材料 (代码)
+ Docker 镜像则是软件的交付品 (.apk)
+ Docker 容器则是软件的运行状态 (客户下载安装执行)

DockerFile 面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可!

### **DockerFile指令**

**关键字:**

```shell
FROM # 基础镜像，当前新镜像是基于哪个镜像的 MAINTAINER # 镜像维护者的姓名混合邮箱地址
RUN # 容器构建时需要运行的命令
EXPOSE # 当前容器对外保留出的端口
WORKDIR # 指定在创建容器后，终端默认登录的进来工作目录，一个落脚点
ENV # 用来在构建镜像过程中设置环境变量
ADD # 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包 
COPY # 类似ADD，拷贝文件和目录到镜像中!
VOLUME # 容器数据卷，用于数据保存和持久化工作
CMD # 指定一个容器启动时要运行的命令，dockerFile中可以有多个CMD指令，但只有最 后一个生效!
ENTRYPOINT # 指定一个容器启动时要运行的命令!和CMD一样 会有一点点区别
ONBUILD # 当构建一个被继承的DockerFile时运行命令，父镜像在被子镜像继承后，父镜像的 ONBUILD被触发
```

### **实战测试**

> 自定义一个centos

**1、编写DockerFile**

**我们拉去官方的centos 进入容器会发现 vim 和 ifconfig 指令都用不了，不方便**

+ 准备编写DockerFlie文件

```shell
[root@wl home]# mkdir dockerfile-test
[root@wl home]# vim mydockerfile-centos # 编辑文件 
[root@wl home]# cat mydockerfile-centos
FROM centos
MAINTAINER wl<24736743@qq.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN yum -y install vim
RUN yum -y install net-tools
EXPOSE 80
CMD echo $MYPATH
CMD echo "----------end--------"
CMD /bin/bash
```

**2、构建**

`docker build -f dockerfile地址 -t 新镜像名字:TAG .`

```shell
# 查看本地机器上是否有刚构建的镜像
docker images
```

**3、运行**

`docker run -it 新镜像名字:TAG`

可以发现，我们自己的新镜像已经支持 vim/ifconfig的命令，扩展OK!

**4、列出镜像地的变更历史**

`docker history 镜像名`



> CMD 和 ENTRYPOINT 的区别

我们之前说过，两个命令都是指定一个容器启动时要运行的命令

**CMD:**Dockerfile 中可以有多个CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数 替换!

**ENTRYPOINT:** docker run 之后的参数会被当做参数传递给 ENTRYPOINT，之后形成新的命令组合! 

> 自定义镜像 tomcat，直接看DockerFile文件的编写吧

```shell
# vim Dockerfile
FROM centos
MAINTAINER wl<24736743@qq.com> 
#把宿主机当前上下文的read.txt拷贝到容器/usr/local/路径下
COPY read.txt /usr/local/cincontainer.txt
#把java与tomcat添加到容器中
ADD jdk-8u11-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.22.tar.gz /usr/local/
#安装vim编辑器
RUN yum -y install vim
#设置工作访问时候的WORKDIR路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH
#配置java与tomcat环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_11
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.22
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin #容器运行时监听的端口
EXPOSE 8080
#启动时运行tomcat 三选一
# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.22/bin/startup.sh" ]
# CMD ["/usr/local/apache-tomcat-9.0.22/bin/catalina.sh","run"] 
# CMD /usr/local/apache-tomcat-9.0.22/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.22/bin/logs/catalina.out
```

### **总结**

> 至此，我认为我们应该算是Docker 简单入门了！！！！！

## **Docker网络讲解**

### **理解Docker0**

```shell
# 准备工作:清空所有的容器，清空所有的镜像

docker rm -f $(docker ps -a -q) # 删除所有容器 
docker rmi -f $(docker images -qa) # 删除全部镜像
```

> 我们先来做个测试

查看本地ip `ip addr`  **注意看docker0**

```shell
lo 127.0.0.1 # 本机回环地址 
eth0 172.17.90.138 # 私有IP 
docker0 172.18.0.1 # docker网桥
# 问题:Docker 是如何处理容器网络访问的?
```

我们思考这样一个问题，开发了很多微服务项目，那些微服务项目都要连接数据库，需要指定数据库的url地 址，通过ip。但是我们用Docker管理的话，假设数据库出问题了，我们重新启动运行一个，这个时候数 据库的地址就会发生变化，docker会给每个容器都分配一个ip，且容器和容器之间是可以互相访问的。 我们可以测试下容器之间能不能ping通过:

```shell
# 启动tomcat01
[root@wl ~]# docker run -d -P --name tomcat01 tomcat

# 查看tomcat01的ip地址，docker会给每个容器都分配一个ip!
[root@wl ~]# docker exec -it tomcat01 ip addr

# 思考，我们的linux服务器是否可以ping通容器内的tomcat ?
[root@kuangshen ~]# ping 172.18.0.2
# 可以ping 通
```

> 思考，问什么可以ping通呢？

1、每一个安装了Docker的linux主机都有一个docker0的虚拟网卡。这是个桥接网卡，使用了veth-pair 技术!

```shell
# 我们再次查看主机的 ip addr
[root@wl ~]# ip addr

docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state
UP group default
    link/ether 02:42:bb:71:07:06 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker0
valid_lft forever preferred_lft forever

123: vethc8584ea@if122: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc
noqueue master docker0 state UP group default
link/ether 0a:4b:bb:40:78:a7 brd ff:ff:ff:ff:ff:ff link-netnsid 0 
# 发现:本来我们有三个网络，我们在启动了个tomcat容器之后，多了一个!123的网络!
```

2、每启动一个容器，linux主机就会多了一个虚拟网卡。

```shell
# 我们启动了一个tomcat01，主机的ip地址多了一个   123: vethc8584ea@if122 
# 然后我们在tomcat01容器中查看容器的ip是        122: eth0@if123
# 我们再启动一个tomcat02观察
[root@wl ~]# docker run -d -P --name tomcat02 tomcat
# 然后发现linux主机上又多了一个网卡 125: veth021eeea@if124: 

[root@wl ~]# docker exec -it tomcat02 ip addr
# 我们看下tomcat02的容器内ip地址是 124: eth0@if125: 


# 观察现象:
# tomcat --- linux主机 vethc8584ea@if122 ---- 容器内 eth0@if123 
# tomcat --- linux主机 veth021eeea@if124 ---- 容器内 eth0@if125 
# 相信到了这里，大家应该能看出点小猫腻了吧!只要启动一个容器，就有一对网卡
# veth-pair 就是一对的虚拟设备接口，它都是成对出现的。一端连着协议栈，一端彼此相连着。
# 正因为有这个特性，它常常充当着一个桥梁，连接着各种虚拟网络设备!
# “Bridge、OVS 之间的连接”，“Docker 容器之间的连接” 等等，以此构建出非常复杂的虚拟网络 结构，比如 OpenStack Neutron。
```

3、我们来测试下tomcat01和tomcat02容器间是否可以互相ping通

```shell
[root@wl ~]# docker exec -it tomcat02 ping 172.18.0.2
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.110 ms
# 结论:容器和容器之间是可以互相访问的。
```

4、我们来画一个网络模型图

{% asset_img 9.png Docker结构图  %}

**结论:**

tomcat1和tomcat2共用一个路由器。是的，他们使用的一个，就是docker0。任何一个容器启动 默认都是docker0网络。
docker默认会给容器分配一个可用ip。

### **--Link**

思考一个场景，我们编写一个微服务，数据库连接地址原来是使用ip的，如果ip变化就不行了，那我们 能不能使用服务名访问呢?

jdbc:mysql://mysql:3306，这样的话哪怕mysql重启，我们也不需要修改配置了!

`docker提供了 --link 的操作!`

```shell
# 我们使用tomcat02，直接通过容器名ping tomcat01，不使用ip 
[root@wl ~]# docker exec -it tomcat02 ping tomcat01 
ping: tomcat01: Name or service not known 
# 发现ping不通

# 我们再启动一个tomcat03，但是启动的时候连接tomcat02
[root@wl ~]# docker run -d -P --name tomcat03 --link tomcat02 tomcat

# 这个时候，我们就可以使用tomcat03 ping通tomcat02 了
[root@wl ~]# docker exec -it tomcat03 ping tomcat02
PING tomcat02 (172.18.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.18.0.3): icmp_seq=1 ttl=64 time=0.093 ms 64 bytes from tomcat02 (172.18.0.3): icmp_seq=2 ttl=64 time=0.066 ms

# 再来测试，tomcat03 是否可以ping tomcat01 失败 
[root@wl ~]# docker exec -it tomcat03 ping tomcat01 
ping: tomcat01: Name or service not known

# 再来测试，tomcat02 是否可以ping tomcat03 反向也ping不通 
[root@wl ~]# docker exec -it tomcat02 ping tomcat03 
ping: tomcat03: Name or service not known
```

思考，这个原理是什么呢?我们进入tomcat03中查看下host配置文件

```shell
[root@wl ~]# docker exec -it tomcat03 cat /etc/hosts
127.0.0.1   localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.18.0.3 tomcat02 b80da266a3ad # 发现tomcat2直接被写在这里 172.18.0.4 a3a4a17a2b70
# 所以这里其实就是配置了一个 hosts 地址而已!
# 原因:--link的时候，直接把需要link的主机的域名和ip直接配置到了hosts文件中了。
```

`--link早都过时了，我们不推荐使用!我们可以使用自定义网络的方式`

### **自定义网络**

```shell
# 命令查看
docker network --help

# 列出所有网络
docker network ls

# 查看一个具体的网络的详细信息
# 命令
[root@wl ~]# docker network inspect 4eb2182ac4b2
```

> 自定义网卡

1. 我们来创建容器，但是我们知道默认创建的容器都是docker0网卡的

```shell
# 默认我们不配置网络，也就相当于默认值 --net bridge 使用的docker0 
docker run -d -P --name tomcat01 --net bridge tomcat

# docker0网络的特点 
1.它是默认的
2.域名访问不通
3.--link 域名通了，但是删了又不行
```

2. 我们可以让容器创建的时候使用自定义网络

```shell
# 查看如何创建网络
docker network create --help

# 自定义创建的默认default "bridge"
# 自定义创建一个网络网络
[root@wl ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

# 确认下
[root@wl ~]# docker network ls

[root@wl ~]# docker network inspect mynet

# 我们来启动两个容器测试，使用自己的 mynet!
[root@wl ~]# docker run -d -P --name tomcat-net-01 --net mynet tomcat
[root@wl ~]# docker run -d -P --name tomcat-net-02 --net mynet tomcat

# 我们来测试ping容器名和ip试试，都可以ping通
[root@wl ~]# docker exec -it tomcat-net-01 ping 192.168.0.3
[root@wl ~]# docker exec -it tomcat-net-01 ping tomcat-net-02

# 发现，我们自定义的网络docker都已经帮我们维护好了对应的关系
# 所以我们平时都可以这样使用网络，不使用--link效果一样，所有东西实时维护好，直接域名 ping 通。
```

> 思考：上述其实就相当于两个容器同属于一个局域网，当然可以连通了，但是现在有两个不同的网络，这样该怎么相互连通呢？

docker0和自定义网络肯定不通，我们使用自定义网络的好处就是网络隔离:

大家公司项目部署的业务都非常多，假设我们有一个商城，我们会有订单业务(操作不同数据)，会有 订单业务购物车业务(操作不同缓存)。如果在一个网络下，有的程序猿的恶意代码就不能防止了，所 以我们就在部署的时候网络隔离，创建两个桥接网卡，比如订单业务(里面的数据库，redis，mq，全 部业务 都在order-net网络下)其他业务在其他网络。

那关键的问题来了，如何让 tomcat-net-01 访问 tomcat1?

```shell
# 启动默认的容器，在docker0网络下
[root@wl ~]# docker run -d -P --name tomcat01 tomcat
[root@wl ~]# docker run -d -P --name tomcat02 tomcat

# 我们来查看下network帮助，发现一个命令 connect 
[root@wl ~]# docker network --help

# 我们来测试一下!打通mynet-docker0
# 命令 docker network connect [OPTIONS] NETWORK CONTAINER

[root@wl ~]# docker network connect mynet tomcat01
[root@wl ~]# docker network inspect mynet

# tomcat01 可以ping通了
[root@wl ~]# docker exec -it tomcat01 ping tomcat-net-01

# tomcat02 依旧ping不通，大家应该就理解了
[root@wl ~]# docker exec -it tomcat02 ping tomcat-net-01
ping: tomcat-net-01: Name or service not known
```

**结论**

> 如果要跨网络操作别人，就需要使用
>
> `docker network connect [OPTIONS] NETWOEK CONTAINER`

## **实战:部署一个Redis集群**