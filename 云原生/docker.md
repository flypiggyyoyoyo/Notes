# Docker

## 什么是Docker

### 基本概念

流程：研发镜像（image）->上传到镜像仓库（Registry）->通过Docker引擎down下来->运行down下来的容器（container）

- Docker是一个容器引擎，用于管理容器的声明周期

- 容器：包含用于运行一个软件的环境（包含了目标软件运行所需的所有依赖）和需要运行的软件，镜像的实例

- 镜像：容器的安装包，必须先有镜像，才能运行容器

- 仓库：用于存放镜像，对镜像进行统一管理

打个比方：仓库是软件商店，镜像是软件安装包，容器是安装成功后的软件

### Docker与虚拟机对比理解

![img_v3_02l1_6a2ab284-c6c6-4cae-8f0e-419693d50b4g](C:\Users\flypiggy\AppData\Roaming\LarkShell\sdk_storage\ca2990cc442c33c85ce584e6725af788\resources\images\img_v3_02l1_6a2ab284-c6c6-4cae-8f0e-419693d50b4g.jpg)

### Docker的架构

C/S架构：Client客户端，Server服务端

**Docker Client**

Docker客户端，向服务端发起请求，比如下载镜像，管理容器声明周期

**Docker Daemon**

Docker的后台守护程序，包含：

- Docker Server：服务端，接收请求

- Engine：容器引擎，负责执行任务

**Docker Registry**

镜像仓库



# Docker入门

## 基础命令

### 镜像

**``docker search jdk``**   用于查找jdk镜像的版本

**``docker pull openjdk:11``** 把jdk的镜像拉取到本地，冒号用于指定版本

**``docker images``**   查看本地镜像

**``docker rmi 123456``** 删除id号为123456的**本地镜像****

### 容器

**``docker ps``** 查看当前运行的容器

**``docker ps -a``** 查看当前所有容器名称 

**``docker run [容器名称]``** 运行目标名称的容器，默认前台运行

> **前台运行vs后台运行**
> 
> **前台运行**：启动容器时，将容器的输入输出与宿主机的终端相连，用户可以直接在当前终端与容器进行交互
> 
> **后台运行**：容器在后台独立运行，不与宿主机的终端直接关联。容器启动后，命令行会立即返回，你可以继续在宿主机终端执行其他操作

**``docker run -d -p 80:81 [容器名称]``**  运行目标名称的容器，-d指定后台运行，-p指定容器监听主机80端口，若有请求则转发到容器内81端口

> 因为资源隔离，所以容器内外网络不互通，eg内外都有80port，所以用到-p参数标识路由转发

**``docker run -d -P [容器名称]``** -P 参数的作用是让 Docker 自动在宿主机上选取一个可用的端口，并将其映射到容器内暴露(`EXPOSE`)的端口上，暴露端口指的是在`Dockerfile`里通过`EXPOSE`命令声明的容器内监听的端口

**``docker rm 123456``** 删除id为123456的停止的**容器**，id不一定为123456，只要123456能定位到一个唯一id的容器即可

**``docker rm -f 123456``** 强制删除id为123456的容器，无论是否运行中

**``docker stop 123456``** 停止id为123465的容器

**``docker start 123456``** 启动id为123456的容器

**``docker run -d -name cont01 [容器名称]``** 启动目标容器并指定名称为cont01，后续操作可用名字代替id

**``docker run --rm [容器名称]``** 运行容器，``rm``标识该容器停止后会自动删除

 **``docker run -d --restart on-failure:3 [容器名称]``**  ``--restart``标识重启策略，默认不重启；``on-failure``失败时重启，可以加上``:3``表示第三次部署失败就不重启了；``always``表示只要已关闭就自动重启

**``docker run -d -P -e JAVA_ENV=dev -e JAVA_VM=G1 nginx``** 启动 `nginx` 容器时设置 `JAVA_ENV=dev` 和 `JAVA_VM=G1` 这两个环境变量

 **``docker inspect NAME|ID``**  查看docker对象的详细信息

**``docker exec -it cont01 env``** exec命令是指定cont01容器，基于容器内终端（it），来执行env命令（显示环境变量）

**``-m 8m``** 限制内存最大为8mb，**``--cpus 1``** 限制cpu最多用一个

**``docker logs cont01``** 输出cont01容器的所有日志

**``docker exec -it cont01 bash``** 进入容器cont01内部，并使用bash

# Docker进阶

## 数据卷Volume

### 基本概念

- 是容器外部的独立存储空间，即使容器被删除，数据卷仍会保留。

- 同一个数据卷可被多个容器同时挂载，所有容器对卷内数据的读写操作是实时同步的

- 容器可通过数据卷直接访问宿主机文件系统，高性能

### 绑定方式

#### 匿名绑定

``docker run -v /app/data nginx``Docker 自动创建无名称卷，存储路径为宿主机的 `Docker 数据目录/volumes/随机哈希`

- 容器删除后卷仍然存在，需要手动清理

- 容器启动时加 `--rm`，停止后卷会自动删 除

#### 具名绑定

``docker run -d -v my-named-volume:/app/data my-image`` 不关心存放的位置，只想持久化存储一些内容

- 有自定义的名字，能够轻松识别和管理卷

- 除非手动删除，否则会一直存在

#### Bind Mount

把宿主机的目录直接映射到容器内的目录

`docker run -v /host/path:/container/path my-image`  `/host/path` 是宿主机上的路径，`/container/path` 是容器内的挂载路径

- 只要宿主机上的文件或目录存在，即使容器被删除，数据也不会丢失

- 可以挂载宿主机上的任意文件或目录，不受 Docker 卷存储管理的限制

- 宿主机和容器内的文件修改会实时同步

### 数据卷管理

```shell
Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  prune       Remove unused local volumes
  rm          Remove one or more volumes
```

## 网络Network

### 基本概念

#### 是什么

 是`Docker`对容器网络隔离的一项技术，提供了多种不同模式供用户使用，选择不同的网络模式来实现**容器网络**的互通以及的隔离

#### 需求

- 容器间的网络隔离

- 实现部分容器之间的网络隔离

- 管理多个子网下的容器ip

#### 作用

- 提供了多种模式，可以定制化的为每个容器制定不同的网络

- 自定义网络模式，划分不同的子网|网关|dns等配置

- 实现网络互通
  
  - 实现不同子网之间的网络互通
  
  - 基于容器名（主机名）的方式在网络内访问

### 网络模式

#### bridge桥接模式（默认模式）

在主机中创建一个`Docker0`的网桥，在`Docker0`创建一对虚拟网卡，有一半在主机上`vethxxx`，有一半在容器内`eth0`

![808eff48-2074-4c5d-83c6-60f78b1e24fb](file:///C:/Users/flypiggy/Pictures/Typedown/808eff48-2074-4c5d-83c6-60f78b1e24fb.png)

#### host模式

容器不再拥有自己的网络空间，而是直接与主机共享网络空间，基于该模式创建的容器对应的ip实际及时与主机为同一个子网，同一个网段

![fe78cec5-3527-4646-93af-a1f1e9644586](file:///C:/Users/flypiggy/Pictures/Typedown/fe78cec5-3527-4646-93af-a1f1e9644586.png)

#### none模式

`Docker`会拥有自己的网络空间，不与主机共享，在这个网络模式下的容器，不会被分配网卡&ip&路由等相关信息

- 完全隔离，与外部任何机器都无网络访问，只与自己的io，本地网络127.0.0.1

- 绝对安全

![771ff4b4-6b79-4bf5-9f4e-0dbd0b2940a7](file:///C:/Users/flypiggy/Pictures/Typedown/771ff4b4-6b79-4bf5-9f4e-0dbd0b2940a7.png)

#### Container模式

不会创建自己的网络，与其他容器共享网络空间，直接使用指定容器的ip/端口

![928768f6-dc31-4ee0-8217-b56d3d8f3e13](file:///C:/Users/flypiggy/Pictures/Typedown/928768f6-dc31-4ee0-8217-b56d3d8f3e13.png)

#### 自定义网络模式（推荐）

不适用`Docker`自带的网络模式，而是自己定制化自己持有的网络模式

命令：`docker network COMMOND` 

```shell
Manage networks

Commands:
  connect     Connect a container to  a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks
```

**为什么自定义网路中的容器能通过容器名ping通，但是bridge模式不行**

在`Docker`自定义网络中，`DNS`被深度集成以实现容器间的自动服务发现。当容器加入自定义网络时，`Docker`会将其容器名和`ip`地址注册到`DNS`服务器

bridge模式不嵌入DNS

### 不同模式下的链接

`docker network connect wolfcode net1`

![96dfcb27-c8a2-4f57-8d35-da986e99a865](file:///C:/Users/flypiggy/Pictures/Typedown/96dfcb27-c8a2-4f57-8d35-da986e99a865.png)

## Dockerfile文件

### 基本概念

#### 是什么

- **构建镜像**用的

- `Docker`为我们提供的一个用于自定义构建镜像的一个配置文件：描述如何构建一个对象

- 利用`Docker`提供的`build`命令，指定`Dockerfile`文件，就可以按照配置的内容将镜像构建出来 

#### 为什么需要

- 作为开发者，需要将自己开发好的项目打包成`Docker`镜像，便于后面直接作为`Docker`容器运行

- 作为运维人员，需要构建更精简的基础设施服务镜像，满足公司的需求以及尽可能减少冗余的功能占用过多的资源

#### 能做什么

- 可以自定义镜像内容

- 构建公共基础镜像减少其他镜像配置

- 开源程序快速部署

- 实现企业内容项目快速交付

### 常用指令

**`FROM`**

指定以什么镜像为基础镜像，在改进项的基础之上构建新的镜像   

如果不想以任何镜像为基础：`FROM scratch`

语法：   

- `FROM <image>`

- `FROM <image>:<tag>`

- `FRom <image>:<digest>`

以上三种写法，后两者为指定具体版本，第一种则使用`latest`也就是最新版

**`MAINTAINER`**

指定镜像的维护者

```dockerfile
MAINTAINER John Doe <johndoe@example.com> 
```

`MAINTAINER` 指令指定了镜像的维护者是 John Doe，其邮箱为 `johndoe@example.com`    

*Docker官方现已弃用MAINTAINER命令，他的功能可以用LABEL指令更灵活的实现*

**`LABEL`**

为镜像|容器|卷添加元数据

语法  

- `LABEL <key>=<value> <key>=<value> <key>=<value> ...`

可以在一条 `LABEL` 指令中定义多个键值对，也能使用多条 `LABEL` 指令。键值对之间用空格分隔，键和值之间用等号连接    

eg  

```dockerfile

LABEL maintainer="John Doe <johndoe@example.com>"
```

**`ENV`**

设置容器的环境变量，可以设置多个

语法：  

- `ENV <key> <value>`

- `ENV <key>=<value> <key>=<value>...`

**`RUN`**

`RUN` 指令在**构建镜像**期间执行，在 Dockerfile 中可以有多个 `RUN` 指令，每个 `RUN` 指令都会在镜像中创建一个新的层。为了减少镜像的层数和体积，建议将相关的命令组合在一个 `RUN` 指令中执行

语法：  

- `RUN <command>`

- `RUN ["executable","param1","param2"]`

第一种写法直接写`shell`脚本即可  

第二种写法类似于函数调用，第一个参数为可执行文件，后面的都是参数 

eg

```dockerfile
RUN ["/bin/bash", "-c", "echo hello"]
```

`-c` 后面跟着的 `"echo hello"` 会被当作 shell 命令来执行，所以直接调用 `/bin/bash` 来执行 `echo hello` 命令

**`ADD`**

复制命令，把`src`的文件复制到镜像的`dest`位置（从主机复制到容器）

语法：  

- `ADD <src> <dest>`

- `ADD ["<src>","<dest>"`

**`WORKDIR`**

设置并`cd`到容器中的工作目录，如果该目录不存在则自动创建。  

语法：  

- `WORKDIR /app`

在根目录下创建`app`目录

**`VOLUME`**

设置挂载目录，可以将主机中的指定目录挂载到容器中

语法：  

- `VOLUME ["<dir>"]`

- `VOLUME <dir>`

- `VOLUME <dir> <dir>`

**`EXPOSE`**

设置容器启动后要暴露的端口（容器在运行时会监听的网络端口）

语法：  

- `EXPOSE <port> [<port>/<protocol> ...]`

`<port>`表示容器要监听的端口号,`<protocol>`是可选参数，用于指定端口使用的协议，默认值为 `tcp`，也可以设置为 `udp`

*注意：`EXPOSE`只是暴露端口，并没有与主机的端口形成映射。映射行为是在容器`run`的时候加`-p`参数实现的*

**`CMD`**

`CMD` 指令在**容器**启动时执行，在 Dockerfile 中只能有一个 `CMD` 指令，若有多个 `CMD` 指令，只有最后一个会生效

语法：   

- `CMD echo "Hello world!"`

- `CMD ["echo","Hello world!"]` 

**`ENTRYPOINT`**

用来设置容器启动时要执行的命令，并且这个命令不能被 `docker run` 后面的参数直接覆盖，它提供了一种固定容器启动行为的方式，常用于设置容器的主命令



**`CMD`和`ENTRYPOINT`的区别**

- 单独使用 `ENTRYPOINT`，`docker run` 后面添加的参数会被当作 `ENTRYPOINT` 命令的参数

- 单独使用 `CMD`，如果 `docker run` 后面添加了其他命令，`CMD` 指定的命令会被覆盖

- 当 `ENTRYPOINT` 和 `CMD` 结合使用时，`CMD` 提供的参数会作为 `ENTRYPOINT` 命令的默认参数，若 `docker run` 后面添加了参数，这些参数会替换 `CMD` 的默认参数 

### 构建镜像

#### commit

基于一个现有的容器，构建一个新的镜像（image）

*定制化操作不方便！*

#### build

~~自己构建两个镜像练练手~~

eg   

```dockerfile
# 使用Python 3.9作为基础镜像
FROM python:3.9-slim

# 设置工作目录
WORKDIR /app

# 复制当前目录下的所有文件到工作目录
COPY . /app

# 安装应用所需的依赖
RUN pip install --no-cache-dir -r requirements.txt

# 暴露应用运行的端口
EXPOSE 5000

# 设置环境变量
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0

# 定义容器启动时执行的命令
CMD ["flask", "run"]
```

## 仓库Registry

- 用于管理`Docker`的镜像

- 快速交付

- 便于镜像的重复利用

![屏幕截图 2025-04-08 190149](C:\Users\flypiggy\Pictures\Screenshots\屏幕截图%202025-04-08%20190149.png)

### 常见仓库

- `Docker Hub`

- `Aliyun` 

- `Nexus`

- `Harbor`

## 容器编排

**这块没啥理论上的，实际部署一下就会了**

针对容器声明周期的管理，对容器声明周期进行更快速方便的方式进行管理

- 依赖管理，当一个容器必须在另一个容器运行完成后才能运行时，就需要进行依赖管理

- 副本数控制，容器有时候也需要集群，快速对容器集群进行弹性伸缩

- 配置共享，通过配置文件统一描述需要运行的服务相关信息，自动化的解析配置内容，并构建对应服务

*更简单的使用容器*

### Docker Compose(单机)

- 基于构建好的镜像来**创建和管理多个容器**

- 需求：在一台机器上部署多个容器   

- `Docker-Compose`可以高效便捷的管理单机上运行的所有容器，通过`yaml`配置文件的方式完成之前执行`docker run`命令所设置的所有参数。

![c50d5329-2370-4ae0-bea8-c9bc00c34e30](file:///C:/Users/flypiggy/Pictures/Typedown/c50d5329-2370-4ae0-bea8-c9bc00c34e30.png)

#### 服务services

需要运行的容器配置，可以理解为原先用`docker run`命令后面跟的一系列配置信息，都配在这里面

#### 网络networks

`docker-compose`公共自定义网络管理，配置好以后，可以直接在`services`中引用该网络配置，这个配置可以多个`services`使用

#### 数据卷volums

`docker-compose`下的统一数据卷管理，可以给多个`services`使用

### Swarm(分布式)

做`docker`的公司写的，与`k8s`竞争

## Portainer可视化工具

**很推荐啊，天天只在命令行docker来docker去，快docker抑郁了**

```shell
PS C:\Users\flypiggy> docker --version
Docker version 28.0.1, build 068a01e
PS C:\Users\flypiggy> docker volume create portainer_data
portainer_data
PS C:\Users\flypiggy> docker run -d -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
Unable to find image 'portainer/portainer-ce:latest' locally
latest: Pulling from portainer/portainer-ce
e2e06b27b87e: Pull complete
1fed1531b45b: Pull complete
04de093ad5ed: Pull complete
86a7cce72d42: Pull complete
e09df2601140: Pull complete
eae3ebf29ea8: Pull complete
c12aa3fbd31a: Pull complete
f111bda3f9a6: Pull complete
81021110ed01: Pull complete
4f4fb700ef54: Pull complete
Digest: sha256:7f10a26bfdda3fc58295ea09b860117ecd86a642d66fb94ce1f27a4c221d4649
Status: Downloaded newer image for portainer/portainer-ce:latest
61cbab51c68ba3134921a2d7beb1ce320716682cac10f2094c0f08f3ae01c4d8
```


