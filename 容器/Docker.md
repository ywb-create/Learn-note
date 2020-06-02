# Docker

## 概述

#### 介绍

​	Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中,然后发布到任何流行的Linux机器或Windows 机器上,也可以实现虚拟化,容器是完全使用沙箱机制,相互之间不会有任何接口。

#### 架构

​	Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。Docker 容器通过 Docker 镜像来创建。容器与镜像的关系类似于面向对象编程中的对象与类。

| Docker | 面向对象 |
| ------ | -------- |
| 镜像   | 类       |
| 容器   | 对象     |

​	Docker采用 C/S架构 Docker daemon 作为服务端接受来自客户的请求，并处理这些请求（创建、运行、分发容器）。 客户端和服务端既可以运行在一个机器上，也可通过 socket 或者RESTful API 来进行通信。

​	Docker daemon 一般在宿主主机后台运行，等待接收来自客户端的消息。 Docker 客户端则为用户提供一系列可执行命令，用户用这些命令实现跟 Docker daemon 交互

![image-20200601114411186](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601114411186.png)

镜像（image)：

​	docker镜像就好比是一个目标，可以通过这个目标来创建容器服务，tomcat镜像-->run-->容器（提供服务器），通过这个镜像可以创建多个容器（最终服务运行或者项目运行就是在容器中的）。

容器(container)：

Docker利用容器技术，独立运行一个或者一组应用，通过镜像来创建的.
启动，停止，删除，基本命令
目前就可以把这个容器理解为就是一个简易的 Linux系统。

仓库(repository)：

仓库就是存放镜像的地方！
仓库分为公有仓库和私有仓库。(很类似git)
Docker Hub是国外的。
阿里云…都有容器服务器(配置镜像加速!)

#### 容器和虚拟机

​	容器时在linux上本机运行，并与其他容器共享主机的内核，它运行的一个独立的进程，不占用其他任何可执行文件的内存，非常轻量。

​	虚拟机运行的是一个完成的操作系统，通过虚拟机管理程序对主机资源进行虚拟访问，相比之下需要的资源更多。

![image-20200601120407571](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601120407571.png)

## 安装

#### linux安装

```shell
#1.卸载旧版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
#2.需要的安装包
yum install -y yum-utils
#3.设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
#默认是从国外的，不推荐
#推荐使用国内的
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#更新yum软件包索引
yum makecache fast
#4.安装docker相关的 docker-ce 社区版 而ee是企业版
yum install docker-ce docker-ce-cli containerd.io
#6. 使用docker version查看是否按照成功
docker version
#7. 测试
docker run hello-world

#8. 卸载docker
	#删除依赖
yum remove docker-ce docker-ce-cli containerd.io
	#删除资源
rm -rf /var/lib/docker# /var/lib/docker 是docker的默认工作路径！
```

#### mac安装

[下载](https://download.docker.com/mac/stable/Docker.dmg)		https://download.docker.com/mac/stable/Docker.dmg

下载后解压拖入Application

下载后打开preferences-->Docker Engine 在配置中添加阿里云镜像

```json
"registry-mirrors": [
    "https://kj6th76.mirror.aliyuncs.com"
  ]
```

阿里云镜像可在阿里云官网中https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors配置

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601120203461.png" alt="image-20200601120203461" style="zoom:50%;" />

## 常用命令

#### 帮助命令

```shell
docker version
docker info
docker 命令 --help
```

#### 镜像命令

##### 查看镜像

```shell
docker images #查看本地主机上的所有镜像
Usage:	docker images [OPTIONS] [REPOSITORY[:TAG]]

List images

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
```

##### 搜索镜像

```shell
docker search --help

Usage:	docker search [OPTIONS] TERM

Search the Docker Hub for images

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output
      
#测试
$ docker search centos
NAME                DESCRIPTION                           STARS               OFFICIAL           AUTOMATED
centos            The official build of CentOS.           6020                [OK]                
```

##### 下载镜像

```shell
docker pull centos

Usage:	docker pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
  -q, --quiet                   Suppress verbose output
  
#测试
$ docker pull mysql:5.7#指定版本下载
```

##### 删除镜像

```shell
docker rmi --help

Usage:	docker rmi [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
      
#测试
$ docker rmi -f bf756fb1ae65
$ docker rmi -f $(docker images -aq) #删除全部镜像
```

#### 容器命令

##### 新建容器并启动

```shell
docker run [参数] image

#参数
--name="name" 给容器命名来区分容器
-d 						后台启动
-it						使用交互方式运行
-p						指定容器端口
-P						随机指定端口

#测试
$ docker run -it centos
```

##### 从容器内退回主机

```shell
exit		#退回主机并关闭容器

ctrl+q+p#仅退回主机
```

##### 列出所有容器

```shell
docker ps --help

Usage:	docker ps [OPTIONS]

List containers

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display numeric IDs
  -s, --size            Display total file sizes
```

##### 删除容器

```shell
docker rm 容器id
$ docker rmi -f $(docker images -aq) #删除全部容器
```

##### 启动和停止当前容器

```shell
docker start 容器id  		#启动容器
docker restart 容器id 	#重启容器
docker stop 容器id  		#停止容器
docker kill 容器id 			#强制停止容器
```

#### 其他命令

##### 后台启动

```shell
docker run -d centos
```

`docker ps`发现centos停止了，docker容器后台运行时，必须有一个前台进程（docker run -it centos），否则会自动停止。也就是说，容器后台启动后，发现自己没有提供服务就会自杀

##### 查看日志

```shell
docker logs
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m for 42 minutes)
      --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37) or relative (e.g. 42m for 42 minutes)
      
#测试
$ docker logs -tf --tail 10 f7aaddf38ff5
```

##### 查看容器中进程信息

```shell
docker top
Usage:	docker top CONTAINER [ps OPTIONS]

Display the running processes of a container

#测试
$ docker top f7aaddf38ff5
PID                 USER                TIME                COMMAND
2354                root                0:00                /bin/bash
```

##### 查看镜像的元数据

```shell
docker inspect 

Usage:	docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Return low-level information on Docker objects

Options:
  -f, --format string   Format the output using the given Go template
  -s, --size            Display total file sizes if the type is container
      --type string     Return JSON for specified type
      
#测试     
$ docker inspect f7aaddf38ff5
```

##### 进入当前正在运行的容器

###### 方式1

```shell
docker exec --help

Usage:	docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container
  
#测试
$ docker exec -it f7aaddf38ff5 /bin/bash
```

###### 方式2

```shell
docker attach --help

Usage:	docker attach [OPTIONS] CONTAINER

Attach local standard input, output, and error streams to a running container

Options:
      --detach-keys string   Override the key sequence for detaching a container
      --no-stdin             Do not attach STDIN
      --sig-proxy            Proxy all received signals to the process (default true)
      
#测试
$ docker attach f7aaddf38ff5
```

###### 区别

```shell
docker exec   #进入容器后开启一个新的终端
docker attach #进入容器正在执行的终端，不会启动新的进程
```

##### 从容器内拷贝文件到主机

```shell
docker cp --help

Usage:	docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
	docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

Copy files/folders between a container and the local filesystem

Use '-' as the source to read a tar archive from stdin
and extract it to a directory destination in a container.
Use '-' as the destination to stream a tar archive of a
container source to stdout.

Options:
  -a, --archive       Archive mode (copy all uid/gid information)
  -L, --follow-link   Always follow symbol link in SRC_PATH

#测试
$docker cp f7aaddf38ff5:/home/a.java /Users/ywb/a
docker cp 容器id：容器内地址 目的主机地址
```

#### 练习

##### 1.nginx

```shell
#下载nginx
$ docker pull nginx
#运行nginx（3344映射内部的80端口，外部可直接通过访问3344端口访问到nginx服务器）
$ docker run -d --name nginx01 -p 3344:80 nginx
#测试
$ curl localhost:3344
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
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
#进入容器
docker exec -it nginx01 /bin/bash
root@2df7c4edfd3e:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@2df7c4edfd3e:/# cd /etc/nginx/
root@2df7c4edfd3e:/etc/nginx# ls
conf.d	fastcgi_params	koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params	uwsgi_params  win-utf
```

##### 2.tomcat

```shell
#服务停止后直接删除容器（练习时使用）
docker run -it --rm tomcat:9.0

$ docker run -d --name tomcat01 -p 3344:8080 tomcat
#启动后访问时404 是阿里云把webapp下的文件删除了，可以进入容器内复制cp -r webapps.dist/* webapps，复制后在访问就可以看到欢迎页面

```

##### 3.elasticsearch

```shell
$ docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.7.0

#可以用docker stats查看服务器的资源消耗
$ docker stats
CONTAINER ID        NAME          CPU %      MEM USAGE / LIMIT     MEM %     NET I/O      BLOCK I/O    PIDS
0523cd1d4cc3    elasticsearch    0.81%      1.235GiB / 1.945GiB   63.48%  2.64kB / 2.59kB   0B / 0B     53

#es非常耗内存（MEM %：63.48%），增加内存限制
$ docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.7.0

```

#### 小结

![image-20200531184306729](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200531184306729.png)

```shell
attach	Attach to a running container#当前she11下attach 连接指定运行镜像
build		Build an image from a Dockerfile#通过Dockerfile定制镜像
commit	Create a new image from a container changes#提交当前容器为新的镜像
cp			Copy files/folders from the containers fi lesystem to the host path#从容器中拷贝指定文件或者目录到宿主机中
create	Create a new container#创建一个新的容器，同run， 但不启动容器
diff		Inspect changes on a containers filesys tem #查看docker 容器变化
events	Get rea1 time events from the server#从docker服务获取容器实时事件
exec		Run a command in an existing container#在已存在的容器上运行命令
export	Stream the contents of a container as a tar archive#导出容器的内容流作为一个tar 归档文件[对应import ]
history	Show the history of an image#展示一个镜像形成历史
images	List images#列出系统当前镜像
import	Create a new filesystem image from the contents of a tarbal1 #从tar包中的内容创建一个新的文件 系统映像[对应export]
info		Display system-wide information#显示系统相关信息
inspect	Return 1ow-1eve1 information on a container#查看容器详细信息
ki11		Ki11 a running container# ki11指定docker容器
1oad 		Load an image from a tar archive#从一个tar包中加载一个镜像[对应save]
login 	Register or Login to the docker registry server#注册或者登陆一个docker源服务器
logout	Log out from a Docker registry server#从当前Docker registry退出
1ogs		Fetch the 1ogs of a container#输出当前容器日志信息
port		Lookup the pubTic- facing port which is NAT-ed to PRIVATE_ PORT#查看映射端口对应的容器内部源端
pause		Pause a11 processes within a container#暂停容器
ps			List containers	#列出容器列表
pu11		Pu11 an image or a repository from the docker registry server#从docker镜像源服务器拉取指定镜像或者库镜像
push		Push an image or a repository to the docker registry server#推送指定镜像或者库镜像至docker源服务器
restart	Restart a running container#重启运行的容器
rm			Remove one or more containers #移除一个或者多个容器
rmi			Remove one or more i mages#移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或-f强制删除]
run			Run a command in a new container#创建一个新的容器并运行一个命令
save		Save an image to a tar archive#保存一个镜像为一个tar包[对应load]
search	Search for an image on the Docker Hub	#在dockerhub中搜索镜像
start		Start a stopped containers#启动容器
stop		Stop a running containers	#停止容器
tag	Tag an image into a repository #给源中镜像打标签
top 		Lookup the running processes of a container#查看容器中运行的进程信息
unpause	Unpause a paused container	#取消暂停容器
version	Show the docker version informati on	#查看docker版本号
wait		B1ock until a container stops， then print its exit code#截取容器停止时的退出状态值
```

## Docker镜像

​	镜像是一种轻量级、 可执行的独立软件包,用来打包软件运行环境和基于运行环境开发的软件,它包含运行某个软件所需的所有内容,包括代码、运行时、库、环境变量和配置文件。

​	所有的应用,直接打包docker镜像,就可以直接跑起来!

#### 联合文件系统

UnionFS (联合文件系统) : Union文件系统( UnionFS)是-种分层、 轻量级并且高性能的文件系统,它支持对文件系统的修改作为一次提交来一层层的叠加 ,同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtualfilesystem)。Union 文件系统是Docker镜像的基础。镜像可以通过分层来进行继承,基于基础镜像(没有父镜像)，可以制作各
种具体的应用镜像。

特性: 一次同时加载多个文件系统,但从外面看起来,只能看到一个文件系统,联合加载会把各层文件系统叠加起来,这样最终的文件系统会包含所有底层的文件和目录

#### 镜像加载原理

docker的镜像实际上由一层一层的文件 系统组成,这种层级的文件系统UnionFS。

bootfs(boot file system)主要包含==bootloader和kernel==, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统,在Docker镜像的**最底层是bootfs**。这-层与我们典型的Linux/Unix系统是一样的,包含boot加载器和内核。当boo加载完成之后整个内核就都在内存中了,此时内存的使用权已由bootfs转交给内核,此时系统也会卸载bootfs。

rootfs (root file system) ,在bootfs之上。包含的就是典型Linux系统中的/dev, /proc, /bin, /etc等标准目录和文件。rootfs就是各种不同的操作系统发行版,比如Ubuntu , Centos等等。

#### 分层原理

下载镜像时，时一层一层的下载

![image-20200601103902486](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601103902486.png)

这样做可以有利于资源共享,多个镜像可以使用同一个base镜像，极大的减少内存和磁盘资源的消耗

可以使用inspect命令查看分层信息

```shell
$ docker image inspect redis:latest
```

![image-20200601104402312](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601104402312.png)

所有的Docker镜像都起始于-个基础镜像层,当进行修改或增加新的内容时,就会在当前镜像层之上,创建新的镜像层。举一个简单的例子,假如基于Ubuntu Linux 16.04创建一个新的镜像 ,这就是新镜像的第一层;如果在该镜像中添加Python包就会在基础镜像层之上创建第二个镜像层;如果继续添加一个安全补丁,就会创建第三个镜像层。

该镜像当前已经包含3个镜像层, 如下图所示

![image-20200601104715417](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601104715417.png)

在添加额外的镜像层的同时，镜像始终保持是当前所有镜像的组合，理解这一点非常重要。下图中举了一个简单的例子，每个镜像层包含3个文件，而镜像包含了来自两个镜像层的6个文件。

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601105856770.png" alt="image-20200601105856770" style="zoom:50%;" />

上图中的镜像层跟之前图中的略有区別，主要目的是便于展示文件
下图中展示了一个稍微复杂的三层镜像，在外部看来整个镜像只有6个文件，这是因为最上层中的文件7是文件5的一个更新版

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601105922184.png" alt="image-20200601105922184" style="zoom:50%;" />

这种情況下，上层镜像层中的文件覆盖了底层镜像层中的文件。这样就使得文件的更新版本作为一个新镜像层添加到镜像当中

Docker通过存储引擎（新版本采用快照机制）的方式来实现镜像层堆栈，并保证多镜像层对外展示为统一的文件系统

Linux上可用的存储引撃有AUFS、 Overlay2、 Device Mapper、Btrfs以及ZFS。顾名思义，每种存储引擎都基于 Linux中对应的件系统或者块设备技术，井且每种存储引擎都有其独有的性能特点。

下图展示了与系统显示相同的三层镜像。所有镜像层堆并合井，对外提供统一的视图

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601110152306.png" alt="image-20200601110152306" style="zoom: 33%;" />

##### 镜像与容器的分层

Docker 镜像都是**只读的**，当容器启动时，一个新的可写层加载到镜像的顶部！

`pull`下来的镜像时不可变的，当`run`后，镜像启动变为容器，容器可以更改配置，相当于容器是在镜像上新增加的一层，此时的容器和镜像可以一起打包为一个新的镜像

#### commit镜像

```shell
docker commit #提交一个镜像成为一个新的副本

docker commit -m="描述信息" -a="作者" 容器id 目标镜像名:[TAG]
#测试
#启动tomcat
$ docker run -it -p 8080:8080 tomcat
#进入tomcat拷贝文件
$ docker exec -it 98a36904a4fa /bin/bash
$ cp -r webapps.dist/* webapps
#提交修改后的tomcat
docker commit -m="add webapps" -a="y" 98a36904a4fa tomcat-y:1.0
#查看镜像
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat-y            1.0                 bcce9a16d998        8 seconds ago       652MB
redis               latest              36304d3b4540        3 days ago          104MB
mysql               5.7                 a4fdfd462add        10 days ago         448MB
mysql               latest              30f937e841c8        10 days ago         541MB
tomcat              9.0                 1b6b1fe7261e        2 weeks ago         647MB
tomcat              latest              1b6b1fe7261e        2 weeks ago         647MB
nginx               latest              9beeba249f3e        2 weeks ago         127MB
elasticsearch       7.7.0               7ec4f35ab452        2 weeks ago         757MB
centos              latest              470671670cac        4 months ago        237MB

```

## 容器数据卷

#### 简介

​	在使用docker容器的时候，会产生一系列的数据文件，这些数据文件在我们关闭docker容器时是会消失的，但是其中产生的部分内容我们是希望能够把它给保存起来另作用途的，Docker将应用与运行环境打包成容器发布，我们希望在运行过程钟产生的部分**数据是可以持久化**的的，而且**容器之间**我们希望能够实现**数据共享**

​	docker容器数据卷就是为了实现这些需求，它可以看成u盘，它存在于一个或多个的容器中，由docker挂载到容器，但不属于联合文件系统，Docker不会在容器删除时删除其挂载的数据卷。



数据卷特点：

- 数据卷可以在容器之间共享或重用数据
- 数据卷中的更改可以直接生效
- 数据卷中的更改不会包含在镜像的更新中
- 数据卷的生命周期一直持续到没有容器使用它为止

#### 使用

##### 命令行挂载

 ```shell
docker run -it -v  /宿主机绝对路径目录:  /容器内目录  镜像名
docker inspect 容器id
#"Mounts": [
#            {
#                "Type": "bind",
#                "Source": "/Users/ywb/test",	挂载具体信息
#                "Destination": "/home",
#                "Mode": "",
#                "RW": true,
#                "Propagation": "rprivate"
#            }
#        ],
#测试
$ docker run -it -v /Users/ywb/test:/home centos /bin/bash

#拓展
# 通过 -v 容器内路径： ro rw 改变读写权限
#	ro readonly 	只读		这个路径只能通过宿主机来操作，容器内部是无法操作
#	rw readwrite 	读写 
docker run -d -P --name nginx -v /Users/ywb/nginx:/etc/nginx:ro nginx
 ```

###### 安装mysql

```shell
# 获取mysql镜像
docker pull mysql:5.7
# 运行容器,需要做数据挂载 #安装启动mysql，需要配置密码的，这是要注意点！
# 参考官网hub  -e 配置mysql密码
# docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

#测试
docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
```

此时，如果将docker中的容器删除，挂载到本地的数据卷仍未丢失，实现了数据的持久化

##### DockerFile挂载

dockerfile：用来构建docker镜像的文件，是一段命令脚本

```shell
#dockerfile文件中内容，通过这个脚本可以生成镜像
FROM  镜像名

VOLUME ["/生成的目录路径"]  -- privileged=true

CMD echo "success build"

CMD /bin/bash
```

编写脚本后构建镜像

```shell
docker build -f  dockerfile的路径 -t  镜像名:tag

#测试
#. 代表当前目录
$ docker build -f dockerfile1 -t y/centos:1.0 .

$ docker run -it y/centos:1.0 /bin/bash
$ ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var	   volume02
dev  home  lib64  media       opt  root  sbin  sys  usr  volume01

# volume01 volume02就是生成镜像时自动挂载的数据卷目录
```

使用`docker inspect 9c5d3b30aa38`查看挂载信息，其中source是挂载的文件对应的卷

```json
"Mounts": [
            {
                "Type": "volume",
                "Name": "db6d99f14ef429a44be7c9a8d44a89ac482b04b3a819d7838067fe36acea9173",
                "Source": "/var/lib/docker/volumes/db6d99f14ef429a44be7c9a8d44a89ac482b04b3a819d7838067fe36acea9173/_data",
                "Destination": "volume01",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "0f807ea1bedc12fa7a515fde5df0390eb53f7908ed8e7d7983dde81bb871d943",
                "Source": "/var/lib/docker/volumes/0f807ea1bedc12fa7a515fde5df0390eb53f7908ed8e7d7983dde81bb871d943/_data",
                "Destination": "volume02",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
```

通过dockerFile中的`VOLUME ["volume01","volume02"]`构建的镜像，挂载的类型属于匿名挂载。

##### 挂载类型

```shell
# 三种挂载： 匿名挂载、具名挂载、指定路径挂载
-v 容器内路径						#匿名挂载
-v 卷名：容器内路径				#具名挂载
-v /宿主机路径：容器内路径 #指定路径挂载 docker volume ls 是查看不到的
```

###### 匿名挂载

```shell
# 匿名挂载	在 -v只写了容器内的路径，没有写容器外的路径
docker run -d -P --name nginx01 -v /etc/nginx nginx

# 查看所有的volume的情况
docker volume ls 
```

###### 具名挂载

```shell
# 具名挂载
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
# 查看所有的volume的情况
docker volume ls  
```



所有的docker容器内的卷没有指定目录的情况下都在`/var/lib/docker/volumes/xxxx/_data`下。如果指定了目录，docker volume ls 是查看不到的

#### 数据卷容器

​	命名的容器挂载数据卷，其他的容器通过挂载这个父容器实现数据共享，挂载数据卷的容器，我们称为数据卷容器

​	如果docker容器数据卷相当于生活中的活动硬盘，那么**数据卷容器**就相当于把多个活动硬盘再挂载到一个活动硬盘上，实现数据的传递。

​	简单来说数据卷容器就是实现**容器间的数据传递**

```shell
$ docker run -it --name centos1 y/centos:1.0
$ docker run -it --name centos2 --columes-from centos1 y/centos:1.0
#测试后发现centos2中的volume01中有centos1的volume01的数据，再删除centos1后，centos2中的数据依旧存在，说明centos2虽是挂载到centos1的，但是数据是互相拷贝的，并不会随着centos1的删除而丢失
```

## DockerFile

#### 介绍

dockerfile就是一段脚本，用于构建docker镜像

构建步骤：

1、 编写一个dockerfile文件

2、 docker build 构建称为一个镜像

3、 docker run运行镜像

4、 docker push发布镜像（DockerHub 、阿里云仓库)

#### 构建过程

1、每个保留关键字(指令）都是必须是大写字母

2、执行从上到下顺序

3、#表示注释

4、每一个指令都会创建提交一个新的镜像并提交！



<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601163027879.png" alt="image-20200601163027879" style="zoom:50%;" />

#### DockerFile指令

```shell
FROM						# 基础镜像，一切从这里开始构建
MAINTAINER			# 镜像是谁写的， 姓名+邮箱
RUN							# 镜像构建的时候需要运行的命令
ADD							#	步骤，tomcat镜像，这个tomcat压缩包！添加内容 添加同目录
WORKDIR					# 镜像的工作目录
VOLUME					# 挂载的目录
EXPOSE					# 保留端口配置
CMD							# 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代。
ENTRYPOINT			# 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD					# 当构建一个被继承 DockerFile 这个时候就会运行ONBUILD的指令，触发指令。
COPY						# 类似ADD，将我们文件拷贝到镜像中
ENV							# 构建的时候设置环境变量！
```

##### CMD与ENTRYPOINT区别

```shell
#CMD
$ vim dockerfile-test-cmd
FROM centos
CMD ["ls","-a"]
# 构建镜像
$ docker build  -f dockerfile-test-cmd -t cmd-test:0.1 .
# 运行镜像
$ docker run cmd-test:0.1
.
..
.dockerenv
bin
dev

# 想追加一个命令  -l 成为ls -al	报错
$ docker run cmd-test:0.1 -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"-l\":
 executable file not found in $PATH": unknown.
ERRO[0000] error waiting for container: context canceled 
# cmd的情况下 -l 替换了CMD["ls","-l"]。 -l  不是命令所有报错
```

```shell
#NTRYPOINT
$ vim dockerfile-test-entrypoint
FROM centos
ENTRYPOINT ["ls","-a"]
$ docker run entrypoint-test:0.1
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found ...
# 命令直接拼接在我们得ENTRYPOINT命令后面的,可以运行
$ docker run entrypoint-test:0.1 -l
total 56
drwxr-xr-x   1 root root 4096 May 16 06:32 .
drwxr-xr-x   1 root root 4096 May 16 06:32 ..
-rwxr-xr-x   1 root root    0 May 16 06:32 .dockerenv
lrwxrwxrwx   1 root root    7 May 11  2019 bin -> usr/bin
drwxr-xr-x   5 root root  340 May 16 06:32 dev
drwxr-xr-x   1 root root 4096 May 16 06:32 etc
drwxr-xr-x   2 root root 4096 May 11  2019 home
lrwxrwxrwx   1 root root    7 May 11  2019 lib -> usr/lib
lrwxrwxrwx   1 root root    9 May 11  2019 lib64 -> usr/lib64 ....
```

##### diyCentos

```shell
#编写dockerfile文件
$ vim Dockerfile

FROM centos
MAINTAINER y<1111111111@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim			#安装vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "-----end----"
CMD /bin/bash
```

```shell
#构建镜像
docker build -f Dockerfile -t diycentos:0.1 .

#可以查看创建的镜像的变更历史
docker history 镜像id
```

##### diyTomcat

准备文件

```shell
$ docker-tomcat ls
apache-tomcat-9.0.35.tar.gz dockerfile jdk-8u231-linux-x64.tar.gz
```

编写Dockerfile

```shell
FROM centos #
MAINTAINER cheng<1204598429@qq.com>
COPY README /usr/local/README #复制文件
ADD jdk-8u231-linux-x64.tar.gz /usr/local/ #复制解压
ADD apache-tomcat-9.0.35.tar.gz /usr/local/ #复制解压
RUN yum -y install vim
ENV MYPATH /usr/local #设置环境变量
WORKDIR $MYPATH #设置工作目录
ENV JAVA_HOME /usr/local/jdk1.8.0_231 #设置环境变量
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.35 #设置环境变量
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib #设置环境变量 分隔符是：
EXPOSE 8080 #设置暴露的端口
CMD /usr/local/apache-tomcat-9.0.35/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.35/logs/catalina.out # 设置默认命令
```

构建镜像

```shell
# dockerfile命名使用默认命名 因此不用使用-f 指定文件
$ docker build -t mytomcat:0.1 .
```

运行镜像

```shell
$ docker run -d -p 8080:8080 --name tomcat01 -v /home/ywb/build/tomcat/test:/usr/local/apache-tomcat-9.0.35/webapps/test -v /home/ywb/build/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.35/logs mytomcat:0.1
```

#### 发布镜像

##### 发布到dockerHub

```shell
$ docker images
REPOSITORY          TAG    IMAGE ID            CREATED             SIZE
ywbcreate/centos    1.0   c873387d7f29        4 hours ago         237MB

#在https://hub.docker.com/repository/docker/ywbcreate/centos-test创建账号和仓库

$ docker push ywbcreate/centos:1.0
The push refers to repository [docker.io/ywbcreate/centos]
0683de282177: Pushing [=>                                                 ]  7.627MB/237.1MB


#出错时
# 可能因为没有前缀的话默认是push到 官方的library
# 解决方法
# 第一种 build的时候添加你的dockerhub用户名，然后在push就可以放到自己的仓库了
$ docker build -t chengcoder/mytomcat:0.1 .
# 第二种 使用docker tag #然后再次push
$ docker tag 容器id chengcoder/mytomcat:1.0 #然后再次push
```

##### 发布到阿里云镜像

参考官方文档 	https://cr.console.aliyun.com/repository/

```shell
$ sudo docker login --username=111 registry.cn-hangzhou.aliyuncs.com
$ sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/ywbcreate/ywbcreate:[镜像版本号]
$ sudo docker push registry.cn-hangzhou.aliyuncs.com/ywbcreate/ywbcreate:[镜像版本号]
```

## Docker网络

#### 实现原理

​	Docker启动一个容器时会宿主机虚拟一个Docker容器网桥(docker0)，然后根据网桥的网段分配给容器一个IP地址，称为Container-IP，同时Docker网桥是每个容器的默认网关，这样容器之间就能够通过容器的Container-IP直接通信。

​	Docker网桥是宿主机虚拟出来的，并不是真实存在的网络设备，外部网络是无法寻址到的，这也意味着外部网络无法通过直接Container-IP访问到容器。如果容器希望外部访问能够访问到，可以通过映射容器端口到宿主主机（端口映射），即docker run创建容器时候通过 -p 或 -P 参数来启用，访问容器的时候就通过[宿主机IP]:[容器端口]访问容器。

#### 网络模式

```shell
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
03e001c25390        bridge              bridge              local
03651ebf9be4        host                host                local
aaf59b46115a        none                null                local
```

| 网络模式   | 配置方式                 | 说明                                                         |
| ---------- | ------------------------ | ------------------------------------------------------------ |
| Host       | –net=host                | 容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。 |
| **Bridge** | –net=bridge（默认模式）  | 此模式会为每一个容器分配、设置IP等，并将容器连接到一个docker0虚拟网桥，通过docker0网桥以及Iptables nat表配置与宿主机通信 |
| None       | –net=none                | 该模式关闭了容器的网络功能                                   |
| Container  | net=container:NAME_or_ID | 创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围。 |

##### ost模式

​	如果启动容器的时候使用host模式，那么这个容器将不会获得一个独立的Network Namespace，而是和宿主机**共用**一个Network Namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的。

​	使用host模式的容器可以直接使用宿主机的IP地址与外界通信，容器内部的服务端口也可以使用宿主机的端口，不需要进行NAT，host最大的优势就是**网络性能比较好**，但是docker host上已经使用的端口就不能再用了，**网络的隔离性不好**。

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200602194138676.png" alt="image-20200602194138676" style="zoom: 33%;" />

##### Bridge模式

​	当Docker进程启动时，会在主机上创建一个名为docker0的虚拟网桥，此主机上启动的Docker容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。

​	从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。在主机上创建一对虚拟网卡veth pair设备，Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0（容器的网卡），另一端放在主机中，以vethxxx这样类似的名字命名，并将这个网络设备加入到docker0网桥中。可以通过brctl show命令查看。

​	bridge模式是docker的默认网络模式，不写--net参数，就是bridge模式。使用docker run -p时，docker实际是在iptables做了DNAT规则，实现端口转发功能。可以使用iptables -t nat -vnL查看。

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200602194328031.png" alt="image-20200602194328031" style="zoom:33%;" />

##### Container模式

​	这个模式指定新创建的容器和已经存在的一个**容器共享**一个 Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过 lo 网卡设备通信。

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200602195147006.png" alt="image-20200602195147006" style="zoom: 33%;" />

##### None模式

​	使用none模式，Docker容器拥有自己的Network Namespace，但是，并不为Docker容器进行任何网络配置。也就是说，这个Docker容器没有网卡、IP、路由等信息。需要我们自己为Docker容器添加网卡、配置IP等。

这种网络模式下容器只有lo回环网络，没有其他网卡。none模式可以在容器创建时通过--network=none来指定。这种类型的网络没有办法联网，封闭的网络能很好的**保证容器的安全性。**

<img src="/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200602195245970.png" alt="image-20200602195245970" style="zoom:33%;" />

#### 自定义网络

​	可以使用`docker network create`自定义网络网络，用以控制那些容器可以相互通信

```shell
#自定义网络
$ docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
#查看网络
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
03e001c25390        bridge              bridge              local
03651ebf9be4        host                host                local
6f1c95061403        mynet               bridge              local
aaf59b46115a        none                null                local
#查看mynet
$ docker network inspect mynet
 				"Name": "mynet",
        "Id": "6f1c95061403a2b930dcacf59274074a81d1f52b86cec38d7d5bfd208bbbe0ff",
        "Created": "2020-06-02T12:06:38.9342452Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        }

```

自定义后测试同一网络下的容器

```shell
#在mynet下启动两个容器
$ docker run -d -P --name tomcat1 --net mynet tomcat
bf82c1631361647ce8905212de93bed882f6485d273119054e5204aa6c079ec7
$ docker run -d -P --name tomcat2 --net mynet tomcat
e83464b02c754ccc55b0385059130c9ae072a32887efa25a61ad5a61ece1d20c

#测试连通
$ docker exec tomcat1 ping tomcat2
PING tomcat2 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat2.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.173 ms
64 bytes from tomcat2.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.214 ms
$ docker exec tomcat2 ping tomcat1
PING tomcat1 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat1.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.118 ms
64 bytes from tomcat1.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.299 ms
```

这样启动容器可以保证不同的集群使用不同的网络，保证集群的安全

#### 网络连通

可以用`docker network connect`完成自定义网络下的容器连接docker0下的容器

```shell
$ docker run -d -P --name tomcat-net --net mynet tomcat
26a400c81a4a5e6f535702bc4b8b1c0fade79ebee4d9975fcaae0a265c2c2b94
$ docker run -d -P --name tomcat-docker0 tomcat
d499ba194170ad4097ce9cb34f0204a8d7fe98dc2186a22665ea00d4fa13648e
$ docker exec tomcat-net ping tomcat-docker0	#无法ping通不同网络下的容器
ping: tomcat-docker0: Temporary failure in name resolution

#将docker0下的tomcat加到mynet网络
$ docker network connect mynet tomcat-docker0
$ docker exec tomcat-net ping tomcat-docker0	#ping通
PING tomcat-docker0 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-docker0.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.172 ms
64 bytes from tomcat-docker0.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.209 ms
```

#### 部署Redis集群

```shell
#创建一个网卡
$ docker network create redis --subnet 172.38.0.0/16
#通过脚本创建六个redis集群
for port in $(seq 1 6);\
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >> /mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done
# 通过脚本运行六个redis
for port in $(seq 1 6);\
docker run -p 637${port}:6379 -p 1667${port}:16379 --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
#进入第一个redis
$ docker exec -it redis-1 /bin/sh #redis默认没有bash
#搭建集群
$ redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379  --cluster-replicas 1

#测试
$ redis -cli -c
$ cluster info
$ cluster nodes
$ set a b
$ docker stop redis-3#停掉主机测试高可用
$ get a
```

## springboot打包docker镜像

1.准备好打包好的springboot项目和Dockerfile

```shell
$ ls
Dockerfile              demo-0.0.1-SNAPSHOT.jar
```

Dockerfile

```shell
FROM java:8
COPY *.jar /app.jar
CMD ["--server.port=8080"]
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```

2.构建镜像

```shell
$ docker build -t hello .

Sending build context to Docker daemon  16.47MB
Step 1/5 : FROM java:8
8: Pulling from library/java
5040bd298390: Pull complete 
fce5728aad85: Pull complete 
76610ec20bf5: Pull complete 
60170fec2151: Pull complete 
e98f73de8f0d: Pull complete 
11f7af24ed9c: Pull complete 
49e2d6393f32: Pull complete 
bb9cdec9c7f3: Pull complete 
Digest: sha256:c1ff613e8ba25833d2e1940da0940c3824f03f802c449f3d1815a66b7f8c0e9d
Status: Downloaded newer image for java:8
 ---> d23bdf5b1b1b
Step 2/5 : COPY *.jar /app.jar
 ---> e551f769d10e
Step 3/5 : CMD ["--server.port=8080"]
 ---> Running in 2fcb15b89c34
Removing intermediate container 2fcb15b89c34
 ---> 52e4bba541d5
Step 4/5 : EXPOSE 8080
 ---> Running in 1c69291dcde0
Removing intermediate container 1c69291dcde0
 ---> 44da08687f42
Step 5/5 : ENTRYPOINT ["java","-jar","app.jar"]
 ---> Running in 56f6a79138b4
Removing intermediate container 56f6a79138b4
 ---> 63bf9f316619
Successfully built 63bf9f316619
Successfully tagged hello:latest
```

3.运行镜像

```shell
$ docker run -d -P --name hello1 hello
1139341c6e2490530adbe7efe672977798f61f540d9f907151c88a349e959398
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
1139341c6e24        hello               "java -jar app.jar -…"   4 seconds ago       Up 3 seconds        0.0.0.0:32768->8080/tcp   hello1
```

4.访问测试

http://localhost:32768/hello

![image-20200601223027846](/Users/yinwenbo/Library/Application Support/typora-user-images/image-20200601223027846.png)















