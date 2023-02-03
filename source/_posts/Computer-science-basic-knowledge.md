---
title: 原码，反码和补码
date: 2019-12-07 22:52:24
categories:
- 科普
tags:
- 原码
- 反码
- 补码
---

通过搜索 计算机组成原理 话题话的 计算机减法 了解更多细节

## 概念

二进制数表示带符号值时，最高位位符号位，0表示正数，1表示负数，以 -1 为例：

反码：负数反码为其绝对值按位取反 1110
补码：负数补码等于反码 +1， -1 补码 1111

有符号的变量，内存中 1111 就表示 -1

反码，补码 这些概念都是为了方便计算机做减法而创建的，所有的数都以补码形式存在。正数的补码是本身，负数是以之前叙述的逻辑转化的数。减法结果和两个补码的加法就过时一样的。至此，我们就不必设计额外的减法器了，直接重用加法器就行了。