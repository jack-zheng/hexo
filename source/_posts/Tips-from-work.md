---
title: 工作中的那些琐碎小事
date: 2019-12-19 17:32:33
categories:
- 杂记
tags:
- bug
---
该页面用于记录实际工作中遇到的 bug，以示警戒

## 2021-05-26 Cache Issue

几年前经手的一个功能 Manage IP Restrictions, 当时还是 QA，考虑功能时还不够完善。Dev 在修改某个值时没有将对应的 cache 清掉，导致测试经常遇到不一致的情况。产品情况下修改不是很频繁，但是没这种情况，在测试的时候可把我愁坏了，干。。。

## Exception Handle 遗漏

有同事打补丁时对 checked exception 和 RunTimeException 处理有遗漏导致客户使用出问题，反馈后 debug 发现，简化后场景如下

```java
/**
* 场景描述：
*
* 在处理 filterData() 时，作者只考虑到 checked exception, 没有考虑 runtime exception.
* 实际使用时，客户在某些情况下会抛出 NPE 这种 runtime exception, 导致返回 null, 显示出现错误
*/
private List<Integer> populateDatas(datas) {
    try {
        for (data : datas) {
            try {
                filterData(data);
            } catch (FilterException fe) {
                System.out.println("Err when filter " + data);
            }
        }
    } catch (Exception e) {
        System.out.println("Populate data failed.");
        return null;
    }
}
```

### Java 中的异常分类

![Throwable关系图](relation.png)

### 常见的异常种类

RunTimeException:

* NPE
* AuthmeticException
* NumberFormatException
* IndexOutOfBoundsException

CheckedException:

* 反射相关：NoSuchMethod,FieldException
* NoSuchFileException

Error:

* OutOfMemmoryError
* ZipError

### 一点感悟

以后处理这样的问题还是要多留心 log, 从这个点出发的话估计这个问题发现只需要一个小时就够了。这次应为有很多干扰的 exception 跑出来，没有仔细查看导致绕了好大一个圈，要不是刚好本地有一个可以重现的样本就爆炸了╭(°A°`)╮ 谨记谨记

## Event 数据量撑爆了产品环境

开发完 event 相关的 feature 之后没有对测试环境进行跟踪，功能没有问题，但是产生了很多冗余数据，比如包含了很多将 field 从 null 跟新到 "" 空字串的 event。很多 data center 因为业务过重，单这个 event 每天产生 500w 数据，Kafka 就危了。。。引以为戒。

## JDBC 空字串存为 NULL

通过 JDBC 存储空字串时，他会自动将它存为 NULL

## 记录一个 jar 升级导致的问题

在原先的 code 中，我们有个 UT 需要 xstream 的 Mapper 类，就在 UT 里面直接实现了类接口。某天， xstream 突然被人升级到 1.4.9+ 了，原来的 UT 就挂了，在这个版本里新添加了一个方法 `isReferenceable` 原来的 case 是没有实现的

## Cache 处理的一些小技巧

在产品中发现处理 cache 的逻辑是，更新数据时删掉对应的 cache, 然后在取数据时再重新将 cache 存储起来。以前没注意，现在再看看发现挺有意思。

## @NonNull 标签

Java 方法的参数列表中加入 @NonNull 并不会在写 code 的时候为你提供 NPE check, 更多的是结合其他框架, 比如 Spring 使用, 本身只起到提示作用。

## Bug track 2021-04-07

今天遇到一个很诡异的问题，在 provisioning 中有一些 saveFeature 的 log 表明有时候 save 的时候会由于缺少 param 信息导致 GetSysConfig 的时候抛异常，而且频率很高。但是当我 manual 去重现这些功能时一切正常。通过查异常的上下文，发现这些有问题的 company 多是用于自动化测试的 instance。然后又仔细对比了手动正常工作时的 log 和出问题的 log 发现当异常产生时，save 的一系列动作都是在一个 transaction 中的，manual 操作是这一系列动作应该时分布在几个 transaction 中的。再结合以前的 auto 经验，这个东西大概率就是有一些 auto case 在调用了自己写的 script 操作 save feature 的时候出了问题，导致了一系列问题。这个问题如果不是对公司现有的技术手段都有所涉及，还真是不好找呢。。。

## 一个判断条件的优化

在 code review 的时候，有一个 if 需要判断 Boolean 对象为为空或者 false 才执行我就写了如下代码

```java
if (Objects.nonNull(obj) || !obj) {
    // do something
}
```

然后 Yi 就给了建议

```java
if (ojb != Boolean.True) {
    // do something
}
```

建议的修改更简单明了，哈哈

## 命令行查找目标文件夹(ls)

想要使用 ls 查找当前目录下的某个特定前缀的文件夹，但是 `ls prefix*` 会将对应的文件夹下面的自文件也列出来，不方便查看。可以加 `ls -d prefix*`。

通过 `man ls` 可以看到这个 flag 的作用: -d      Directories are listed as plain files (not searched recursively).

SF 上也给出了其他的解，可以用 `echo prefix*` 达到同样的效果

## 2021-05-29

今天遇到一个很 tricky 的 security issue. 客户在用我们的 provisioning 系统登陆的时候，是使用 SSO 的。前面还有一层 SAP 的授权层，叫 IDS 的。具体的 flow 可以表示为 IDS -> IAS -> Bizx. IAS 会用 email 做授权登陆。IDS 则是用 id. 但是 IDS 里面的用户可以随便更改自己的 email 就导致登陆到 Bizx 的用户可能串了。这是一个很严重的 security issue。如果真有心的话，这个 issue 说实话，可以让公司倒闭。

这个 issue 漏出去的原因还是因为这些部分分属于不同的公司维护，彼此之前的既成测试缺失，同时也不好做的缘故。

## 2021-08-25

遇到 bug 没关系，但是有些 bug 调查起来就是揪心。特别是那些，你在他们的调用链里。他很把锅甩给你，你还实锤不了他的这种情况。

Best Practice 说他们有个功能挂了，调用链如下 service(service1() -> myService2() -> service3()),  service3 挂了，整个 transaction roll back, myService2 执行完有一个 event 会发出去。现在的情况是 myService2 roll back 了，但是据说 event 还被消费了。整的我一脸问号。event 我们也是调用的其他模块的服务，不熟。调用方的代码，我们也不熟。我就直接黑人问号了？？？

## 2021-09-22

今天和 PM 讨论一个 last minutes 修改，只是改一个网站的地址(Prov Access Contral - HCM to HXM)。从我(开发)的角度来看，只是一个无所谓的改动，看了下代码，只是 5 分钟的改动。和 PM confirm 的时候，她反问了一句，这个新网址是不是已经上线了，老的是不是已经下线了。突然意识到，这是个很关键的问题，直接关系到 fix 的优先级，之前没考虑到。姜还是老的辣，哈哈，受教。