---
title: Python Superset install
date: 2022-07-22 11:19:30
categories:
- python
tags:
- superset
---

## 准备工作

docker 方式安装 Superset 的步骤中有一步是 load_example, 这步的过程中会去 github 上下载案例。不过由于 GFW 的问题，下载会失败。解决方案为，手动下载这个 repo 并在本地起一个服务器挂载这些资源，然后修改 docker 中下载 sample 脚本的配置即可。以下是下载资源，起服务部分配置

```bash
wget https://github.com/apache-superset/examples-data/archive/refs/heads/master.zip
unzip master.zip
# 或者
git clone https://github.com/apache-superset/examples-data.git

# 如果是 zip 包方式，输入 cd examples-data-master
cd examples-data 
# 启动服务器，这是你通过访问 localhost:9999 就能看到这个网站服务了
python -m http.server 9999 
# 查看本机 ip 用于后续修改下载地址
# 显示结果中有一行格式类似 inet xx.xx.xx.xx netmask 0xffffe000 broadcast xx.xx.xx.xx
# 第一个 ip 段就是我们想要的
ifconfig | grep inet
```


## 安装 Superset

访问 [dockerhub](https://hub.docker.com/r/apache/superset) 查看安装文档，其实就是跟着指导 CV 一遍指令

```bash
# 拉镜像并启动服务，通过 -p 参数修改外部暴露的端口 -p port_you_want:8088
docker run -d -p 8080:8088 --name superset apache/superset
# 创建账户
docker exec -it superset superset fab create-admin \
              --username admin \
              --firstname Superset \
              --lastname Admin \
              --email admin@superset.com \
              --password admin
# 升级 DB
docker exec -it superset superset db upgrade
# 修改下载地址
docker exec -it superset /bin/bash
sed -i 's/BASE_URL = .*"/BASE_URL = "http:\/\/xx.xx.xx.xx:9999\/"/g' superset/examples/helpers.py
sed -i 's/https:\/\/github.com\/apache-superset\/examples-data\/raw\/master\//http:\/\/xx.xx.xx.xx:9999\//g' superset/examples/configs/datasets/examples/*.yaml
sed -i 's/https:\/\/github.com\/apache-superset\/examples-data\/raw\/lowercase_columns_examples\//http:\/\/xx.xx.xx.xx:9999\//g' superset/examples/configs/datasets/examples/*.yaml
sed -i 's/https:\/\/raw.githubusercontent.com\/apache-superset\/examples-data\/master\//http:\/\/xx.xx.xx.xx:9999\//g' superset/examples/configs/datasets/examples/*.yaml
exit
# 加载案例
docker exec -it superset superset load_examples
# 官方说是 setup roles, 目测是账号权限之类的东西
docker exec -it superset superset init
```

到这里安装结束，通过访问 http://localhost:8080/login/ 查看页面，使用 admin/admin 登陆

## 画图案例

