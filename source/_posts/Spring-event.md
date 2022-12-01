---
title: Spring event
date: 2022-10-19 14:16:16
categories:
- Spring
tags:
- event
---

Spring支持 event 模型，自带了几种 event，比如 ContextRefreshedEvent，ContextStartedEvent 等，会在 application context setup 的时候发出来，还可以自定义 event，参考官方文档中的 BlockedListEvent 例子。

自带的 event 是在 AbstractApplicationContext::finishRefresh::publishEvent 中处理的。如果是自定义的 event，我们还需要定义出对应的 Listener，可以通过实现 ApplicationListener 接口，在 context setup 的时候自动注入到 context 中。这个 listener 其实就是一段如何处理 event 的逻辑。

目标和受体都准备好了，接下来就是如何触发的问题了。我们可以让 service 实现 ApplicationEventPublisherAware 接口，然后调用 `publish()` 触发事件。publish 即 context，实现在 AbstractApplicationContext::publishEvent 中。他会根据 event type 等条件，删选出匹配的 listener 并执行，逻辑还是很直观的。

这整个 event 处理模型，使用了观察者的设计模式，实现的时候很简单，就是将所有的观察者放到一个 list 中，当事件发生时，通过 for 或者 iterator 遍历所有的观察者执行目标函数即可。
