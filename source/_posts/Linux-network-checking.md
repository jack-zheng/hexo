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