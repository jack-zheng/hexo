---
title: Linux network checking
date: 2023-01-10 15:49:14
categories:
- linux
tags:
- network
---

最近 Mac 系统升级之后，休眠再唤醒，经常断网，而且重启 Wi-Fi 也不管用，对这方面也不熟悉，乘现在机子还正常，记录一下，下次按图索骥找找 root cause 看看

查看各 en 口的作用：`networksetup -listallhardwareports`, 运行结果

```bash
Hardware Port: USB 10/100/1000 LAN
Device: en7
Ethernet Address: 34:29:8f:xxx...

Hardware Port: Thunderbolt Bridge
Device: bridge0
Ethernet Address: 82:1a:67:xxx...

Hardware Port: Wi-Fi
Device: en0
Ethernet Address: 3c:22:fb:xxx...

Hardware Port: Thunderbolt 1
Device: en1
Ethernet Address: 82:1a:67:xxx...
```

可以看到 en0 是我们想要查看的端口代号，接着通过 `ifconfig en0` 查看具体情况

```bash
# en0: 以太网0
# flags=8863:网络状态标识
# BROADCAST:有广播地址，支持发广播包
# SMART:
# RUNNING:代表网卡的网线被接上
# MULTICAS:网卡可以发送多播包
# mtu: 最大传输单元
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	options=6463<RXCSUM,TXCSUM,TSO4,TSO6,CHANNEL_IO,PARTIAL_CSUM,ZEROINVERT_CSUM>
	ether 3c:22:fb:xxx...
	inet6 fe80::ca5:xxxx:xxxx:xxxx%en0 prefixlen 64 secured scopeid 0x6
	inet xxxx.xxxx.xxxx.252 netmask 0xffffe000 broadcast xxxx.xxxx.xxxx.255
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect
	status: active
```

输入 `sudo ifconfig en0 down` 密码为登陆密码来关闭网卡，可以看到 Wi-Fi 标志被关闭了， 同理输入 `sudo ifconfig en0 up` 来重新启动网卡

下次断网了可以用上面的命令先看看网卡状态，并试试重启。之前直接通过 UI 去重启并没有什么卵用。

如果还不能解决问题，可能是一些其他配置有问题，比如 网关，DNS，DHCP 等，这个到时候再看看吧，我倒是挺希望遇到这种问题的，可以涨涨见识。

## 二更

长见识的时候到了，晚上临下班，突然连不上网了。看了一下之前搜集的参数，网卡是激活的。我突然想到另一个办法，用热点代替公司 Wi-Fi, 可以上网，也就侧面论证了，我的机子是 OK 的，问题肯定在后面，明天搜一下更底层的问题怎么排错

下一步就是查看网络配置是否正确，涉及四个方面

* IP，通过 ifconfig 可以看到，由 DHCP(Dynamic Host Configuration Protocal) 服务器自动分配给你
* 子网掩码
* 网关，通过 `netstat -nr | grep 'default'` 查看
* DNS服务器, 通过 `cat /etc/resolv.conf` 查看

只有上面四者都正确网络才能正常工作。先 ping 网关 IP 或者内网其他主机的 IP 确认内网是否正常，后面就是 ping 外网，看外网连接是否正常

## 三更

今天 6:08 分网又挂了，做了如下实现

1. 通过热点 ping baidu, 访问成功
2. 通过 ifconfig 和 netstat 查看网络配置，重启前后没有区别

随即产生一些思考：

* 网关是什么，如果是网络关口，那么是最终端，还是第一个连接点。如果是第一个连接点，为什么我有线和无线分别链接的时候，显示的是同一个IP？这么看的话应该是最末端的节点喽？
* 下一步可以试试断网的时候去其他楼层连接一下看看
* 整个公司的无线网络名字都是一样的，那我怎么查看我连上的路由器是哪个呢，能查看IP吗
* 还有上网的时候能查看调过的节点吗，比如我ping的时候，走过了哪些路由器，服务器之类的

刚试了下 Mac 上如果关了 Wi-Fi 的那个标志，就算接了网线也不能联网额，是不是什么设置有问题，如果真这样，那我之前做的有线和无线的实验不是白瞎了。。。明天查查配置看

