# Docker 从入门到放弃

## Docker 介绍

Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

## Docker 的应用场景

- Web 应用的自动化打包和发布。

- 自动化测试和持续集成、发布。

- 在服务型环境中部署和调整数据库或其他的后-台应用。

- 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境。

## Docker 镜像命令

### 查看镜像

Docker 需要频繁地操作相关的镜像，所以我们先来了解一下 Docker 中的镜像指令。

若想查看 Docker 中当前拥有哪些镜像，则可以使用 docker images 命令。

```sh
(base)  ~  docker image ls
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
redis                                           latest              de974760ddb2        6 days ago          105MB
docker.elastic.co/kibana/kibana                 7.1.0               714b175e84e8        23 months ago       745MB
docker.elastic.co/elasticsearch/elasticsearch   7.1.0               12ad640a1ec0        23 months ago       894MB
lmenezes/cerebro                                0.8.3               3a2daf87f0c7        2 years ago         333MB
```

其中 REPOSITORY 为镜像名，TAG 为版本标志，IMAGE ID 为镜像 id(唯一的)，CREATED 为创建时间，注意这个时间并不是我们将镜像下载到 Docker 中的时间，而是镜像创建者创建的时间，SIZE 为镜像大小。

## 下载镜像

如果需要下载镜像，使用以下命令：

```
docker pull mysql:8.0
```

docker pull 是固定的，后面写上需要下载的镜像名及版本标志；若是不写版本标志，而是直接执行 docker pull mysql，则会下载镜像的最新版本。

一般在下载镜像前我们需要搜索一下镜像有哪些版本才能对指定版本进行下载，使用指令：

```
(base)  ✘  ~  docker search mysql
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10760               [OK]
mariadb                           MariaDB Server is a high performing open sou…   4050                [OK]
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   791                                     [OK]
percona                           Percona Server is a fork of the MySQL relati…   533                 [OK]
centos/mysql-57-centos7           MySQL 5.7 SQL database server                   87
mysql/mysql-cluster               Experimental MySQL Cluster Docker images. Cr…   81
centurylink/mysql                 Image containing mysql. Optimized to be link…   59                                      [OK]
bitnami/mysql                     Bitnami MySQL Docker Image                      50                                      [OK]
databack/mysql-backup             Back up mysql databases to... anywhere!         43
deitch/mysql-backup               REPLACED! Please use http://hub.docker.com/r…   41                                      [OK]
prom/mysqld-exporter                                                              37                                      [OK]
```

不过该指令只能查看 MySQL 相关的镜像信息，而不能知道有哪些版本，若想知道版本，则只能这样查询：

```
(base)  ~  docker search mysql:8.0
NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
benoit93260/mysql-server8.0.19   server mysql:8.0.19 CentOs7 mysql-shell:8.0.…   0
lkhoho/mysql                     Thin wrapper of official mysql:8.0 image.       0
kamatimaru/mysql80-ja            Added Japanese support settings to mysql:8.0…   0
c3p16l12/mysql                   Built with mysql:8.0.13.                        0
```

若是查询的版本不存在，则结果为空

### 删除镜像

删除镜像使用指令：

```sh
# 删除指定镜像版本
docker image rm redis:latest

# 如果不指定版本，默认删除最新版本镜像
docker image rm redis

# 还可以通过镜像id来删除
docker image rm de974760ddb2

# 删除镜像的简化版本
docker rmi de974760ddb2
```

现在想删除所有的 redis 镜像，那么你需要查询出 redis 镜像的 id，并根据这些 id 一个一个地执行 docker rmi 进行删除。

但是现在，我们可以这样：

```sh
docker rmi -f $(docker images redis -q)
```

首先通过 docker images MySQL -q 查询出 MySQL 的所有镜像 id，-q 表示仅查询 id，并将这些 id 作为参数传递给 docker rmi -f 指令，这样所有的 MySQL 镜像就都被删除了。

## Docker 容器指令

掌握了镜像的相关指令之后，我们需要了解一下容器的指令，容器是基于镜像的。

### 通过镜像运行容器

若需要通过镜像运行一个容器，则使用：

```sh
# 运行容器
docker run redis:latest

# 运行镜像，并进入打开终端
docker run -t -i redis:latest /bin/bash

# 后台启动镜像
docker run -d redis:latest

# 指定端口映射运行
docker run -d  --name redis  -p 6379:6379  redis
```

当然了，运行的前提是你拥有这个镜像

### 查看容器

运行后查看一下当前运行的容器：

```sh
(base)  ~  docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
96c3d09f3040        redis               "docker-entrypoint.s…"   2 days ago          Up 3 hours          0.0.0.0:6379->6379/tcp   redis
```

其中 CONTAINER_ID 为容器的 id，IMAGE 为镜像名，COMMAND 为容器内执行的命令，CREATED 为容器的创建时间，STATUS 为容器的状态，PORTS 为容器内服务监听的端口，NAMES 为容器的名称。

通过该方式运行的 tomcat 是不能直接被外部访问的，因为容器具有隔离性，若是想直接通过 6379 端口访问容器内部的 redis，则需要对宿主机端口与容器内的端口进行映射：

```
docker run -d  --name redis  -p 6379:6379  redis
```

解释一下这两个端口的作用(6379:6379)，第一个 6379 为宿主机端口，第二个 6379 为容器内的端口，外部访问 6379 端口就会通过映射访问容器内的 6379 端口。

### 进入容器

从宿主机访问容器内部

```sh
# 进入容器
 docker exec -it 96c3d09f3040 bash
```

### 启停容器

```sh
# 终止M容器
docker stop 96c3d09f3040

# 强制停止容器
docker kill 96c3d09f3040

# 删除容器
docker rm 96c3d09f3040
```

## 其他命令

```sh
# 宿主机文件拷贝到容器
docker cp /opt/test/file.txt mycontainer:/opt/testnew/

# 容器文件拷贝到宿主机
docker cp mycontainer:/usr/local/file.txt /opt

# 构建新镜像
docker commit 9fc62fec647d imagename:v0.0.1

# 标签镜像
docker tag imagename:v0.0.1 imagename1:v0.0.1

# 上传镜像中心
docker push hub.docker.com/imagename1:v0.0.1

# 保存镜像
##-o：指定保存的镜像的名字；rocketmq.tar：保存到本地的镜像名称；imagename：镜像名字，通过"docker images"查看
docker save -o imagename.tar imagename
```
