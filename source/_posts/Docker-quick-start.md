---
title: Docker 弹射起步
date: 2021-04-15 19:59:17
categories:
- 弹射起步
tags:
- docker
---

- [学后总结](#学后总结)
- [虚拟机 Vs 容器](#虚拟机-vs-容器)
- [常用命令](#常用命令)
- [Nginx 镜像操作实践](#nginx-镜像操作实践)
- [Tomcat 镜像操作实践](#tomcat-镜像操作实践)
- [部署 ES + kibana 操作实践](#部署-es--kibana-操作实践)
- [可视化](#可视化)
- [Docker 镜像加载原理](#docker-镜像加载原理)
- [制作镜像](#制作镜像)
- [数据卷](#数据卷)
- [安装 MySQL 操作实践](#安装-mysql-操作实践)
- [具名挂载 Vs 匿名挂载](#具名挂载-vs-匿名挂载)
- [初识 Dockerfile](#初识-dockerfile)
- [数据卷容器](#数据卷容器)
- [Dockerfile](#dockerfile)
- [Dockerfile 构建过程](#dockerfile-构建过程)
- [Dockerfile 指令](#dockerfile-指令)
- [CMD Vs ENTRYPOINT](#cmd-vs-entrypoint)
- [实战： 制作 Tomcat 镜像](#实战-制作-tomcat-镜像)
- [Docker 网络详解](#docker-网络详解)
- [自定义网络](#自定义网络)
- [网络联通](#网络联通)
- [实战：部署 Redis 集群](#实战部署-redis-集群)
- [SpringBoot 微服务打包 Docker 镜像](#springboot-微服务打包-docker-镜像)
- [Docker Compose](#docker-compose)
- [安装](#安装)
- [官方起步教程](#官方起步教程)
- [YAML 规则](#yaml-规则)
- [实战](#实战)
- [Docker Swarm](#docker-swarm)
- [Raft 协议](#raft-协议)
- [搭建集群](#搭建集群)
- [以后还要学 Go](#以后还要学-go)
- [问题](#问题)

## 学后总结

* 基本了解了 docker 相关的整个生态的基本情况
* 熟悉了 docker 的基本用法
* 有机会要学一下 Go 语言

## 虚拟机 Vs 容器

* 虚拟机运行整个系统，在系统上安装运行软件
* 容器内的应用直接运行在宿主机内，容器没有自己的内核，也没有虚拟硬件

## 常用命令

```bash
# docker client 信息显示
docker version  # docker engin, api 等的版本信息
docker info     # docker 环境信息，包括 image, container 数量，内核版本等
docker --help   # 帮助

# 镜像相关命令
docker pull [OPTIONS] NAME[:TAG|@DIGEST]    # 拉镜像
# Host> docker pull mysql                   # 没有指定 tag 就默认下载 latest 版本
# Using default tag: latest
# latest: Pulling from library/mysql
# f7ec5a41d630: Already exists 
# 9444bb562699: Pull complete               # 分层下载， docker image 的核心，联合文件系统
# ...
# e47da95a5aab: Pull complete 
# Digest: sha256:04ee7141256e83797ea4a84a4d31b1f1bc10111c8d1bc1879d52729ccd19e20a # 签名
# Status: Downloaded newer image for mysql:latest
# docker.io/library/mysql:latest            # 真实地址，等价于 docker pull docker.io/library/mysql:latest

# 常用参数
# -f: 强制删除
docker rmi [OPTIONS] IMAGE [IMAGE...]               # 删除镜像
docker rmi -f $(docker images -aq)                  # 组合命令，删除全部镜像

# 容器相关
# 常用参数
# --name="my_name"          # 指定容器名字
# -d                        # 后台运行
# -it                       # 交互方式运行，进去容器查看, 示例: docker run -it centos /bin/bash
# -p                        # 指定端口
#   -p ip:主机端口:容器端口
#   -p 主机端口:容器端口
#   -p 容器端口
# -P                        # 随机指定端口
# 常见坑：docker 容器使用后台运行，必须要给一个前台进程，如果 docker 发现没有应用就会自动停止
# 比如启动 nginx 容器，如果没有 -it 参数，容器就会立刻停止，ps 之后不会显示这个容器，加 -a 可以
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# 生成临时 log
docker run -d centos /bin/sh -c "while true;do echo testlog;sleep 1;done"

# -t                # 时间戳
# -f                # 持续打印
# --tail num        # 输出n条
# sample: docker logs -tf --tail 10 627b379be47a
docker logs [OPTIONS] CONTAINER         # 输出 console log

# -a                # 所有容器, 包括已经停止的
# -aq               # 只显示容器 id
# -n=2 or -n 2      # 最近创建的2个容器
docker ps [OPTIONS] # 显示容器信息

ctrl + p + q        # 交互模式下推出容器并后台运行。mac 也是这个命令，不过 vscode 里不好使，应该是快捷键冲突

# -f                                                    # 强制移除
# sample: docker rm -f $(docker ps -qa)                 # 删除所有
# sample: docker ps -aq | xargs docker rm               # 通过 Linux pip 方式删除所有
docker rm [OPTIONS] CONTAINER [CONTAINER...]            # 删除容器，不能删除正在运行的容器，除非加 -f

# 删除由某个 image 生成的所有 containers
# 这里有个小技巧，先通过 docker ps 打印一下，避免误删：docker ps -a --filter ancestor=nginx
docker rm -f $(docker ps -aq --filter ancestor=nginx)

docker start [OPTIONS] CONTAINER [CONTAINER...]         # 启动停止的容器
docker restart [OPTIONS] CONTAINER [CONTAINER...]       # 重启容器
docker stop [OPTIONS] CONTAINER [CONTAINER...]          # 停止容器
docker docker kill [OPTIONS] CONTAINER [CONTAINER...]   # stop 报错了可以用这个强制杀进程

docker top CONTAINER [ps OPTIONS]                       # 显示容器中的进程, 如下显示志之前 log 例子的 top 信息
# docker top 627b379be47a
# UID          PID          PPID         C            STIME               TTY          TIME         CMD
# root         12866         12840       0            10:29               ?            00:00:00     /bin/bash -c while true; do echo testlog; sleep 1; done

docker inspect [OPTIONS] NAME|ID [NAME|ID...]           # 显示容器底层信息, 包括 id, image, 共享卷，网络等信息
# [
#     {
#         "Id": "247d2a88573fdb2893a90a3d35275bfaa2889f7fa450d875456646ed684643d4",
#         "Created": "2021-04-20T13:06:45.2632089Z",
#         "Path": "/bin/sh",
#         "Args": [
#             "-c",
#             "while true;do echo testlog;sleep 1;done"
#         ],
#         "State": {
#             "Status": "running",
#             ...
#         },
#         "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
#         ...
#         "GraphDriver": {
#             "Data": {
#                 "WorkDir": "/var/lib/docker/overlay2/80783254f01fcdde559ac63ff7503d2dc317929d0328fb1c66846f9e519d98df/work"
#                 ...
#             },
#             "Name": "overlay2"
#         },
#         "Mounts": [],
#         "Config": {
#             ...
#             "Cmd": [
#                 "/bin/sh",
#                 "-c",
#                 "while true;do echo testlog;sleep 1;done"
#             ],
#             "Image": "centos",
#             "Volumes": null,
#             "WorkingDir": "",
#             "Entrypoint": null,
#             "OnBuild": null,
#             "Labels": {
#                 "org.label-schema.build-date": "20201204",
#                 ...
#             }
#         },
#         "NetworkSettings": {
#             "Bridge": "",
#             ...
#             "MacAddress": "02:42:ac:11:00:02",
#             "Networks": {
#                 "bridge": {
#                     "IPAMConfig": null,
#                     "Links": null,
#                     "Aliases": null,
#                     "NetworkID": "2fbb6bb1ed5e760a8350664377ea726ffbf35fab4794d45926ab9f9f9bd28d8a",
#                     "EndpointID": "aa43be83ff078d4c2dbdff62a2e69217ef2e17c0585edfecdcda030dca3aef0f",
#                     "Gateway": "172.17.0.1",
#                     "IPAddress": "172.17.0.2",
#                     "IPPrefixLen": 16,
#                     "IPv6Gateway": "",
#                     "GlobalIPv6Address": "",
#                     "GlobalIPv6PrefixLen": 0,
#                     "MacAddress": "02:42:ac:11:00:02",
#                     "DriverOpts": null
#                 }
#             }
#         }
#     }
# ]

# sample: docker exec -it 627b379be47a /bin/bash 
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]        # 进入容器，开启一个新的终端
       
docker attach [OPTIONS] CONTAINER                       # 进去容器，为当前正在执行的终端

# docker cp ./myyyyyy.tar  9b41928f2fb3:/
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-      # 容器和宿主机之间文件**互相**拷贝, - 这个符号可以操作 tar，查了一下没使用案例，测试失败
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

## Nginx 镜像操作实践

```bash
docker search nginx         # 从官方 repo 搜索镜像
# NAME                               DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
# nginx                              Official build of Nginx.                        14752     [OK]
# jwilder/nginx-proxy                Automated Nginx reverse proxy for docker con…   2018                 [OK]

docker pull nginx                               # 下载镜像

# -d                        # 后台运行
# --name:my-nginx           # 自定义容器名称
# -p                        # 指定端口号
docker run -d --name nginx01 -p 3344:80 nginx   # 启动容器

curl localhost:3344                             # 访问暴露的地址测试是否成功启动, 返回页面如下
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
# <body>
# <h1>Welcome to nginx!</h1>
# <p>If you see this page, the nginx web server is successfully installed and
# working. Further configuration is required.</p>
# ...
# <p><em>Thank you for using nginx.</em></p>
# </body>
# </html>

docker exec -it nginx01 /bin/bash               # 进入容器
whereis nginx                                   # linux 基础命令，查看配置
```

## Tomcat 镜像操作实践

```bash
# --rm      # 一般用于测试，用完即删除
docker run -it --rm tomcat:9.0                      # tomcat docker 镜像官方命令，可以达到测试完毕，推出即删除的效果

docker run -d -p 3355:8080 --name tomcat01 tomcat   # 容器命名为 tomcat01 跑在 3355 端口
curl localhost:3355                                 # 访问测试, 访问失败
# <!doctype html>...</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists...</body></html>

docker exec -it tomcat01 /bin/bash                  # 进入容器查看原因

cp -r webapps.dist/* webapps                        # 官方镜像 webapps 文件夹为空，样板页面放到了 webapps.dist 下了。拷贝一下，问题解决
```

## 部署 ES + kibana 操作实践

ES 的问题：

* ES 暴露的接口多
* ES 十分耗内存
* ES 数据需要备份

```bash
# sample of official:
#   docker network create somenetwork
#   docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:tag
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2

docker stats            # 实时显示容器的资源使用情况
# CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O       PIDS
# 5e20f16bed99   elasticsearch   1.01%     1.332GiB / 15.64GiB   8.52%     836B / 0B   106MB / 729kB   46

curl localhost:9200     # 发送请求测试
# {
#   "name" : "4d87cd0d4ff6",
#   "cluster_name" : "docker-cluster",
#   "cluster_uuid" : "ojWX85pITJyL7WkVnoKZcA",
#   "version" : {
#     "number" : "7.6.2",
#    ...
#   },
#   "tagline" : "You Know, for Search"
# }

# -e        # 启动时添加环境配置，sample: ES_JAVA_OPTS="-Xms64m -Xmx512m" 修改内存配置, 再次查看 stats 可以看到内存使用量变化
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2
# CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O    PIDS
# 7a804bd391d6   elasticsearch   229.04%   318.1MiB / 15.64GiB   1.99%     766B / 0B   0B / 246kB   27
```

## 可视化

* Portainer - 图形化管理工具
* Rancher - CI/CD

```bash
# 访问 localhost:9000 可见页面
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

## Docker 镜像加载原理

UnionFS 联合文件系统，分层，轻量级且高性能。有机会再做扩展。

## 制作镜像

```bash
# -a                # 作者
# -m                # commit 信息
# 630bab3ed5c2      # 修改过的 container id
# tomcat02:1.0      # 新镜像名称:版本号
docker commit -a='jzheng' -m='add webapps' 630bab3ed5c2 tomcat02:1.0
docker images
# REPOSITORY            TAG            IMAGE ID       CREATED         SIZE
# tomcat02              1.0            67be5e0517c6   7 seconds ago   672MB
```

## 数据卷

docker 容器删除之后，运行时产生的数据也会一起删除，为了保留这些数据，我们有了数据卷技术。通过数据卷技术，我们将容器中数据同步到宿主机，容器之间也可以通过这个技术做数据共享。

```bash
# -v, --volume list         # 挂载数据卷(Bind mount a volume)
# sample: docker run -it -v /Users/id/tmp/mount:/home centos /bin/bash
docker run -it -v host_addr:container_addr [REPOSITORY[:TAG]]
# 测试 tomcat 时发现，-v 会以本地的文件夹为基准
# sample: docker run -d --name my-tomcat -P -v /Users/jack/tmp/mount:/usr/local/tomcat/webapps.dist tomcat
# 问题：
#   1. 本地路径必须是全路径 './mount' 会查找失败
#   2. 挂载之后会以本地文件为基准，比如上例，webapps.dist 挂载到本地后，进入容器，这个文件夹下原有的文件都没了

docker inspect [OPTIONS] NAME|ID [NAME|ID...]       # 通过 inspect 可以看到具体的挂载信息

# ...
# "Mounts": [
#     {
#         "Type": "bind",
#         "Source": "/Users/jack/tmp/mount",
#         "Destination": "/usr/local/tomcat/webapps.dist",
#         "Mode": "",
#         "RW": true,
#         "Propagation": "rprivate"
#     }
# ],
# ...

停止容器，修改宿主机下的同步文件夹内容，容器启动后改动会同步到容器中
```

## 安装 MySQL 操作实践

```bash
docker pull mysql:5.7

# -e MYSQL_ROOT_PASSWORD=my-secret-pw           # 按官方镜像文档提示，启动容器时设置密码
docker run -d -p 3000:3306 -v /Users/id/tmp/mysql/conf:/etc/mysql/conf.d -v /Users/id/tmp/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name=mysql01 mysql:5.7

# 启动 DBeaver，链接数据库，报错：`Unable to load authentication plugin 'caching_sha2_password'.`
# 搜索之后，发现是 mysql 驱动有跟新，需要修稿客户端的 pom, 升级到 8.x 就行。DBeaver 直接就在创建选项里给了方案，选 8.x 那个就行 [GitIssue](https://github.com/dbeaver/dbeaver/issues/4691)
# 使用高版本的 Mysql connection 还是有问题，不过 msg 变了：`Public Key Retrieval is not allowed`
# 搜索之后，发现还要改配置, connection setting -> Driver properties -> 'allowPlblicKeyRetrieval' 改为 true
# 还有问题。。。继续抛错：`Access denied for user 'root'@'localhost' (using password: YES)`
docker exec -it mysql01 /bin/bash               # 进去容器，输入 `mysql -u root -p` 尝试登陆，成功。推测是链接客户端的问题
ps -ef | grep mysql                             # 查看了一下，突然想起来，本地我也有安装 mysql 可能有冲突。果断将之前安装的 docker mysql 删除，重新指定一个新的端口，用 DBeaver 链接，成功！

# 通过客户端创建一个新的数据库 new_test, 在本地映射的 data 目录下 ls 一下，可以看到新数据库文件可以同步创建
# > ~/tmp/mydb/data ls
# auto.cnf           ca.pem             client-key.pem     ib_logfile0        ibdata1            mysql              performance_schema public_key.pem     server-key.pem
# ca-key.pem         client-cert.pem    ib_buffer_pool     ib_logfile1        ibtmp1             new_test           private_key.pem    server-cert.pem    sys

# 删除容器，本地文件依然存在！
```

## 具名挂载 Vs 匿名挂载

具名挂载和匿名挂载是 `docker run` 命令中 `-v` 参数不同的使用情况。在明确指定挂载路径时(比如之前 mysql 和 tomcat 测试时的挂载指定), 通过 `docker volume ls` 可以看到是不会生产临时文件夹的

```bash
docker volume rm $(docker volume ls -q)             # 参考 rm 示例，删除所有的 volume, 准备测试环境

# -v path_in_container                              # 匿名挂载, 不指定宿主机挂载目录
docker run -d --name my-tomcat01 -P -v /usr/local/tomcat/webapps.dist tomcat

docker volume ls                                    # 查看卷情况
# DRIVER    VOLUME NAME
# local     0cd33950f5d8a050e61e58eaddae66b397db7b0e6968d40a1908a469c8386b03


# -v name:path_in_container                         # 具名挂载, 名字:容器中挂载点地址
docker run -d --name my-tomcat02 -P -v tomcat02-volume:/usr/local/tomcat/webapps.dist tomcat
# DRIVER    VOLUME NAME
# local     0cd33950f5d8a050e61e58eaddae66b397db7b0e6968d40a1908a469c8386b03
# local     tomcat02-volum

docker volume inspect juming-nginx                  # 使用 inspect 查看挂载点具体信息
# [
#     {
#         "CreatedAt": "2021-04-26T12:20:25Z",
#         "Driver": "local",
#         "Labels": null,
#         "Mountpoint": "/var/lib/docker/volumes/tomcat02-volume/_data",
#         "Name": "tomcat02-volume",
#         "Options": null,
#         "Scope": "local"
#     }
# ]

# 总结：
#   -v path-in-container                  # 匿名挂载
#   -v volume-name:path-in-container      # 具名挂载
#   -v host-path:path-in-container        # 指定路径挂载

# 其他使用方式，加 ro/rw 参数：
#   -v host-path:path-in-container:ro/rw
#       ro: read only, 只能通过宿主机改变，容器内部不能改变
#       rw: read and write, 默认的权限设置
```

## 初识 Dockerfile

用来构建 docker 镜像的构建文件

```dockerfile
# 新建文件写入如下内容
FROM centos

VOLUME ["volume01", "volume02"]

CMD echo "-----end-----"
CMD /bin/bash
```
```bash
docker build -f dfile -t my/centos:1.0 .
# docker images 查看自建的镜像
docker images
# REPOSITORY                                          TAG            IMAGE ID       CREATED         SIZE
# ...
# my/centos                                           1.0            f75a47123694   4 months ago    209MB
# ...

# 启动测试
docker run -it f75a47123694  /bin/bash
# ls 查看挂载卷
# drwxr-xr-x   2 root root 4096 Apr 22 12:42 volume01
# drwxr-xr-x   2 root root 4096 Apr 22 12:42 volume02

# 在 volume1 中新建文件 echo "new" >> new_file.txt
docker inspect 3996ebe1b343
# "Mounts": [
#             {
#                 "Type": "volume",
#                 "Name": "4c0081e951fc91397280afa09c9f5928115850f9ccaa17468b301c4aedc00bca",
#                 "Source": "/var/lib/docker/volumes/4c0081e951fc91397280afa09c9f5928115850f9ccaa17468b301c4aedc00bca/_data",
#                 "Destination": "volume02",
#                 "Driver": "local",
#                 "Mode": "",
#                 "RW": true,
#                 "Propagation": ""
#             },
#             {
#                 "Type": "volume",
#                 "Name": "33e8e61250b705ab4c2fba0972be84a541647c5aa327e5e3055431be836054b9",
#                 "Source": "/var/lib/docker/volumes/33e8e61250b705ab4c2fba0972be84a541647c5aa327e5e3055431be836054b9/_data",
#                 "Destination": "volume01",
#                 "Driver": "local",
#                 "Mode": "",
#                 "RW": true,
#                 "Propagation": ""
#             }
#         ]
```

## 数据卷容器

多个容器之间同步数据

```bash
# 启动自制容器作为父容器
ocker run -it --name docker01 my/centos:1.0
# 启动子容器
docker run -it --name docker02 --volumes-from docker01 my/centos:1.0
# docker01 下的 volume01 中新建文件，docker02 下的对应目录也有这个文件
# [root@d8c3cdf6d43b volume01]# echo "123" >> new_in_01.txt
# [root@d8c3cdf6d43b volume01]# ls
# new_in_01.txt
#
# [root@aee652f1fda3 /]# cd volume01
# [root@aee652f1fda3 volume01]# ls
# new_in_01.txt
# 重复以上操作新建的镜像都会同步文件

# 删除父容器，自容器中同步的文件依旧存在
docker rm -f docker01
```

结论：
* 容器之间配置信息传递，数据卷容器的生命周期一直持续到没有容器使用为止
* 一旦持久化到本地，本地数据是不会删除的

## Dockerfile

构建 Docker 镜像的文件，包含命令参数的脚本

步骤：

1. 创建 Dockerfile 文件
2. docker build 构建镜像
3. docker run 运行镜像
4. docker push 发布镜像

## Dockerfile 构建过程

1. 每个保留关键字都必须是大写的字母
2. 执行顺序从上倒下
3. `#` 表示注释
4. 每个指令都会创建提交一个新的镜像层，并提交

## Dockerfile 指令

```Dockerfile
FROM  # 基础镜像，起点
MAINTAINER # 作者
RUN # 镜像构建的时候需要运行的命令
ADD # 步骤，比如添加tomcat 压缩包
WORKDIR # 镜像工作目录
VOLUME # 挂载目录
EXPOSE # 暴露端口
CMD # 指定容器启动的时候运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT # 指定容器启动时运行的命令，可以追加命令
ONBUILD # 当构建一个被继承的 Dockerfile 就会运行 ONBUILD指令，触发指令
COPY # 类似 ADD， 将文件拷贝到镜像中
ENV # 设置环境变量
```

测试：

构建自己的 centos

编写 dockerfile

```dockerfile
FROM centos
MAINTAINER jzheng<jzheng@my.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "----end----"
CMD /bin/bash
```

运行构建命令

```txt
> docker build -f mydockerfile -t mycentos:0.1 .
[+] Building 22.6s (8/8) FINISHED                                                                                                              
 => [internal] load build definition from mydockerfile                                                                                    0.0s
 => => transferring dockerfile: 250B                                                                                                      0.0s
 => [internal] load .dockerignore                                                                                                         0.0s
 => => transferring context: 2B                                                                                                           0.0s
 => [internal] load metadata for docker.io/library/centos:latest                                                                          0.0s
 => CACHED [1/4] FROM docker.io/library/centos                                                                                            0.0s
 => [2/4] WORKDIR /usr/local                                                                                                              0.0s
 => [3/4] RUN yum -y install vim                                                                                                         19.3s
 => [4/4] RUN yum -y install net-tools                                                                                                    2.8s
 => exporting to image                                                                                                                    0.4s 
 => => exporting layers                                                                                                                   0.4s 
 => => writing image sha256:1fa2eebe33e71576555379a1f113cdc8a7a4023f1c0004f9a2b988540fcaa738                                              0.0s 
 => => naming to docker.io/library/mycentos:0.1                                                                                           0.0s 
```

运行测试

```bash
> docker run -it  --name osfromfile01 mycentos:0.1
# [root@b0ae777b8acf local]# pwd
# /usr/local
# 起始目录已经和设定的一样发生了变化
# 输入 ifconfig 和 vim 也能正常运行
```

查看 image 构建历史, 可以查看热门 image 学习构建过程

```txt
> docker history mycentos:0.1
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
1fa2eebe33e7   13 minutes ago   CMD ["/bin/sh" "-c" "/bin/bash"]                0B        buildkit.dockerfile.v0
<missing>      13 minutes ago   CMD ["/bin/sh" "-c" "echo \"----end----\""]     0B        buildkit.dockerfile.v0
<missing>      13 minutes ago   CMD ["/bin/sh" "-c" "echo $MYPATH"]             0B        buildkit.dockerfile.v0
<missing>      13 minutes ago   EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
<missing>      13 minutes ago   RUN /bin/sh -c yum -y install net-tools # bu…   14.4MB    buildkit.dockerfile.v0
<missing>      13 minutes ago   RUN /bin/sh -c yum -y install vim # buildkit    58.1MB    buildkit.dockerfile.v0
<missing>      14 minutes ago   WORKDIR /usr/local                              0B        buildkit.dockerfile.v0
<missing>      14 minutes ago   ENV MYPATH=/usr/local                           0B        buildkit.dockerfile.v0
<missing>      14 minutes ago   MAINTAINER jzheng<jzheng@sap.com>               0B        buildkit.dockerfile.v0
<missing>      4 months ago     /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      4 months ago     /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
<missing>      4 months ago     /bin/sh -c #(nop) ADD file:bd7a2aed6ede423b7…   209MB  
```

## CMD Vs ENTRYPOINT

CMD 只有最后一个命令会生效 由下面的 file 构建的 image, run 时只会输出 2

```dockerfile
FROM centos
CMD echo "1"
CMD echo "2"
```

```dockerfile
# filename: t_cmd, 测试 cmd
FROM centos
CMD ["ls", "-a"]
```

构建运行, run 之后终端输出 `ls -a`
```txt
> docker build -f t_cmd  -t cmdtest .     

[+] Building 0.1s (5/5) FINISHED                                                                                                                 
 => [internal] load build definition from t_cmd                                                                                             0.0s
 => => transferring dockerfile: 65B                                                                                                         0.0s
 => [internal] load .dockerignore                                                                                                           0.0s
 => => transferring context: 2B                                                                                                             0.0s
 => [internal] load metadata for docker.io/library/centos:latest                                                                            0.0s
 => CACHED [1/1] FROM docker.io/library/centos                                                                                              0.0s
 => exporting to image                                                                                                                      0.0s
 => => exporting layers                                                                                                                     0.0s
 => => writing image sha256:be8cf7de876380f7f60e83475b483161f43687087acd253990789c607f3b9848                                                0.0s
 => => naming to docker.io/library/cmdtest                                                                                                  0.0s

> docker run be8cf7de8                                     
.
..
.dockerenv
bin
dev
etc
home
...
```

如果想要追加 `l` 给出 `ls -al` 的效果怎么办？直接在 run 后接参数会报错

```txt
> docker run be8cf7de8 -l 
docker: Error response from daemon: OCI runtime create failed: container_linux.go:370: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container: context canceled
```

需要输入全命令 `docker run be8cf7de8 ls -al` 才行

如果想要直接接命令参数，可以用 ENTRYPOINT

```dockerfile
FROM centos
ENTRYPOINT ["ls", "-a"]
```
构建镜像并运行 `docker run 5198b187e -l` 追加命令成功, 直接拼接在 entrypoint 后

## 实战： 制作 Tomcat 镜像

1. 准备镜像文件 + tomcat压缩包 + JDK压缩包
2. 编写 dockerfile 文件

Google 搜索名字可直接下载压缩包 `jdk-8u202-linux-x64.tar.gz` + `apache-tomcat-9.0.22.tar.gz`

```dockerfile
FROM centos

LABEL maintainer="jzheng@aa.com"

COPY readme.txt /usr/local/readme.txt

ADD jdk-8u202-linux-x64.tar.gz /usr/local/

ADD apache-tomcat-9.0.22.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_11
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.22
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.22/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.22/bin/logs/catalina.out
```

构建镜像 `docker build -t diytomcat .`, 由于使用官方标准的名字 Dockerfile 就不需要用 -f 参数了, 构建 log 如下

```txt
[+] Building 119.9s (11/11) FINISHED                                                                                                             
 => [internal] load build definition from Dockerfile                                                                                        0.0s
 => => transferring dockerfile: 677B                                                                                                        0.0s
 => [internal] load .dockerignore                                                                                                           0.0s
 => => transferring context: 2B                                                                                                             0.0s
 => [internal] load metadata for docker.io/library/centos:latest                                                                            0.0s
 => [internal] load build context                                                                                                           3.6s
 => => transferring context: 205.02MB                                                                                                       3.6s
 => CACHED [1/6] FROM docker.io/library/centos                                                                                              0.0s
 => [2/6] COPY readme.txt /usr/local/readme.txt                                                                                             0.3s
 => [3/6] ADD jdk-8u202-linux-x64.tar.gz /usr/local/                                                                                        4.8s
 => [4/6] ADD apache-tomcat-9.0.22.tar.gz /usr/local/                                                                                       0.4s
 => [5/6] RUN yum -y install vim                                                                                                          109.1s
 => [6/6] WORKDIR /usr/local                                                                                                                0.0s
 => exporting to image                                                                                                                      1.7s
 => => exporting layers                                                                                                                     1.7s
 => => writing image sha256:4be1d03815380be7ca336c90e33b928890b5c604d646646b6087303385b0c8c0                                                0.0s 
 => => naming to docker.io/library/diytomcat                                                                                                0.0s 
```

启动镜像，并为之挂载节点 `docker run -d -p 9090:8080 --name mytomcat -v /Users/i306454/tmp/tmount/test:/usr/local/apache-tomcat-9.0.22/webapps/test -v /Users/i306454/tmp/tmount/tomcatlogs:/usr/local/apache-tomcat-9.0.22/logs diytomcat`

查看本地 logs 文件，报错

```txt
> cat catalina.out 
/usr/local/apache-tomcat-9.0.22/bin/catalina.sh: line 464: /usr/local/jdk1.8.0_11/bin/java: No such file or directory
```

检查后发现我下载的是 `jdk-8u202-linux-x64` 和视频上的不一样，很多地方变量都错了，改了重新 build 一下

突发奇想：虽然配置错了，但是我其实是可以直接进到终端重新配置使 Java 生效的，但是回头整个 follow 过了一下，发现还是不行，无法暴露端口。。。

再次查看 logs 日志，并访问 localhost:9090 tomcat 启动成功 (●°u°●)​ 」

在本地 test 目录下创建测试用页面

```txt
.
├── WEB-INF
│   └── web.xml
└── index.jsp

// index.jsp 内容如下
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<title>hello, docker</title>
<body>
<h2>Hello World!</h2>
 <%System.out.println("-------my test web logs--------");%>
</body>

// web.xml 内容如下
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4" 
    xmlns="http://java.sun.com/xml/ns/j2ee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
        http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
</web-app>
```

直接访问 localhost:9090/test 可以看到新写的页面显示成功, logs 下的 catalina.out 会输出 jsp 里的打印信息

## Docker 网络详解

ip addr

lo: 本机回环地址
eh0: 阿里云内网地址
docker0: docker0地址

启动测试容器

docker run -d -P --name tomcat01 tomcat

输入 ip addr 查看网络配置, 容器启动时会得到 eth0@if65 的 IP 地址

```txt
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN group default qlen 1000
    link/tunnel6 :: brd ::
64: eth0@if65: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

ping 172.17.0.2 测试, mac 不能 ping 通，不过视频用的 Linux 可以，应该时 OS 差异导致的

docker0 ip 为 172.17.0.1

原理

1. 我们每启动一个 docker 容器，docker 就会给 docker 容器分配一个 ip，我们只要安装了 docker 就会有一个 docker0，采用桥接模式，使用 veth-pair 技术。

再次宿主机终端输入 ip addr 可以看到有一个新的网卡 vethcxxx@ifxx 的网卡

新建容器生产的网卡都是成对出现的，这里采用 veth-pair 技术，一端连着协议，一端彼此相连

veth-pair 充当一个桥梁，链接各种虚拟网络设备

起两个 tomcat 容器也可以互相 ping 通

docker0 相当于一个虚拟路由器, 通信模型如下

![docker network](docker_network.png)

tomcat01 和 tomcat02 都是公用一个路由 - docker0, 所有容器不指定网络的情况下，都是 docker0 路由，docker 会给我们的容器分配默认可用 ip

255.255.0.1/16: 16 表示前 16 为同一个网络

![docker network02](docker_network01.png)

Docker 中所有的网络接口都是虚拟的，虚拟的转发效率高

> 思考：我们编写一个微服务， database url=ip...，项目不重启，数据库 ip 换掉了，配置就失效了。我们是否可以通过指定名字进行访问 --link

默认通过 name 是不能 ping 通的

```txt
> docker exec -it tomcat01 ping tomcat02
ping: tomcat02: Name or service not known
```

启动容器时加入 --link 参数即可实现上述效果

```txt
> docker run -d -P --name tomcat03 --link tomcat02 tomcat
4ccaa4545718ec75ba1d9c1a9d1941f3185502b4e754a6c6f940db353ea99d1c

> docker exec -it tomcat03 ping tomcat02                
PING tomcat02 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.348 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.282 ms

# 但是反向是 ping 不通的！！?
> docker exec -it tomcat02 ping tomcat03                
ping: tomcat03: Name or service not known
```

使用 docker network ls 查看当前网络配置

```txt
> docker network ls

NETWORK ID     NAME                      DRIVER    SCOPE
a6af0500c961   bizx-docker-dev_default   bridge    local
39a92d7cce05   bridge                    bridge    local
301b50497bc1   hana2_bizxdev             bridge    local
28c768cdf8c2   host                      host      local
3da61f734ed6   none                      null      local
```

查看桥接网卡信息

```bash
docker network inspect 39a92d7cce05

[
    {
        "Name": "bridge",
        "Id": "39a92d7cce055ead7bd2ba06365c5ad20a381730cba57137850e619eccdb993b",
        "Created": "2021-04-22T11:01:08.474840175Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16", <- 最多 255*255 个地址
                    "Gateway": "172.17.0.1" <- 默认网关 docker0
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": { <- 新建容器 tomcat01-03 信息
            "3bf7596cfde91a8a668621c750f744df4163aa4a3d836b7c3779f5cbe292e8ed": {
                "Name": "tomcat01",
                "EndpointID": "bc46d41fdbade310124ee65b5a1e1e8959d51ecb05f4441339a1da6cebd9daaa",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "4ccaa4545718ec75ba1d9c1a9d1941f3185502b4e754a6c6f940db353ea99d1c": {
                "Name": "tomcat03",
                "EndpointID": "64bffe5af0e30814d0b947edf3fe27d27db1f82cb2288b22ed1f2f3af60e38a9",
                "MacAddress": "02:42:ac:11:00:04",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            },
            "c5012b3643975869e49facb62d6d68f1c5153ea3cfa3f4a5fa36b91073aae07a": {
                "Name": "tomcat02",
                "EndpointID": "8c9fef294e0fd087ebb440f071d41440375c04603d4575b6eeece0e91e733dac",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

查看 tomcat03 的 host 配置

```bash
docker exec -it tomcat03 cat /etc/hosts

# 127.0.0.1       localhost
# ::1     localhost ip6-localhost ip6-loopback
# fe00::0 ip6-localnet
# ff00::0 ip6-mcastprefix
# ff02::1 ip6-allnodes
# ff02::2 ip6-allrouters
# 172.17.0.3      tomcat02 c5012b364397 < host 中直接指定的 ip - host 的对应关系，域名劫持
# 172.17.0.4      4ccaa4545718

docker ps

# CONTAINER ID   IMAGE     COMMAND             CREATED             STATUS             PORTS                     NAMES
# 4ccaa4545718   tomcat    "catalina.sh run"   15 minutes ago      Up 15 minutes      0.0.0.0:55004->8080/tcp   tomcat03
# c5012b364397   tomcat    "catalina.sh run"   17 minutes ago      Up 17 minutes      0.0.0.0:55003->8080/tcp   tomcat02
# 3bf7596cfde9   tomcat    "catalina.sh run"   About an hour ago   Up About an hour   0.0.0.0:55002->8080/tcp   tomcat01
```

本质：--link 就是在 hosts 中增加了一个域名劫持效果，但是现在这种做法已经**不推荐了！！！**

## 自定义网络

容器互联

```bash
> docker network ls     

NETWORK ID     NAME                      DRIVER    SCOPE
a6af0500c961   bizx-docker-dev_default   bridge    local
39a92d7cce05   bridge                    bridge    local
301b50497bc1   hana2_bizxdev             bridge    local
28c768cdf8c2   host                      host      local
3da61f734ed6   none                      null      local
```

网络模式

* brige: 桥接模式 - 默认
* none: 不配置
* host: 和宿主机共享网络
* container: 容器网络连通 - 用的少，局限大

cmd: `docker run -d -P --name tomcat01 tomcat` 其实是默认带有 `--net bridge` 的参数的

```bash
# 创建自定义网络
# --driver bridge 桥接模式
# --subnet 192.168.0.0/16 子网掩码
# --gateway 192.168.0.1 网关
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
# 7751a2b22fafcf8d7c4b5080cc15e46cb798f137f469826a547dd4bcf68f79d7

docker network ls                                                                        
# NETWORK ID     NAME                      DRIVER    SCOPE
# ...
# 28c768cdf8c2   host                      host      local
# 7751a2b22faf   mynet                     bridge    local
# ...

docker network inspect mynet       
# [
#     {
#         "Name": "mynet",
#         "Id": "7751a2b22fafcf8d7c4b5080cc15e46cb798f137f469826a547dd4bcf68f79d7",
#         "Created": "2021-04-24T08:43:56.4363152Z",
#         "Scope": "local",
#         "Driver": "bridge",
#         "EnableIPv6": false,
#         "IPAM": {
#             "Driver": "default",
#             "Options": {},
#             "Config": [
#                 {
#                     "Subnet": "192.168.0.0/16",
#                     "Gateway": "192.168.0.1"
#                 }
#             ]
#         },
#         "Internal": false,
#         "Attachable": false,
#         "Ingress": false,
#         "ConfigFrom": {
#             "Network": ""
#         },
#         "ConfigOnly": false,
#         "Containers": {},
#         "Options": {},
#         "Labels": {}
#     }
# ]

# 添加容器到自定义网络
docker run -d -P --name tomcat01 --net mynet tomcat         
docker run -d -P --name tomcat02 --net mynet tomcat

# 再次查看 mynet 信息可以看到新建容器已经加入到网络中
docker network inspect mynet                       
# [
#  ...   
# "Containers": {
#     "063c3bfb96d0328c67fb411944ca650cc2f1e4fac7ec00137bc35306f2da76ed": {
#         "Name": "tomcat01",
#         "EndpointID": "3fee6c414247c987f5f17dea54def8c3b162615f5d78c9fdaa81d7697057e9c4",
#         "MacAddress": "02:42:c0:a8:00:02",
#         "IPv4Address": "192.168.0.2/16",
#         "IPv6Address": ""
#     },
#     "163204a683461fef05b51b5eedd871dd9829fe5a78824e934edcdff2b83b31e7": {
#         "Name": "tomcat02",
#         "EndpointID": "2cc8c11a10b6bf3afedc8f31ee01576407cb1c1885ab5834465ca08d7af2f4b7",
#         "MacAddress": "02:42:c0:a8:00:03",
#         "IPv4Address": "192.168.0.3/16",
#         "IPv6Address": ""
#     }
# }
# ...
# ]

# 重复之前 --link 的实验，容器间通过 name 互相 ping. 自建网络虽然没有特殊设置，但是可以直接 ping 通
docker exec -it tomcat02 ping tomcat01 
# PING tomcat01 (192.168.0.2) 56(84) bytes of data.
# 64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.206 ms
# 64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.294 ms

docker exec -it tomcat01 ping tomcat02
# PING tomcat02 (192.168.0.3) 56(84) bytes of data.
# 64 bytes from tomcat02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.170 ms
# 64 bytes from tomcat02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.290 ms

# 自定义网络 docker 已经帮我们维护好了对应关系，推荐使用
# 比如 redis, mysql 集群网络互相隔离，保证集群的安全和健康
```

## 网络联通

如果让一个容器连接到另一个网络，比如 docker0 中的 容器连接到 mynet

```bash
# 默认 docker0 网络下新建测试容器 tomcat03
docker run -d -P --name tomcat03 tomcat 

docker network --help 
# Usage:  docker network COMMAND

# Manage networks

# Commands:
#   connect     Connect a container to a network <- 我们想要的命令
#   create      Create a network
#   disconnect  Disconnect a container from a network
#   inspect     Display detailed information on one or more networks
#   ls          List networks
#   prune       Remove all unused networks
#   rm          Remove one or more networks

# Run 'docker network COMMAND --help' for more information on a command.

# 使用方法
docker network connect --help 
# Usage:  docker network connect [OPTIONS] NETWORK CONTAINER

# Connect a container to a network

# Options:
#       --alias strings           Add network-scoped alias for the container
#       --driver-opt strings      driver options for the network
#       --ip string               IPv4 address (e.g., 172.30.100.104)
#       --ip6 string              IPv6 address (e.g., 2001:db8::33)
#       --link list               Add link to another container
#       --link-local-ip strings   Add a link-local address for the container

docker network connect mynet tomcat03

# 再次查看 mynet 信息，可以看到 tomcat3 已经加入网络，这个就是一个容器两个地址
docker network inspect mynet
# "Containers": {
#     "063c3bfb96d0328c67fb411944ca650cc2f1e4fac7ec00137bc35306f2da76ed": {
#         "Name": "tomcat01",
#         "EndpointID": "3fee6c414247c987f5f17dea54def8c3b162615f5d78c9fdaa81d7697057e9c4",
#         "MacAddress": "02:42:c0:a8:00:02",
#         "IPv4Address": "192.168.0.2/16",
#         "IPv6Address": ""
#     },
#     "163204a683461fef05b51b5eedd871dd9829fe5a78824e934edcdff2b83b31e7": {
#         "Name": "tomcat02",
#         "EndpointID": "2cc8c11a10b6bf3afedc8f31ee01576407cb1c1885ab5834465ca08d7af2f4b7",
#         "MacAddress": "02:42:c0:a8:00:03",
#         "IPv4Address": "192.168.0.3/16",
#         "IPv6Address": ""
#     },
#     "ca3d7a33b27b4c02392059c4505c9b4b73eeb3fbe16d28483400c60b5ea17643": {
#         "Name": "tomcat03",
#         "EndpointID": "b0de06f6bac08115b9df8cea1c08de53f675ec80ea675be9f3b1164bd4d069e2",
#         "MacAddress": "02:42:c0:a8:00:04",
#         "IPv4Address": "192.168.0.4/16",
#         "IPv6Address": ""
#     }
# }

# tomcat01, 03 互 ping 测试
docker exec -it tomcat01 ping tomcat03                              
# PING tomcat03 (192.168.0.4) 56(84) bytes of data.
# 64 bytes from tomcat03.mynet (192.168.0.4): icmp_seq=1 ttl=64 time=0.131 ms
# 64 bytes from tomcat03.mynet (192.168.0.4): icmp_seq=2 ttl=64 time=0.123 ms

docker exec -it tomcat03 ping tomcat01
# PING tomcat01 (192.168.0.2) 56(84) bytes of data.
# 64 bytes from tomcat01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.085 ms
```

## 实战：部署 Redis 集群

我 redis 的环境还没有搞好，等看过 redis 的课程之后回头再看把

部署三主三从节点

![redis](redis.png)

```bash
# 创建网络
docker network create redis --subnet 172.38.0.0/16

# 脚本创建六个 redis 配置
# mac 环境下一些命令支持有问题，文件内容没有塞进去...
for port in $(seq 1 6); \
do \
mkdir -p /Users/i306454/tmp/mydata/redis/node-${port}/conf
touch /Users/i306454/tmp/mydata/redis/node-${port}/conf/redis.conf
cat  EOF /Users/i306454/tmp/mydata/redis/node-${port}/conf/redis.conf
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

# 创建 redis 节点
docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /Users/i306454/tmp/mydata/redis/node-1/data:/data \
-v /Users/i306454/tmp/mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

# 依次创建 2-6 节点
docker run -p 637${n}:6379 -p 1637${n}:16379 --name redis-${n} \
-v /Users/i306454/tmp/mydata/redis/node-${n}/data:/data \
-v /Users/i306454/tmp/mydata/redis/node-${n}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${n} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

# 进入 redis 容器
docker exec -it redis-1 /bin/sh

# 创建集群
redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
# >>> Performing hash slots allocation on 6 nodes...
# Master[0] -> Slots 0 - 5460
# Master[1] -> Slots 5461 - 10922
# Master[2] -> Slots 10923 - 16383
# Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
# Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
# Adding replica 172.38.0.14:6379 to 172.38.0.13:6379
# M: 4abbca8e1511fc4a04e8b410e35a93af1392bc62 172.38.0.11:6379
#    slots:[0-5460] (5461 slots) master
# M: 6ff00ea4f03e997b831c2e7d3150c7399c5fb44c 172.38.0.12:6379
#    slots:[5461-10922] (5462 slots) master
# M: 0d6d16438bcb68bf89109b2e91dc6b71208ea113 172.38.0.13:6379
#    slots:[10923-16383] (5461 slots) master
# S: c47e77a26e9c1186b489003c00f1a9c647e913c2 172.38.0.14:6379
#    replicates 0d6d16438bcb68bf89109b2e91dc6b71208ea113
# S: 69fa6ac41672b325cfce023353ea3a35181ca873 172.38.0.15:6379
#    replicates 4abbca8e1511fc4a04e8b410e35a93af1392bc62
# S: c7b681f08fbbe568c5fcb2bddf88660ec3d217bf 172.38.0.16:6379
#    replicates 6ff00ea4f03e997b831c2e7d3150c7399c5fb44c
Can I set the above configuration? (type 'yes' to accept): yes
# >>> Nodes configuration updated
# >>> Assign a different config epoch to each node
# >>> Sending CLUSTER MEET messages to join the cluster
# Waiting for the cluster to join
# ....
# >>> Performing Cluster Check (using node 172.38.0.11:6379)
# M: 4abbca8e1511fc4a04e8b410e35a93af1392bc62 172.38.0.11:6379
#    slots:[0-5460] (5461 slots) master
#    1 additional replica(s)
# S: c47e77a26e9c1186b489003c00f1a9c647e913c2 172.38.0.14:6379
#    slots: (0 slots) slave
#    replicates 0d6d16438bcb68bf89109b2e91dc6b71208ea113
# M: 0d6d16438bcb68bf89109b2e91dc6b71208ea113 172.38.0.13:6379
#    slots:[10923-16383] (5461 slots) master
#    1 additional replica(s)
# M: 6ff00ea4f03e997b831c2e7d3150c7399c5fb44c 172.38.0.12:6379
#    slots:[5461-10922] (5462 slots) master
#    1 additional replica(s)
# S: 69fa6ac41672b325cfce023353ea3a35181ca873 172.38.0.15:6379
#    slots: (0 slots) slave
#    replicates 4abbca8e1511fc4a04e8b410e35a93af1392bc62
# S: c7b681f08fbbe568c5fcb2bddf88660ec3d217bf 172.38.0.16:6379
#    slots: (0 slots) slave
#    replicates 6ff00ea4f03e997b831c2e7d3150c7399c5fb44c
# [OK] All nodes agree about slots configuration.
# >>> Check for open slots...
# >>> Check slots coverage...
# [OK] All 16384 slots covered.

# 查看集群信息
redis-cli -c 
# 127.0.0.1:6379> cluster info
# cluster_state:ok
# cluster_slots_assigned:16384
# cluster_slots_ok:16384
# cluster_slots_pfail:0
# cluster_slots_fail:0
# cluster_known_nodes:6
# cluster_size:3
# cluster_current_epoch:6
# cluster_my_epoch:1
# cluster_stats_messages_ping_sent:203
# cluster_stats_messages_pong_sent:201
# cluster_stats_messages_sent:404
# cluster_stats_messages_ping_received:196
# cluster_stats_messages_pong_received:203
# cluster_stats_messages_meet_received:5
# cluster_stats_messages_received:404

127.0.0.1:6379> cluster noes 
# (error) ERR Unknown subcommand or wrong number of arguments for 'noes'. Try CLUSTER HELP.
# 127.0.0.1:6379> cluster nodes 
# c47e77a26e9c1186b489003c00f1a9c647e913c2 172.38.0.14:6379@16379 slave 0d6d16438bcb68bf89109b2e91dc6b71208ea113 0 1619257734941 4 connected
# 0d6d16438bcb68bf89109b2e91dc6b71208ea113 172.38.0.13:6379@16379 master - 0 1619257736000 3 connected 10923-16383
# 6ff00ea4f03e997b831c2e7d3150c7399c5fb44c 172.38.0.12:6379@16379 master - 0 1619257736471 2 connected 5461-10922
# 4abbca8e1511fc4a04e8b410e35a93af1392bc62 172.38.0.11:6379@16379 myself,master - 0 1619257734000 1 connected 0-5460
# 69fa6ac41672b325cfce023353ea3a35181ca873 172.38.0.15:6379@16379 slave 4abbca8e1511fc4a04e8b410e35a93af1392bc62 0 1619257735453 5 connected
# c7b681f08fbbe568c5fcb2bddf88660ec3d217bf 172.38.0.16:6379@16379 slave 6ff00ea4f03e997b831c2e7d3150c7399c5fb44c 0 1619257735554 6 connected

127.0.0.1:6379> set a b
# -> Redirected to slot [15495] located at 172.38.0.13:6379
# OK
# 从 log 看出，值存到了 13 节点中，下面将 13 节点容器 stop, 通过 get 测试备份是否生效

# 开启一个新的终端，查看 13 节点信息
docker inspect redis
# "59ceffe5a78c6d013f62e44bd78b3e0b30b9716766bfe0d2944c3d4e55ad730a": {
#     "Name": "redis-3",
#     "EndpointID": "4e28b437aa3d725661616200271dab68152ebb3bf93b5e6d66af23e5f44710d6",
#     "MacAddress": "02:42:ac:26:00:0d",
#     "IPv4Address": "172.38.0.13/16",
#     "IPv6Address": ""
# }

docker stop redis-3
docker ps
# CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                                              NAMES
# c0ea4cc5bfc6   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   11 minutes ago   Up 11 minutes   0.0.0.0:6376->6379/tcp, 0.0.0.0:16376->16379/tcp   redis-6
# 4a89995a1cad   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes   0.0.0.0:6375->6379/tcp, 0.0.0.0:16375->16379/tcp   redis-5
# 32da42a0d210   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes   0.0.0.0:6374->6379/tcp, 0.0.0.0:16374->16379/tcp   redis-4
# d7d7751dc13d   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   14 minutes ago   Up 14 minutes   0.0.0.0:6372->6379/tcp, 0.0.0.0:16372->16379/tcp   redis-2
# 71fa6b723afd   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   17 minutes ago   Up 17 minutes   0.0.0.0:6371->6379/tcp, 0.0.0.0:16371->16379/tcp   redis-1

# 回到原来的终端进行 get 操作
172.38.0.13:6379> get a
Error: Operation timed out
# 好像他默认直接从原来的 13 节点拿了，服务停了，回到了 11 节点的 data 目录，再通过 redis-cli -c 进去集群终端 get a 信息从 14 节点，备份节点返回
/data # redis-cli -c 
127.0.0.1:6379> get a
# -> Redirected to slot [15495] located at 172.38.0.14:6379
# "b"

# 通过 nodes 命令查看节点信息可以看到 13 挂了 172.38.0.13:6379@16379 master,fail, slave 直接翻身农奴把歌唱
172.38.0.14:6379> cluster nodes 
# 6ff00ea4f03e997b831c2e7d3150c7399c5fb44c 172.38.0.12:6379@16379 master - 0 1619258378445 2 connected 5461-10922
# 0d6d16438bcb68bf89109b2e91dc6b71208ea113 172.38.0.13:6379@16379 master,fail - 1619258032356 1619258031341 3 connected
# c47e77a26e9c1186b489003c00f1a9c647e913c2 172.38.0.14:6379@16379 myself,master - 0 1619258377000 7 connected 10923-16383
# 4abbca8e1511fc4a04e8b410e35a93af1392bc62 172.38.0.11:6379@16379 master - 0 1619258377432 1 connected 0-5460
# c7b681f08fbbe568c5fcb2bddf88660ec3d217bf 172.38.0.16:6379@16379 slave 6ff00ea4f03e997b831c2e7d3150c7399c5fb44c 0 1619258377533 6 connected
# 69fa6ac41672b325cfce023353ea3a35181ca873 172.38.0.15:6379@16379 slave 4abbca8e1511fc4a04e8b410e35a93af1392bc62 0 1619258376415 5 connected
```

## SpringBoot 微服务打包 Docker 镜像

1. 构建 springboot 的 helloword 项目
2. 打包应用
3. 编写 dockerfile
4. 构建镜像
5. 发布运行

到 Spring Initializr 去打包一个 Hello Word demo 进行测试

Application 同级建一个 controller 文件夹，其下创建 HelloController

```java
@RestController
public class HelloController {
    @RequestMapping("hello")
    public String hello() {
        return "Hello from springboot.";
    }
}
```

右键运行 application，访问 localhost:8080/hello 得到字符串

maven task -> Lifecycle -> package 打包工程, jar 包会生成在 target 目录下

右键 open in terminal, 运行 `java -jar demo-0.0.1-SNAPSHOT.jar` 测试 jar 包是否能正常工作

在 target 目录下新建 Dockerfile

```Dockerfile
FROM java:8

COPY *.jar /app.jar

CMD ["--server.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

在 target 目录下 build image `docker build -t myspringapp .`

等待结束后查看新建是否成功

```bash
docker images 
# REPOSITORY                                          TAG                IMAGE ID       CREATED              SIZE
# myspringapp                                         latest             8b39ad3c5928   About a minute ago   660MB
```

测试运行

```bash
# 第一次忘了加 -P 导致容器端口没有暴露出来，直接就不能访问了
docker run -d -P --name spapp myspringapp

# 查看映射地址
docker ps
# CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                     NAMES
# 0b4da52a4593   myspringapp   "java -jar /app.jar …"   3 seconds ago   Up 2 seconds   0.0.0.0:55010->8080/tcp   spapp

# 访问测试
curl localhost:55010/hello
# Hello from springboot.
```

## Docker Compose

Dockerfile 单个容器，Docker Compose 定义运行多个容器

Using Compose is basically a three-step process:

1. Define your app’s environment with a Dockerfile so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Run `docker compose up` and the Docker compose command starts and runs your entire app. You can alternatively run docker-compose up using the docker-compose binary.

重要概念：

* 服务 services, 容器，应用（web, redis...）
* 项目 project，一组关联的容器

## 安装

Mac 的 docker-compose 工具集是和客户端整合在一起的，所以不需要单独安装，直接可以使用

```bash
docker-compose version
# docker-compose version 1.28.5, build c4eb3a1f
# docker-py version: 4.4.4
# CPython version: 3.9.0
# OpenSSL version: OpenSSL 1.1.1h  22 Sep 202
```

## 官方起步教程

1. 写应用 app
2. Dockerfile 应用打包为镜像
3. Docker-compose yaml 文件整合所有 services
4. 启动 compose 项目(docker-compose up .)

```txt
docker-compose up

# 创建网络
Creating network "composetest_default" with the default driver

# 根据 compose 文件构建
Building web
[+] Building 19.7s (13/13) FINISHED                                                                                  
 => [internal] load build definition from Dockerfile                                                            0.0s
 => => transferring dockerfile: 324B                                                                            0.0s
 => [internal] load .dockerignore                                                                               0.0s
 => => transferring context: 2B                                                                                 0.0s
 => resolve image config for docker.io/docker/dockerfile:1                                                      3.6s
 => docker-image://docker.io/docker/dockerfile:1@sha256:e2a8561e419ab1ba6b2fe6cbdf49fd92b95912df1cf7d313c3e2230a333fdbcc                                          1.5s
 => => resolve docker.io/docker/dockerfile:1@sha256:e2a8561e419ab1ba6b2fe6cbdf49fd92b95912df1cf7d313c3e2230a333fdbcc                                              0.0s
 => => sha256:e2a8561e419ab1ba6b2fe6cbdf49fd92b95912df1cf7d313c3e2230a333fdbcc 1.69kB / 1.69kB                  0.0s
 => => sha256:e3ee2e6b536452d876b1c5aa12db9bca51b8f52b2505178cae6d13e33daeed2b 528B / 528B                      0.0s
 => => sha256:86e43bba076d67c1a890cbc07813806b11eca53843dc643202d939b986c8c332 1.21kB / 1.21kB                  0.0s
 => => sha256:3cc8e449ce9f6e0752ede8f50a7334bf0c7b2d24d76da2ffae7aa6a729dd1da4 9.64MB / 9.64MB                  0.8s
 => => extracting sha256:3cc8e449ce9f6e0752ede8f50a7334bf0c7b2d24d76da2ffae7aa6a729dd1da4                       0.3s
 => [internal] load metadata for docker.io/library/python:3.7-alpine                                            2.7s
 => [1/6] FROM docker.io/library/python:3.7-alpine@sha256:3b0e1a61106a4c73d1253a86b7765b41d87d1122eb70f99c6de06f0b64edc434                                        2.2s
 => => resolve docker.io/library/python:3.7-alpine@sha256:3b0e1a61106a4c73d1253a86b7765b41d87d1122eb70f99c6de06f0b64edc434                                        0.0s
 => => sha256:a7ad1a75a9998a18ceb4b3e77ebc933525c48a5e9b1dd6abf258e86f537a7fbf 281.27kB / 281.27kB              0.6s
 => => sha256:37ce6546d5dd0143d2fd48adccb48cd0f46c5c2587ce49a53fbd9bd1c5816665 10.57MB / 10.57MB                1.1s
 => => sha256:3b0e1a61106a4c73d1253a86b7765b41d87d1122eb70f99c6de06f0b64edc434 1.65kB / 1.65kB                  0.0s
 => => sha256:d7c477920ed69ca5744ae133810c519a5ea72ab2da3edf542375d48e10742720 1.37kB / 1.37kB                  0.0s
 => => sha256:c46f62f378d72f9a78c4fb150000c479f2bf9095f5616b9f85a8387437e7592c 7.85kB / 7.85kB                  0.0s
 => => sha256:540db60ca9383eac9e418f78490994d0af424aab7bf6d0e47ac8ed4e2e9bcbba 2.81MB / 2.81MB                  0.5s
 => => extracting sha256:540db60ca9383eac9e418f78490994d0af424aab7bf6d0e47ac8ed4e2e9bcbba                       0.2s
 => => sha256:ec9e91bed5a295437fbb8a07b8af46fd69d7dd5beb7ac009c839292454bb3d31 234B / 234B                      1.0s
 => => sha256:c629b5f73da8f1113e9c8257d16fc65a95b812483506f56e418183e0d618f07e 2.16MB / 2.16MB                  1.3s
 => => extracting sha256:a7ad1a75a9998a18ceb4b3e77ebc933525c48a5e9b1dd6abf258e86f537a7fbf                       0.1s
 => => extracting sha256:37ce6546d5dd0143d2fd48adccb48cd0f46c5c2587ce49a53fbd9bd1c5816665                       0.5s
 => => extracting sha256:ec9e91bed5a295437fbb8a07b8af46fd69d7dd5beb7ac009c839292454bb3d31                       0.0s
 => => extracting sha256:c629b5f73da8f1113e9c8257d16fc65a95b812483506f56e418183e0d618f07e                       0.2s
 => [internal] load build context                                                                               0.0s
 => => transferring context: 1.08kB                                                                             0.0s
 => [2/6] WORKDIR /code                                                                                         0.1s
 => [3/6] RUN apk add --no-cache gcc musl-dev linux-headers                                                     3.8s
 => [4/6] COPY requirements.txt requirements.txt                                                                0.0s
 => [5/6] RUN pip install -r requirements.txt                                                                   4.7s
 => [6/6] COPY . .                                                                                              0.0s
 => exporting to image                                                                                          0.7s
 => => exporting layers                                                                                         0.7s
 => => writing image sha256:fa0cb4f4c056d1b79ddddb5ec6e21659cb3d84cad4a28d361775161cbe281811                    0.0s
 => => naming to docker.io/library/composetest_web                                                              0.0s
Successfully built fa0cb4f4c056d1b79ddddb5ec6e21659cb3d84cad4a28d361775161cbe281811
WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Pulling redis (redis:alpine)...
alpine: Pulling from library/redis
540db60ca938: Already exists
29712d301e8c: Pull complete
8173c12df40f: Pull complete
0be901b3c77d: Pull complete
c33773bf45b4: Pull complete
6eeb0c30f7e7: Pull complete
Digest: sha256:f9577ac6e68c70b518e691406f2bebee49d8db22118fc87bad3b39c16a1cb46e
Status: Downloaded newer image for redis:alpine
Creating composetest_web_1   ... done
Creating composetest_redis_1 ... done
Attaching to composetest_redis_1, composetest_web_1
redis_1  | 1:C 25 Apr 2021 06:45:13.848 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 25 Apr 2021 06:45:13.848 # Redis version=6.2.2, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 25 Apr 2021 06:45:13.848 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis_1  | 1:M 25 Apr 2021 06:45:13.849 * monotonic clock: POSIX clock_gettime
redis_1  | 1:M 25 Apr 2021 06:45:13.849 * Running mode=standalone, port=6379.
redis_1  | 1:M 25 Apr 2021 06:45:13.849 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis_1  | 1:M 25 Apr 2021 06:45:13.849 # Server initialized
redis_1  | 1:M 25 Apr 2021 06:45:13.850 * Ready to accept connections
web_1    |  * Serving Flask app "app.py"
web_1    |  * Environment: production
web_1    |    WARNING: This is a development server. Do not use it in a production deployment.
web_1    |    Use a production WSGI server instead.
web_1    |  * Debug mode: off
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web_1    | 172.20.0.1 - - [25/Apr/2021 06:45:42] "GET / HTTP/1.1" 200 -
web_1    | 172.20.0.1 - - [25/Apr/2021 06:45:42] "GET /favicon.ico HTTP/1.1" 404 -
web_1    | 172.20.0.1 - - [25/Apr/2021 06:45:46] "GET / HTTP/1.1" 200 -
```

```bash
docker ps
# CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                    NAMES
# 621dc8726fe9   composetest_web   "flask run"              8 minutes ago   Up 8 minutes   0.0.0.0:5000->5000/tcp   composetest_web_1
# 6945405369e2   redis:alpine      "docker-entrypoint.s…"   8 minutes ago   Up 8 minutes   6379/tcp                 composetest_redis_1

docker network ls
# NETWORK ID     NAME                      DRIVER    SCOPE
# 39a92d7cce05   bridge                    bridge    local
# f713400ac914   composetest_default       bridge    local <- compose 启动之后会创建一个对应的网络，和之前的 log 匹配

docker network inspect composetest_default
# "Containers": {
#     "621dc8726fe93265134ead255af6cd572f7a6d82a802a97edd53780248f749dd": {
#         "Name": "composetest_web_1",
#         "EndpointID": "000a2b25cc2ffc6b1d86eca556dd06cac05ff6bb70a2cd37d5cce4054559c993",
#         "MacAddress": "02:42:ac:14:00:02",
#         "IPv4Address": "172.20.0.2/16",
#         "IPv6Address": ""
#     },
#     "6945405369e252c5537743244ed60a7c740f713c20d1df4e0318cf929411f169": {
#         "Name": "composetest_redis_1",
#         "EndpointID": "857a5962c8220bd8972e6793c5ff521503b6e293d31155b8268a8949bfb4bb72",
#         "MacAddress": "02:42:ac:14:00:03",
#         "IPv4Address": "172.20.0.3/16",
#         "IPv6Address": ""
#     }
# },
```

注意点：app 中 redis 是通过域名绑定的 `cache = redis.Redis(host='redis', port=6379)` 并不是指定 IP，这里已经用到了 docker 里面的网络了

## YAML 规则

```txt
# 3层

version: '' # 版本
services: # 服务
    服务1: web
        # 服务配置
        images
        build
        network
        ...
    服务2: redis
        ...
    服务2: redis
        ...
# 其他配置 网络/卷，全局规则
volumes:
network:
```

## 实战

仿照官方的计数器，写一个 Java 版本的并 compose 启动

1. 写项目代码
2. dockerfile 构建镜像
3. docker-compose.yml 整合项目
4. 上传服务器 docker-compose up . 启动


通过 spring initializr 下载模版，将 spring web + spring data reactive redis 加入依赖

新建 controller 层代码, 目录结构如下

```txt
.
├── CounterApplication.java
└── controller
    └── HelloController.java
```

```java
@RestController
public class HelloController {

    @Autowired
    StringRedisTemplate redisTemplate;

    @GetMapping("/hello")
    public String hello() {
        Long views = redisTemplate.opsForValue().increment("views");
        return "hello, counter views: " + views;
    }
}
```

配置端口，由于配置的是 redis，所以本地启动的时候 redis 是会出问题的，docker 里才 OK

```application.properties
server.port=8080
spring.redis.host=redis
```

maven -> counter -> Lifecycle -> package 打包到 target folder 下

在 target 下新建 Dockerfile 和 docker-compose.yml

```Dockerfile
FROM java:8

COPY *.jar /app.jar

CMD ["--server.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

```yml
version: '3.8'
services:
  counterapp:
    build: .
    image: counterapp
    depends_on:
      - redis
    ports:
      - "8080:8080"
  redis:
    image: "library/redis:alpine"
```

这个有一个东西很奇怪，其实在上面的 Dockerfile 里我是没有指定 docker image 叫什么， 但是下面的 compose file 里直接就拿到了。可能是下面的 `build .` 的意思是用但前文件加下的 Dockerfile 构建，并取名叫 counterapp 

由于我本机就有环境，直接运行即可

```bash
cd target
docker-compose up

# network 下会显示 folder 前缀的新网络
docker network ls
# c2b31f70eead   target_default            bridge    local
```

log 输出正常，访问页面正常

`docker-compose --build` 重新构建

## Docker Swarm

## Raft 协议

保证大多数节点存活才可用，只要 >1， 集群则要求 > 3

## 搭建集群

* docker run 容器启动，不能扩容
* docker service 服务，可扩容

## 以后还要学 Go

## 问题

Q: 在最后的 spring 项目实战的 dockerfile 中我明明写了 EXPOSE 8080 为什么我在 run 的时候还要加 p 参数？
A: 查看了一下文档，发现这个 EXPOSE 只是起说明作用， run 的时候并不会自动暴露，但是他有一个好处，就是你写了之后，如果用 P 随机的模式的话，他会自动暴露

官方定义如下

```txt
EXPOSE 指令是声明容器运行时提供服务的端口，这只是一个声明，在容器运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；

另一个用处则是在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。
要将 EXPOSE 和在运行时使用 -p <宿主端口>:<容器端口> 区分开来。-p，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。
```
