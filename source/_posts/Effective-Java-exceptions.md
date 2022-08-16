---
title: Effective Java 异常
date: 2022-08-05 10:44:32
categories:
- Effective Java
tags:
- exception
---

异常章节摘抄

* 只针对异常的情况才使用异常
* 对可恢复的情况使用 checked exception, 不可恢复的使用 runtime exception
* 避免滥用 checked exception
* 优先使用 JDK 提供的标准异常
* 抛出与上下文匹配的异常
* 在方法注释上，为每个抛出的异常编写文档
* 异常信息中包含失败信息
* 让失败保持源字形
* 不要忽略异常

## 只针对异常的情况才使用异常

展示一个产品代码中遇到的违反这条建议的的奇葩例子。底层 team 给了一个很奇葩的接口，一个 findPersonIdByPersonUUID 的方法，当找不到结果的时候，竟然会抛出 exception 而不是返回 null, 导致我这个调用方处理起来像是吃了屎一样难受。。。而且这样的逻辑把 '找不到' 和 '代码异常' 两中情况混在一起了，抛出了异常之后都不知道具体的 root cause，简直了。

```java
// 底层实现
public class BadPracticeServiceA {
    public long getPersonIdByPersonUUID(String personUuid) {
        Long person = getPersonId();
        if (person == null) {
            throw new ServiceApplicationException("Can not find PersonID by PersonUUID");
        }
        return person;
    }
}

// 作为上层的调用者，我在调用上述方法的时候不得不显示的处理他抛出的异常
try {   
    long target = badPracticeServiceA.getPersonIdByPersonUUID("uuid");
} catch (IllegalStateException e) {
    log.info("Exception when get person id by person uuid {}, detail log msg: {}", "uuid", e.getMessage());
}

return null;
```

看 Effective Java 中 Stack 实现的时候又有了新的感悟。这里的 get 方法和 stack 里的 pop() 方法很像，而 pop 在拿不到值的时候会抛异常，和上面找不到的情况也很类似。不过相对以 Stack 的例子，上面如果要在找不到的时候抛异常，也不应该是 Exception 而是更友好的 RuntimeException。或者其他自定义的 NotFoundExcepiton 才对。

## 对可恢复的情况使用 checked exception, 不可恢复的使用 runtime exception

* 如果期望调用者能够适当的恢复，对于这种情况应该使用 checked exception
* runtime exception 和 error 行为上是等同的，没有捕获，当前线程停止
* runtime exception 表明程序错误

## 避免滥用 checked exception

滥用 checked exception 会使你的 API 调用极其麻烦。一旦抛出 checked exception，调用者就必须 catch 他或者在方法标签中 throw，传播出去，无论哪种处理方式，都是对调用者不可忽视的负担。

绕过 checked exception 的方法：

* 使用 runtime exception 代替
* 重构逻辑

## 优先使用 JDK 提供的标准异常

常见的标准异常

* IllegalArgmentException
* IllegalStateException
* NullPointException
* IndexOutOfBoundsException
* ConcurrentModificationException
* UnsupportedOperaitonException

## 抛出与上下文匹配的异常

底层异常，传递到上层是可能和当前上下文不匹配，污染 API。这是需要在上层转译一下，重新抛出异常。如果异常栈对错误排查很有帮助，可以将 exception chain 一并传递给新的异常(Throwable 的 initCause 方法)

异常转译比直接传递底层异常好，但更好的处理方式应该是调用底层方法之前确保他能执行成功，避免抛出一场。有时调用底层之前，先做一下参数监测可能是更好的方式。

如果无法避免底层异常，次选方案是让上层悄悄绕开这些异常，从而使上层调用者和底层问题隔离开来，通常这是还会打个 log 记录，方便回溯问题。
