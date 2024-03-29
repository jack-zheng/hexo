---
title: What is IPv6
date: 2021-07-28 11:30:31
categories:
- What
tags:
- POSIX
---

## What is IPv6

Internet Protocol version 6: 网际协议第六版，用于解决 IPv4 地址枯竭的问题

**格式:** IPv6二进位制下为128位长度，以16位为一组，每组以冒号“:”隔开，可以分为8组，每组以4位十六进制方式表示。例如：2001:0db8:86a3:08d3:1319:8a2e:0370:7344 是一个合法的IPv6地址。

同时IPv6在某些条件下可以省略：

每项数字前导的0可以省略，省略后前导数字仍是0则继续，例如下组IPv6是等价的。

* 2001:0db8:02de:0000:0000:0000:0000:0e13
* 2001:db8:2de:0000:0000:0000:0000:e13
* 2001:db8:2de:000:000:000:000:e13
* 2001:db8:2de:00:00:00:00:e13
* 2001:db8:2de:0:0:0:0:e13

可以用双冒号“::”表示一组0或多组连续的0，但只能出现一次：如果四组数字都是零，可以被省略。遵照以上省略规则，下面这两组IPv6都是相等的。

* 2001:db8:2de:0:0:0:0:e13
* 2001:db8:2de::e13
* 2001:0db8:0000:0000:0000:0000:1428:57ab
* 2001:0db8:0000:0000:0000::1428:57ab
* 2001:0db8:0:0:0:0:1428:57ab
* 2001:0db8:0::0:1428:57ab
* 2001:0db8::1428:57ab

2001::25de::cade 是非法的，因为双冒号出现了两次。它有可能是下种情形之一，造成无法推断。

* 2001:0000:0000:0000:0000:25de:0000:cade
* 2001:0000:0000:0000:25de:0000:0000:cade
* 2001:0000:0000:25de:0000:0000:0000:cade
* 2001:0000:25de:0000:0000:0000:0000:cade

如果这个地址实际上是IPv4的地址，后32位可以用10进制数表示；因此::ffff:192.168.89.9 相等于::ffff:c0a8:5909。另外，::ffff:1.2.3.4 格式叫做IPv4映射地址。

## How to test

本地怎么模拟一个 IPv6 的地址做测试？