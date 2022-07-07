---
title: Lesson keep humble
date: 2022-07-04 13:24:24
categories:
- 杂记
tags:
- 感悟
- bug
---

最近这个从 session 中删除 provisioner bean 的工作做的确实不细致，有点懈怠了。在项目开始之前就没有做好工作安排，导致测试环境上出了 bug, 而且被其他 team challenge 了，实力打脸。以后应该吸取这个教训，案件还原如下。

某日 MCAP 的需求提上日程(MCAP 是将现有服务移到公有云上的这么一个项目，所以要减小内存开销)，突然要落地这个 remove 的项目。前期有调研过，可行性方面还是没问题的，原始业务如下：

在登陆时，将 provisioner bean 塞入 session， 在后续如果需要用到这个 bean, 通过 `context.get("provisionerBean")` 或者 `@Inject ProvisionerBean provisionerBean` 的方式获取。在查看原有业务时，我们发现，在存入 bean 的时候还会将对应的 id 存入 session。 那么作为解决方案，我们可以在现有拿 bean 的地方，通过拿 id 并查 DB 拿到 user 的方式绕过去。

在给 module team 实施修改方案的时候，他们提出一些 concern：多了访问 DB 的过程，有降低 performance 的风险。和 arch review 了这种风险，结合我们的实际使用场景，这个 provisioner 是后台管理人员，作为维护和系统 setup 的角色，使用频率很低，所以可以忽略这种风险。

目前为止都 OK 但是改代码的时候就有点粗暴，原来使用方式

```java
ProvisionerBean provisionerBean = SFContext.getContext().getInstance(SessionConstants.PROVISIONER_BEAN, false);
// 或者
@Qualifier(SessionConstants.PROVISIONER_BEAN)
@Inject
private ProvisionerBean provisionerBean;
```

这种使用方式是不会抛异常的，只要 get 就行了，但是用查 DB 的方式代替之后，会额外附带异常处理类似

```java
String provisionerId = SFContext.getContext().getInstance(SessionConstants.PROVISIONER_ID, false);
ProvisionerDataService provisionerDataService = SFContext.getContext().getInstance(ProvisionerDataService.NAME, true);
ProvisionerBean provisionerBean = null;
try {
    provisionerBean = provisionerDataService.findProvisionerById(provisionerId);
} catch (ServiceApplicationException e) {
    LOGGER.error("Exception occured while getting provisionerBean",e);
}
```

module team 反馈这种改法侵入性太强，会对他们的代码结构有很大的改变，涉及到源码和大批的 UT 改动。和他们讨论之后，我们打算将这个改动封装到我们内部的类中，并通过统一的接口暴露给外部使用。接口中处理异常，module 只负责调用而不产生其他副作用

```java
// provisionerServiceImpl.class
fetchProvisionerFromContext() {
    String provisionerId = context.get("provisionerId");
    ProvisionerBean provisionerBean = null;
    if (provisionerId != empty) {
        provisionerBean = service.findProvisionerFromDB(provisionerId);
    } else {
        provisionerBean = context.get(provisionerBean);
    }

    return provisionerBean
}
```

上面的只是简化版，实际代码中的逻辑还包括了从 session 中拿 provisioner bean 和 db 中查询结果做比较，如果不同则返回 session 中的结果并打印 log 记录。然后我们有另外一套 Splunk 的系统，我们可以在代码部署后通过检查 log 检测是否有预期外的行为，而且这种方式不会对现有的行为产生任何影响。

然而世事难料，和 Arch review 代码的时候，他提出了一种新的修改方式，由于我们要改的那些东西是在 SFContext 里面的，我们也许可以通过框架层面的修改来改变 bean 的生成方式，这种方式的侵入性是最小的。ProvisionerBean 是通过 factory bean 的方式向 context 的中注入 bean 的，我们可以 refactor 一下生成方式，改为先从 session 中拿到 id 然后调用 service 拿 DB 数据组成 bean。这样的话，所有调用点都不用改了，perfect！

又经过一轮 research，大致确定了通过修改 factory bean 的方案，我们返回 prototype 类型的 bean, 使用过后通过 GC 回收。而且后续其他 module 需要修改的代码量减少了很多, 对比如下

| Before            | After            |
| :---------------- | :--------------- |
| repo:12, call: 34 | repo: 6, call: 8 |

感觉这才是 Arch 这个角色的价值。contract, 就硬改，也不会看修改的地方是什么feature，不会想测测是不是有可能做回归。我现能达到的程度是，能够将这个 refactor 的任务条理理清楚，做改动的时候有这个 sense 去看看调用栈，看看会不会有什么没考虑到的情况并考虑如何测试改动。Arch 则是看了要改的地方，先看看系统本身是不是有地方提供了统一处理的能里，在达到目的的同时，将影响最小化，做法很优雅。


