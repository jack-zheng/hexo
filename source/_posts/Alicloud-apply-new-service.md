---
title: 阿里云购买服务器
date: 2021-04-25 21:36:31
categories:
- 配置
tags:
- 阿里云
- docker
---

阿里云购买云服务，本地通过终端连接使用，终端工具，弹幕各种推荐 Mobaxtream, 打算试用一下

## 创建安全组

阿里云上创建实例后，你只能通过他们提供的终端工具访问，如果你想要通过其他终端工具访问服务器，需要设置安全组。这个安全组你可以理解为云服务形势下的开启防火墙端口。

常用端口：

* 21: FTP
* 22: SSH
* 80: HTTP
* 443: HTTPS
* 3306: mysql
* 8080: tomcat
* 6379: redis

安全组 -> 创建安全组，将常用端口添加进去即可

## 创建实例

实例可以理解为远程虚拟机，进入阿里云主界面，顶部选择 控制台 -> 点右上角三条杠 -> 下拉单选择 实例 -> 右上角选择 创建实例

自定义购买下面选择：

* 付费模式： 按量付费
* 地域：就近 - 上海
* 规格：测试 docker 选 1 核 2 G

其他只需要注意一下安全组设置即可，创建完毕后，点击你创建的实例，进入面板后就可以看到你创建的实例的公网 IP，然后点击右上角的全部操作下拉单 -> 修改密码

## 远程链接

下载 Mobaxtream 并安装，没什么好说的，新建 session 然后填写账号密码尝试链接，没什么问题的话都 OK 的

## 安装 docker

```bash
yum update

yum install docker -y

docker -v # 检查版本

service docker start # 启动 docker 服务，不然 docker ps 都没法运行
chkconfig docker on # 设置开机运行
```

## 镜像加速

控制台界面搜索 容器镜像服务 -> 镜像工具 -> 镜像加速器

他会根据账号给出加速地址和配置方法，很直观， 对于 CentOS 可以通过修改 daemon 配置文件/etc/docker/daemon.json来使用加速器

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

## 启动 nginx 镜像测试

```bash
docker run -d -p 8080:80 --name my-nginx nginx 

# 本地测试
curl localhost:8080
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
# <h1>Welcome to nginx!</h1>
# <p>If you see this page, the nginx web server is successfully installed and
# working. Further configuration is required.</p>
# ...
# </body>
# </html>

# 外网直接通过浏览器输入 ip:8080 可以看到 nginx 页面显示成功！
```

PS: 第一次测试内部访问成功，外部失败，查看安全组，发现添加输入框末端需要点击保存...