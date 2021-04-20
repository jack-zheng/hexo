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

