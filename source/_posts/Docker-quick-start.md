---
title: Docker 弹射起步
date: 2021-04-15 19:59:17
categories:
- 弹射起步
tags:
- docker
---

## 虚拟机 Vs 容器

* 虚拟机运行整个系统，在系统上安装运行软件
* 容器内的应用直接运行在宿主机内，容器没有自己的内核，也没有虚拟硬件
* 每个容器互相隔离，都有属于自己的文件系统，互不影响

## DevOps

应用更快速的交付和部署

传统：一堆帮助文件，安装程序
Docker: 打包镜像发布测试，一键运行

更便捷的升级扩容

更简单的系统运维

更高效的计算资源利用：docker 是内核级别的虚拟化，可以在一个乌力吉上运行多个实例，性能压榨到极致

## 安装

## 当你输入 docker run hello-word 时发生了什么

拉镜像的 flow

## 底层原理

client 和 server 交互模型

## Docker 为什么比 VM 快

1. Docker 有比虚拟机更少的抽象出
2. Docker 利用的是宿主机内核，VM 需要自己加载操作系统

## 常用命令

```bash
# Docker 相关
docker version # 版本信息
docker info # 系统信息，container 数量，操纵系统等
docker --help

# 镜像相关
# docker pull image_name[:tag]
Host> docker pull mysql   # 没有指定 tag 就默认下载 latest 版本
Using default tag: latest
latest: Pulling from library/mysql
f7ec5a41d630: Already exists 
9444bb562699: Pull complete  # 分层下载， docker image 的核心，联合文件系统
6a4207b96940: Pull complete 
181cefd361ce: Pull complete 
8a2090759d8a: Pull complete 
15f235e0d7ee: Pull complete 
d870539cd9db: Pull complete 
493aaa84617a: Pull complete 
bfc0e534fc78: Pull complete 
fae20d253f9d: Pull complete 
9350664305b3: Pull complete 
e47da95a5aab: Pull complete 
Digest: sha256:04ee7141256e83797ea4a84a4d31b1f1bc10111c8d1bc1879d52729ccd19e20a # 签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest # 真实地址，等价 docker pull docker.io/library/mysql:latest

docker rmi -f img_id # 删除镜像
docker rmi -f $(docker images -aq) # 删除全部镜像

# 容器相关
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# 常用参数
--name="my_name" 指定容器名字
-d 后台运行
-it 交互方式运行，进去容器查看
-p 指定端口
    -p ip:主机端口:容器端口
    -p 主机端口:容器端口
    -p 容器端口
    容器端口
-P 随机指定端口

docker run -it centos /bin/bash # 启动并进入容器
docker ps # 运行中的容器
docker ps -a # 所有容器
docker ps -n=2 # 最近创建的2个容器

ctrl + p + q # 交互模式下推出容器并后台运行。mac 也是这个命令

docker rm container_id # 删除容器，不能删除正在运行的容器，除非加 -f
docker rm -f $(docker ps -qa) # 删除所有
docker ps -aq | xargs docker rm # 删除所有

docker start container_id
docker restart container_id
docker stop container_id
docker kill container_id # stop 报错了可以用这个强制杀进程

docker run -d centos # 后台运行
# 常见坑：docker 容器使用后台运行，必须要给一个前台进程，dock儿 发现没有应用就会自动停止
# nginx 容器启动后发现自己没有提供服务，就会立刻停止，ps 就看不见了

# 删除某个 image 的 containers, 这里有个小技巧，可以先通过 docker ps 输出一下，避免误删
docker ps -aq --filter ancestor=nginx
docker rm -f $(docker ps -aq --filter ancestor=nginx)

# 生成临时 log
docker run -d centos /bin/sh -c "while true;do echo testlog;sleep 1;done"
# -t 时间戳
# -f 一直打印
# --tail num 输出n条
docker logs -tf --tail 10 contains_id

docker top comtainer_id # 显示容器中的进程

docker inspect contains_id # 显示容器底层信息
[
    {
        "Id": "247d2a88573fdb2893a90a3d35275bfaa2889f7fa450d875456646ed684643d4",
        "Created": "2021-04-20T13:06:45.2632089Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true;do echo testlog;sleep 1;done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 72857,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-04-20T13:06:45.5636259Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/247d2a88573fdb2893a90a3d35275bfaa2889f7fa450d875456646ed684643d4/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/247d2a88573fdb2893a90a3d35275bfaa2889f7fa450d875456646ed684643d4/hostname",
        "HostsPath": "/var/lib/docker/containers/247d2a88573fdb2893a90a3d35275bfaa2889f7fa450d875456646ed684643d4/hosts",
        "LogPath": "/var/lib/docker/containers/247d2a88573fdb2893a90a3d35275bfaa2889f7fa450d875456646ed684643d4/247d2a88573fdb2893a90a3d35275bfaa2889f7fa450d875456646ed684643d4-json.log",
        "Name": "/elastic_newton",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/80783254f01fcdde559ac63ff7503d2dc317929d0328fb1c66846f9e519d98df-init/diff:/var/lib/docker/overlay2/548d80e0e272a3497edbc439f2a231886f5e890933f8f804c28080c2ecd64172/diff",
                "MergedDir": "/var/lib/docker/overlay2/80783254f01fcdde559ac63ff7503d2dc317929d0328fb1c66846f9e519d98df/merged",
                "UpperDir": "/var/lib/docker/overlay2/80783254f01fcdde559ac63ff7503d2dc317929d0328fb1c66846f9e519d98df/diff",
                "WorkDir": "/var/lib/docker/overlay2/80783254f01fcdde559ac63ff7503d2dc317929d0328fb1c66846f9e519d98df/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "247d2a88573f",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "while true;do echo testlog;sleep 1;done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "39c3e27c519197bb099217f5f767fd806ccc7705d4afca44e044df7def928c1b",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/39c3e27c5191",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "aa43be83ff078d4c2dbdff62a2e69217ef2e17c0585edfecdcda030dca3aef0f",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "2fbb6bb1ed5e760a8350664377ea726ffbf35fab4794d45926ab9f9f9bd28d8a",
                    "EndpointID": "aa43be83ff078d4c2dbdff62a2e69217ef2e17c0585edfecdcda030dca3aef0f",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]

docker exec -it contain_id /bin/bash # 进入容器，开启一个新的终端
docker attach contain_id # 进去容器，为当前正在执行的终端

# 拷贝文件到主机
docker cp container_id:path 目的主机路径 # sample: docker cp 247d2a88573f:/test.java .
```

## Nginx

```bash
docker search nginx # 搜索镜像
docker pull nginx # 下载镜像

# -d : 后台运行
# --name: 自定义容器名称
# -p: 指定端口号
docker run -d --name nginx01 -p 3344:80 nginx 
curl localhost:3344 
# 访问 nginx 测试是否成功启动, 返回页面如下
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# <style>
#     body {
#         width: 35em;
#         margin: 0 auto;
#         font-family: Tahoma, Verdana, Arial, sans-serif;
#     }
# </style>
# </head>
# <body>
# <h1>Welcome to nginx!</h1>
# <p>If you see this page, the nginx web server is successfully installed and
# working. Further configuration is required.</p>

# <p>For online documentation and support please refer to
# <a href="http://nginx.org/">nginx.org</a>.<br/>
# Commercial support is available at
# <a href="http://nginx.com/">nginx.com</a>.</p>

# <p><em>Thank you for using nginx.</em></p>
# </body>
# </html>

docker exec -it nginx01 /bin/bash # 进入容器
whereis nginx # 查看配置
```

## Tomcat 练习

```bash
# --rm: 一般用于测试，用完即删除
docker run -it --rm tomcat:9.0

docker run -d -p 3355:8080 --name tomcat01 tomcat
curl localhost:3355
# 访问失败
# <!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class="line" /><p><b>Type</b> Status Report</p><p><b>Description</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.</p><hr class="line" /><h3>Apache Tomcat/9.0.45</h3></body></html>

# 进入容器查看原因
docker exec -it tomcat01 /bin/bash

# ls 发现 webapps 目录下没有文件，官方打镜像的时候把对应的文件放到 webapps.dist 下了。拷贝一下，问题解决
cp -r webapps.dist/* webapps
```

## 部署 EC + kibana

* ES 暴露的接口多
* ES 十分耗内存
* ES 数据需要备份

```bash
# docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:tag
# --net somenetwork?
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2

# 查看服务状态
docker stats
# CONTAINER ID   NAME                          CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
# 4d87cd0d4ff6   elasticsearch                 0.78%     1.26GiB / 15.64GiB    8.06%     936B / 0B         0B / 729kB        45
# d3119daeed2c   bizx-docker-dev_kafka_1       1.36%     437.9MiB / 1.562GiB   27.37%    1.43MB / 2.08MB   1.25MB / 32.8kB   80
# 24fe744a20ea   bizx-docker-dev_zookeeper_1   0.31%     101.8MiB / 768MiB     13.26%    2.92MB / 2MB      50.5MB / 0B       50
# f9b12d209a07   hana2_hana2_1                 4.27%     3.714GiB / 15.64GiB   23.75%    823MB / 3.53GB    1.69GB / 1.08GB   291

# 发送请求测试, 成功
curl localhost:9200
# {
#   "name" : "4d87cd0d4ff6",
#   "cluster_name" : "docker-cluster",
#   "cluster_uuid" : "ojWX85pITJyL7WkVnoKZcA",
#   "version" : {
#     "number" : "7.6.2",
#     "build_flavor" : "default",
#     "build_type" : "docker",
#     "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
#     "build_date" : "2020-03-26T06:34:37.794943Z",
#     "build_snapshot" : false,
#     "lucene_version" : "8.4.0",
#     "minimum_wire_compatibility_version" : "6.8.0",
#     "minimum_index_compatibility_version" : "6.0.0-beta1"
#   },
#   "tagline" : "You Know, for Search"
# }

## 停止服务，修改内存配置 -e ES_JAVA_OPTS="-Xms64m -Xmx512m" 修改内存配置
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2
```

## 可视化

* Portainer - 图形化管理工具
* Rancher - CI/CD

```bash
# 访问 localhost:9000 可见页面
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

## Docker 镜像加载原理

UnionFS 联合文件系统，分层，轻量级且高性能。

## 制作镜像

```bash
# -a: 作者
# -m: commit 信息
# 630bab3ed5c2：container sha
# tomcat02:1.0：镜像名称 + 版本号
docker commit -a='jzheng' -m='add webapps' 630bab3ed5c2 tomcat02:1.0
docker images
# REPOSITORY                                          TAG            IMAGE ID       CREATED         SIZE
# tomcat02                                            1.0            67be5e0517c6   7 seconds ago   672MB
# mysql                                               latest         0627ec6901db   42 hours ago    556MB
```

## 容器数据卷

删除容器，数据一起丢失

Docker 容器中产生的数据，同步到本地

目录挂载，将容器内的目录挂载到宿主机上

容器的持久化和同步操作，容器间也是可以数据共享的

```bash
# sample: docker run -it -v /Users/id/tmp/mount:/home centos /bin/bash
docker run -it -v host_addr:container_addr

docker inspect container_id
# 通过 inspect 可以看到具体的挂载信息
# ...
# "Mounts": [
#             {
#                 "Type": "bind",
#                 "Source": "/Users/id/tmp/mount",
#                 "Destination": "/home",
#                 "Mode": "",
#                 "RW": true,
#                 "Propagation": "rprivate"
#             }
#         ]
# ...

# home 目录下新建文件，内容会同步到外边

停止容器，修改宿主机下的同步文件夹内容，容器启动后改动会同步到容器中
```

## 安装 MySQL

```bash
docker pull mysql:5.7

# 启动并挂载
# 启动 mysql 需要配置密码, 加入 -e MYSQL_ROOT_PASSWORD=my-secret-pw 参数，查看官方镜像文档了解详细信息
docker run -d -p 3306:3306 -v /Users/id/tmp/mysql/conf:/etc/mysql/conf.d -v /Users/id/tmp/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name=mysql01 mysql:5.7

# 测试, 遇到一些小波折。。。
启动 DBeaver，链接数据库，报错：`Unable to load authentication plugin 'caching_sha2_password'.`

搜索之后，发现是 mysql 驱动有跟新，需要修稿客户端的 pom, 升级到 8.x 就行。DBeaver 直接就在创建选项里给了方案，选 8.x 那个就行 [GitIssue](https://github.com/dbeaver/dbeaver/issues/4691)

使用高版本的 Mysql connection 还是有问题，不过 msg 变了：`Public Key Retrieval is not allowed`

再查一下，说是还要该配置, connection setting -> Driver properties -> 'allowPlblicKeyRetrieval' 改为 true

还有问题。。。继续抛错：`Access denied for user 'root'@'localhost' (using password: YES)`

先用 docker exec -it mysql01 /bin/bash 进去容器，输入 `mysql -u root -p` 尝试登陆，成功。推测是链接客户端的问题。。。

再用 `ps -ef | grep mysql` 查看了一下，突然想起来，本地我也有安装 mysql 可能有冲突。果断将之前安装的 docker mysql 删除，重新指定一个新的端口，用 DBeaver 链接，成功！

通过客户端创建一个新的数据库 new_test, 再本地映射的 data 目录下 ls 一下，可以看到新数据库文件生产出来了

# > ~/tmp/mydb/data ls
# auto.cnf           ca.pem             client-key.pem     ib_logfile0        ibdata1            mysql              performance_schema public_key.pem     server-key.pem
# ca-key.pem         client-cert.pem    ib_buffer_pool     ib_logfile1        ibtmp1             new_test           private_key.pem    server-cert.pem    sys

# 删除容器，本地文件依然存在
```

## 具名挂载 Vs 匿名挂载

```bash
# 匿名挂载: -v 不指定宿主机挂载目录
docker run -d -P --name nginx01 -v /ect/nginx nginx

# 查看卷情况
docker volume ls
# DRIVER    VOLUME NAME
# local     125a67b4291a80270d9839b62abbc7143f05fa133bf2e48c1057a4c9476ad591 <- 没有指定路径，宿主机上没有具体的挂载点，即匿名挂载
# local     portainer_data

# 具名挂载
# juming-nginx 即卷名
docker run -d -P --name nginx02 -v juming-nginx:/ect/nginx nginx
# DRIVER    VOLUME NAME
# local     125a67b4291a80270d9839b62abbc7143f05fa133bf2e48c1057a4c9476ad591
# local     juming-nginx
# local     portainer_data

# 使用 inspect 查看具体信息
docker volume inspect juming-nginx
[
    {
        "CreatedAt": "2021-04-22T12:22:21Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming-nginx/_data",
        "Name": "juming-nginx",
        "Options": null,
        "Scope": "local"
    }
]

-v 容器内路径 # 匿名挂载
-v 卷名:容器内路径 # 具名挂载
-v /宿主机路径:容器内路径 # 指定路径挂载

# -v /宿主机路径:容器内路径:ro/rw
ro: read only, 只能通过宿主机改变，容器内部不能改变
rw: read and write 
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

