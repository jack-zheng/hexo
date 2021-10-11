---
title: Tomcat manager app
date: 2021-09-24 10:36:25
categories:
- Tomcat
tags:
- Manager
- Tomcat
---

官方文档：[Manager - howto](https://tomcat.apache.org/tomcat-8.0-doc/manager-howto.html)

长见识了，读这本书之前，完全不知道，原来 tomcat 还有集成这种功能，厉害了。看了 How To 说明。这个 Manager 给了维护人员一个快捷的通道，通过他你可以管理 Tomcat 下的各个 app 的状态，可以随时开启，停止，重新 deploy 而且你不需要问了这个目的重启整个 Tomcat。此外还可以监测各种环境数据，比如 JVM 信息，系统参数，Server 状态等。

## 开启

Manager 功能默认是不开启的，当你本地启动项目时，可以试着访问一下 `/manager` 这个路径，会给 403 err

![403](403.png)

提示很清楚，你要在 /conf 下配置 tomcat-user.xml 才能开启这个功能。给了 manager-gui 之后就不需要 script 和 status 权限了，这是处于安全考虑。为了测试方便也可以全部加上

```xml
<user username="tomcat" password="s3cret" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
```

重启 Tomcat，访问 http://localhost:8080/manager 即可看到管理页面

![status](status.png)

能管理的项目 UI 展示都很直接，不细说了

## Manager Commands

Manager 还支持通过 request 进行控制，你需要按照前面说的给 manager-status 这个 permission 之后才能开启。开启后，访问 http://localhost:8080/manager/text/list 就能看到效果，其实就是 UI 功能的 request 版本，细节参考开头的文档。

此外 Tomcat 还支持 Ant 脚本跑这些命令，用不到，暂时不看了。