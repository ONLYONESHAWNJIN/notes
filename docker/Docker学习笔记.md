# Docker学习笔记#

### Docker 和传统虚拟化方式的不同之处

传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

![传统虚拟化](https://yeasy.gitbooks.io/docker_practice/content/introduction/_images/virtualization.png)
<center>传统虚拟化</center>

![Docker](https://yeasy.gitbooks.io/docker_practice/content/introduction/_images/docker.png)
<center>Docker</center>

### Docker 镜像

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

#### DOCKER基础技术：AUFS
AUFS是一种Union File System，所谓UnionFS就是把不同物理位置的目录合并mount到同一个目录中。

#### 分层镜像
镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。

#### Docker 容器
镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。也因为这种隔离的特性，很多人初学 Docker 时常常会混淆容器和虚拟机。

#### Docker Registry
镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry 就是这样的服务。

一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

以 Ubuntu 镜像 为例，ubuntu 是仓库的名字，其内包含有不同的版本标签，如，14.04, 16.04。我们可以通过 ubuntu:14.04，或者 ubuntu:16.04 来具体指定所需哪个版本的镜像。如果忽略了标签，比如 ubuntu，那将视为 ubuntu:latest。

仓库名经常以 两段式路径 形式出现，比如 jwilder/nginx-proxy，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。

### Docker 命令

1.获取镜像

	docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

	例如 docker pull ubuntu:16.04 拉取ubuntu16.04镜像

2.运行容器

	docker run

	例如 
	docker run -it --rm \
	ubuntu:16.04 \
	bash

	
	options
	--rm   //当容器退出是自动删除
	-e  //设置容器的环境变量

3.列出镜像

	docker image ls
	
	docker system df 查看镜像、容器、数据卷所占用的空间。

4.查看容器

	docker container ls -a

### 使用Dockerfile定制镜像

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。


#### FROM 指定基础镜像

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 nginx 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 FROM 就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 scratch。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

	FROM scratch

#### RUN 执行命令
RUN 指令是用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之

- shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。

		RUN echo '<h1>Hello, Docker!</h1>' > /usr/share nginx/html/index.html
- exec 格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。


#### 构建镜像
	docker build [选项] <上下文路径/URL/->

当构建的时候，用户会指定构建镜像上下文的路径，docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。



### Docker镜像导入导出

1.导出镜像docker save

	docker save 9045 > tomcat8-apr.tar

2.载入镜像

	docker load < tomcat8-apr.tar



### Docker重启策略

Docker容器的重启策略是面向生产环境的一个启动策略，在开发过程中可以忽略该策略。

Docker容器的重启都是由Docker守护进程完成的，因此与守护进程息息相关。

Docker容器的重启策略如下：

- no，默认策略，在容器退出时不重启容器
- on-failure，在容器非正常退出时（退出状态非0），才会重启容器

	on-failure:3，在容器非正常退出时重启容器，最多重启3次

- always，在容器退出时总是重启容器
- unless-stopped，在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器

示例：

	docker run -d --restart=always bba-208
	docker run -d --restart=on-failure:10 bba-208

如果创建时未指定 --restart=always ,可通过update 命令设置

	docker update --restart=always xxx

## Docker部分容器启动示例

### Centos
docker pull centos:7.2.1511


docker run -d -it --name centos1 centos:7.2.1511 /bin/bash


docker exec -it centos1 /bin/bash



---

### Zookeeper


	docker pull zookeeper

	https://github.com/31z4/zookeeper-docker/blob/master/README.md

	docker-compose -f zk.yml -p zk-test up

	docker-compose -f zk.yml -p zktest ps

	docker run -it --rm \
        --net zk_default \
        zookeeper zkCli.sh -server zoo1:2181

---

### Redis


	docker run --name redis1 \
	-v /usr/local/docker/redis/data:/data \
	-p 6379:6379 \
	-d redis:3.2 redis-server --appendonly yes

	docker run -it --rm \
        redis:3.2 redis-cli \
		-h 172.17.0.2 -p 6379

---

### Gitlab类似github的一个网站

	sudo docker pull gitlab/gitlab-ce:latest


	sudo docker run --detach \
	--hostname 192.168.5.101 \
    --publish 10443:443 --publish 1080:80 --publish 1022:22 \
    --name gitlab \
    --volume /usr/local/docker/gitlab/config:/etc/gitlab \
    --volume /usr/local/docker/gitlab/logs:/var/log/gitlab \
    --volume /usr/local/docker/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest

### ngrok内网转发

[使用Docker搭建Ngrok服务器](https://hteen.cn/docker/docker-ngrok.html)

启动一个容器生成ngrok客户端,服务器端和CA证书

	docker run --rm -it -e DOMAIN="ngrok.comeon111.xyz" \
	-v /data/ngrok:/myfiles hteen/ngrok /bin/sh /build.sh

启动Ngrok server不绑定80和443端口

	docker run -idt --name ngrok-server \
	-v /data/ngrok:/myfiles \
	-p 8082:80 \
	-p 4432:443 \
	-p 4443:4443 \
	-e DOMAIN='ngrok.comeon111.xyz' hteen/ngrok /bin/sh /server.sh



## Dockerfile



### FROM 

指定基础镜像


### ADD COPY


将宿主文件添加到镜像中

	ADD <src>... <dest>



src可以是文件，文件夹，也可以是多个文件，如果是多文件或文件夹，dest必须是文件夹后面加`/`

COPY和ADD使用方法一致

区别：
ADD支持解压tar,从url拉取文件

### WORKDIR

指定工作目录，docker会自动跳转到此目录

	WORKDIR <dirpath>

### VOLUME

用来创建挂载点

	VOLUME [<mountpoint>]

后面的参数指定的是镜像的目录，docker挂载完后会自动在宿主机上生成一个目录，随机生成的好处是，防止多容器在宿主机上挂载点冲突。

### EXPOSE

打开容器的端口，使容器可以和外部通讯

	EXPOSE <port>[/<protocol>] [<port>[/<protocol>]]

protocol可以是 TCP UDP


### ENV

用来定义镜像所需要的环境变量，供后面的命令使用

	ENV <key> <value>
	ENV <key>=<value> ... 

### ARG

用法同ENV相同

	ARG <name>[=<default value>]

在dockers build构建镜像时可以使用 `--build-arg <varname>=<value>`来指定参数，如果不指定，则使用默认值

### RUN

指定docker build过程中运行的命令

	RUN <command>
	RUN [<executanle>,<param1>,<param2>]


### CMD

容器启动时运行的命令

	CMD <command>
	CMD [<executanle>,<param1>,<param2>]
	CMD [<param1>,<param2>]


1. RUN和CMD的区别，RUN运行在build过程，CMD则运行在构建后的镜像上
2. 如果有多个CMD指令，只有最后一个生效
3. CMD主要给容器指定默认要运行的程序，且运行结束后，容器也会终止。

### ENTERPOINT

类似CMD指令的功能，用户预定默认运行程序

	ENTERPOINT <command>
	ENTERPOINT [<executanle>,<param1>,<param2>]

ENTERPOINT和CMD的区别是：ENTERPOINT不会被docker的指令覆盖


	

