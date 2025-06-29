# 容器概述

容器：一个运行在隔离环境中的程序

镜像：镜像是只读的模板，包含了创建容器所需的所有文件和配置信息

仓库：用来存储、分发、管理镜像的地方

# 容器安装

安装docker软件包

容器对外提供服务依赖路由转发功能，检查宿主机是否开启路由转发功能

# Docker管理

## 镜像管理

镜像可以从官方镜像仓库下载，也可以自己制作

官方镜像仓库https://hub.docker.com，由于网络环境，国内无法正常访问官方网站，需要通过配置镜像加速器访问`/etc/docker/daemon.json`

```bash
{
    "registry-mirrors": ["这里配置镜像仓库加速器地址"],
    "insecure-registries":[]
}
```

### **指定镜像的方法**

每个镜像都对应一个唯一的ID;

镜像名称+标签 == 唯一

每一个镜像都有标签，默认标签 latest

### **管理命令**

| 镜像管理命令                        | 说明                                                         |
| :---------------------------------- | :----------------------------------------------------------- |
| docker images                       | 查看本机镜像                                                 |
| docker pull 镜像名称:标签           | 下载镜像                                                     |
| docker save 镜像名称:标签 -o 文件名 | 备份镜像为tar包                                              |
| docker load -i 备份文件名称         | 导入备份的镜像文件                                           |
| docker history 镜像名称:标签        | 查看镜像的制作历史                                           |
| docker rmi [- f ]  镜像名称/镜像ID  | 删除镜像（镜像创建的容器运行中则不能删除，强制删除只是删除了名称和ID） |
| docker  tag 镜像  新名称:标签       | 设置镜像名称标签                                             |

###  镜像概述

镜像是创建容器的核心，采用<font color='red'>COW</font>技术，分层技术，镜像始终是<font color='red'>只读</font>

![image-20250603143134433](./image-20250603143134433.png)

**Copy on Write 写时拷贝技术**

该技术使用指针指向原始盘所有块；若一个块要被改写，首先把数据从原始盘中拷贝到前端盘，然后在前端盘进行改写，最后把数据指针指向改写过的数据，原始盘始终是只读

![image-20250603144250830](./image-20250603144250830.png)

创建容器时，COW为镜像创建一个读写层，容器在读写层运行

## 容器管理

| 容器管理命令                         | 说明                                         |
| ------------------------------------ | -------------------------------------------- |
| docker run -it(d) 镜像名称:标签      | 创建容器                                     |
| docker ps                            | 查看容器的信息，-a 显示所有容器  -q 只显示ID |
| docker inspect 镜像名称\|容器名称    | 查询（容器/镜像）的详细信息                  |
| docker [start\|stop\|restart] 容器id | 启动、停止、重启容器                         |
| docker exec -it 容器ID 启动命令      | 在容器内执行命令                             |
| docker logs 容器ID                   | 查看容器日志                                 |
| docker cp 路径1 路径2                | 拷贝文件：路径格式（本机路径、容器ID/路径）  |
| docker rm [-f]  容器ID               | 删除容器，-f 强制删除                        |

### run 命令

docker run 常用参数

- **-i** 交互式
- **-t** 分配终端
- **-d** 后台运行
- **--name**  容器名称
- **--rm** 容器结束后自动删除，临时容器
- 转入后台快捷键 ctrl + p + ctrl + q，

注：`docker ps -aq`只显示id可以用于容器启动关闭等操作的参数

**在容器内执行命令**

```bash
docker exec -it web1 ls -l	#执行非交互命令
docker exec -it web1 bash 	#执行交互命令，进入到容器中
```

##  自定义镜像

使用现有镜像启动容器，在该容器基础上修改，使用commit制作新镜像

```bash
docker commit linux  mylinux:latest  #linux为容器名称，后面定义了新镜像的名称和标签
```

## 容器服务原理

如何在容器中启动服务？

由于容器内没有systemd，参考服务文件手工执行启动程序，如果没有service文件，参考官方手册配置、启动服务

```mermaid
sequenceDiagram
    Docker Daemon->>容器进程: 启动容器
    loop 健康检查
        Docker Daemon->>容器进程: 监控 PID 1 状态
        alt PID 1 存在
            容器进程-->>Docker Daemon: 正常运行
        else PID 1 退出
            容器进程-->>Docker Daemon: 退出信号
            Docker Daemon->>容器进程: 发送 SIGKILL
        end
    end
```

### 守护进程的原理

在Unix/Linux系统中，守护进程是一种在后台运行的进程，它独立于控制终端。通常，一个程序要成为守护进程，会通过以下步骤：

\- <font color='red'>调用`fork()`创建一个子进程，然后父进程退出</font>（这样使得子进程成为孤儿进程，并被init进程接管）。

\- 在子进程中调用`setsid()`创建一个新的会话，并脱离控制终端。

\- 再次`fork()`一个子进程并退出父进程（即第一次创建的子进程），这样确保新的进程不是会话组长，从而防止其重新获取终端。

\- 改变工作目录（如`/`），重设文件掩码，关闭不需要的文件描述符等。

# Dockerfile镜像制作

commit的局限性

很容易制作简单的镜像，但碰到复杂的情况就十分不方便，例如碰到下面的情况：

- 需要设置默认的启动命令
- 需要设置环境变量
- 需要指定镜像开放某些特定的端口

Dockerfile是一种更强大的镜像制作方式，编写类似脚本的Dockerfile文件，通过该文件制作镜像

```bash
docker build  -t 镜像名称:标签 Dockerfile所在目录
```

| **指令**   | **说明**                                               |
| ---------- | ------------------------------------------------------ |
| FROM       | 指定基础镜像（scratch 代表空镜像）                     |
| RUN        | 在容器内执行命令，可以写多条                           |
| ADD        | 把文件拷贝到容器内，如果文件是 tar.xx 格式，会自动解压 |
| COPY       | 把文件拷贝到容器内，不会自动解压                       |
| ENV        | 设置启动容器的环境变量                                 |
| WORKDIR    | 设置启动容器的默认工作目录（唯一）                     |
| CMD        | 容器默认的启动参数（唯一）                             |
| ENTRYPOINT | 容器默认的启动命令（唯一）                             |
| USER       | 启动容器使用的用户（唯一）                             |
| EXPOSE     | 使用镜像创建的容器默认监听使用的端口号/协议            |

# 部署容器服务

## 对外发布服务-端口映射

容器化带来的问题：新建容器的IP随机，每次重启后IP地址都会发生变化，只有宿主机可以访问。

解决方法：容器端口可以和宿主机端口进行映射绑定，从而把宿主机变成对应的服务，不用关心容器的IP地址，每个端口都只能和一个容器绑定

语法格式

```bash
docker run -p [可选IP]:宿主机端口:容器端口 
```

## 容器卷

容器化问题：容器不适合保存任何数据，容器造成数据丢失以及管理问题，修改多个容器中的数据非常困难，多容器之间有数据共享、同步的需求，数据文件和配置文件频繁更改

Docker可以映射宿主机文件或目录到容器中

- 目标对象不存在就自动创建
- 目标对象存在就直接覆盖掉

- 多个容器可以映射同一个目标达到数据共享的目的

命令格式：

```bash
docker run -itd -v 宿主机对象:容器内对象 镜像名称:标签  #-v的参数可以有多个
```

## 容器网络通信

docker的网络通信模式

- bridge 模式，默认
- host 模式，与宿主机共享网络
- none 模式，无网络模式
- container 模式，共享其他容器的网络命名空间
- 自定义模式

### 共享网络命名空间

参数 `--network=container:容器名称/ID`

# 微服务

微服务架构：微服务并不是一种技术，而是架构思想，它以容器技术为载体，演进出一种以软件运行环境、产品、研发、运营为一体的全新模式。站在 Docker 的角度，软件就是容器的组合，而容器又是服务的最佳载体，一台计算机同时运行多个容器，从而能很轻松地实现复杂的架构。

微服务架构的优点：松耦合高内聚，高度可扩展，出色的弹性，易于部署，易于访问

容器服务编排

- 在微服务架构中每个微服务一般都会包含多个容器实例。
- 如果每个微服务都要手动管理，那么效率之低、维护量之大可想而知。为了解决编排部署的问题，docker 公司推出了 docker Compose 工具
- Compose 是一个用于定义和运行多容器的应用的工具。
- 使用 Compose，可以在一个文件中配置多个容器服务，然后使用一个简单的命令就可以轻松、高效地管理配置中引用的所有容器服务。

```bash
[root@docker ~]# vim docker-compose.yaml
name: myweb # 项目名称
version: "3" # 语法格式版本  
services: # 关键字，定义服务
  websvc: # 服务名称
    container_name: nginx # 容器名称
      image: myos:nginx # 创建容器使用的镜像
```

| **指令**           | **说明**                   |
| ------------------ | -------------------------- |
| up                 | 创建项目并启动容器         |
| ls                 | 列出可以管理的项目         |
| images             | 列出项目使用的镜像         |
| ps                 | 显示项目中容器的状态       |
| logs               | 查看下项目中容器的日志     |
| start/stop/restart | 启动项目/停止项目/重启项目 |
| down               | 删除项目容器及网络         |

容器项目管理

```bash
docker compose -f docker-compose.yaml up -d 		#创建项目，并启动
docker compose ls -a								#查看项目
docker compose -p myweb ps							#查看项目中的容器
docker compose -p myweb images 						#查看项中使用的镜像
```

### compose 语法

| **指令**       | **说明**                                                     |
| :------------- | ------------------------------------------------------------ |
| container_name | 指定容器名称                                                 |
| image          | 指定为镜像名称或镜像 ID                                      |
| ports          | 暴露端口信息                                                 |
| volumes        | 数据卷,支持 [volume、bind、tmpfs、npipe]                     |
| network_mode   | 设置网络模式                                                 |
| environment    | 设置环境变量                                                 |
| restart        | 容器保护策略[always、no、on-failure]                         |
| command        | 覆盖容器启动后默认执行的命令                                 |
| healthcheck    | 配置服务健康检测                                             |
| depends_on     | 服务依赖关系 services_[started、healthy、completed_successfully] |

### 容器服务编排

```yaml
[root@docker ~]# vim docker-compose.yaml 
name: myweb
version: "3"
services:
  websvc:
    container_name: nginx
    image: myos:nginx
    ports:
      - 80:80
    environment:
      - "TZ=Asia/Shanghai"
    volumes:
      - type: bind
        source: /root/conf/nginx.conf
        target: /usr/local/nginx/conf/nginx.conf
      - type: bind
        source: /var/webroot
        target: /usr/local/nginx/html
  phpsvc:
    container_name: php
    image: myos:php-fpm
    restart: always
    network_mode: "service:websvc"
    volumes:
      - type: bind
        source: /var/webroot
        target: /usr/local/nginx/html
```

# harbor私有仓库搭建

