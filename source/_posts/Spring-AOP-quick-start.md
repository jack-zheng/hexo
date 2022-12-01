---
title: Spring AOP quick start
date: 2022-06-27 14:45:45
categories:
- Spring
tags:
- AOP
---

快速了解 AOP 必要知识并给出 demo。这篇文章是在我系统学了 AOP 的基本使用之后再写的，之前遇到的很多痛点都体现不出来了。花了挺多时间读官方文档的时候，然后很多问题都迎刃而解了，就是比较花时间。之前失败的主要原因是 pointcut 的表达式写错了，tomcat 起了，但是服务都失败了。。。。

## 简单概括什么是 AOP

不破坏代码结构的情况下，为代码添加功能，典型案例如打印方法的执行时间。

## AOP 涉及的专有名词

* Joinpoint - 要匹配的方法
* Pointcut - 匹配方法的规则
* Advice - 检测到匹配方法的时候要执行的 额外 逻辑
* Aspect - 带 @Aspect 的那个类

详细的指代会在下面的例子中标记出来，光是名词实在难记

## 怎么写

如果 Spring 整体环境已经搭建完成，如果你想要新建一个 Aspect 只需要做两件事

1. 写 Aspect 文件，最简单的方式是写 Advice 方法，并在注解中添加 pointcut 表达式
1. 注册 Aspect, 可以通过 Xml 或者 Annotation 注册

在实际例子中标注 AOP 相关的概念，直接看名词很难记住，理解

```java
// @Aspect 指代的就是一个切面，即 aspect 这个概念
@Aspect
@Component
public class MyAspect {
    
    // pointcut expression, 表示匹配规则，哪些方法需要被处理
    @Pointcut("target(official.AopTestService)")
    public void pointcutName() {}
    
    // @Around 即为 Advice, 表示切入方式，其他还有 @Before, @After 等多种方式
    @Around("pointcutName()")
    // 可以合并简写为 @Around("target(official.AopTestService)")
    // join point 表示被拦截的方法，可以从它里面拿到方法名，参数等信息
    public Object aroundSayHello(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("Before join point...");
        Object obj = joinPoint.proceed();
        System.out.println("After join point...");
        return obj;
    }
}

// 目标 service，当 service 方法执行时会触发切面
@Component
public class AopTestService {
    public void sayHello(String name) {
        System.out.println("Hello, " + name);
    }

    public void greet() {
        System.out.println("Hello, there");
    }
}

// 配置类
@Configuration
@EnableAspectJAutoProxy
@ComponentScan(basePackages = "official")
public class AppConfig {}

// 执行方法
public class Client {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        ctx.getBean("aopTestService", AopTestService.class).greet();
        ctx.getBean("aopTestService", AopTestService.class).sayHello("Jack");
    }
}
```

## 原理剖析

所有的实现都是基于代理模式完成的。基础版本，不借助任何 lib, 纯 Java 实现。假设我们有 ISubject 接口，和对应的实现类 SubjectImpl，代理类 SubjectProxy。我们可以通过调用 SubjectProxy 来触发 SubjectImpl 并添加我们自己的定制逻辑。

```java
// 抽象的业务逻辑接口
public interface ISubject {
    void request();
}

// 具体的实现类
public class SubjectImpl implements ISubject {
    @Override
    public void request() {
        System.out.println("Invoke request in SubjectImpl...");
    }
}

// 代理类，可以添加定制的方法
public class SubjectProxy implements ISubject {
    private ISubject subject;

   // constructor + set method

    @Override
    public void request() {
        System.out.println("Call request in proxy...");
        subject.request();
    }
}

// 调用场景
public class Client {
    public static void main(String[] args) {
        ISubject subject = new SubjectProxy(new SubjectImpl());
        subject.request();
    }
}
```

进阶使用，如果还有其他接口比如 IRequestable 也有 request 方法，那么为了实现代理，我们还需要为他新建一个代理类，如果类似的类很多，就会有很多代码冗余。JDK提供了动态代理的机制解决这种问题，概括来就是给出时间对象和目标接口，就能实现代理逻辑的统一操作

```java
public class RequestCtrlInvocationHandler implements InvocationHandler {
    private Object target;

    public RequestCtrlInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Invoke method in InvocationHandler...");
        if (method.getName().equals("request")) {
            return method.invoke(target, args);
        }
        return null;
    }
}

public class InvocationHandlerClient {
    public static void main(String[] args) {
        SubjectImpl impl = new SubjectImpl();
        ISubject subject = (ISubject) Proxy.newProxyInstance(
                impl.getClass().getClassLoader(),
                new Class[]{ISubject.class},
                new RequestCtrlInvocationHandler(impl));
        subject.request();
    }
}
```

再进一步，上面的方法只能处理实现了接口的情况，如果类并没有实现接口，还想要增强的话，我们需要借助 Cglib 这个第三方库，在类上派生出一个子类，通过复写方法达到扩展的目的。

```java
// 目标类，虽然后 request 方法，但是没有任何的接口
public class Requestable {
    public void request() {
        System.out.println("Call in Requestable, without interface...");
    }
}

// 通过 Cglib 实现的方法扩展
public class RequestCtrlCallBack implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        if (method.getName().equals("request")) {
            System.out.println("Invoke method in MethodInterceptor...");
            return methodProxy.invokeSuper(o, objects);
        }
        return null;
    }
}

// 调用方式
public class CallBackClient {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Requestable.class);
        enhancer.setCallback(new RequestCtrlCallBack());

        Requestable proxy = (Requestable) enhancer.create();
        proxy.request();
    }
}
```

Spring 的实现中，会判断是否有接口，如果有则使用动态代理，如果没有则使用字节码增强。

## 奇怪的 behavior

最近发现产品上的 AOP 代码在两个 aspect 嵌套的情况下只会执行第一个 aspect，第二个直接跳过了，不知道是公司特有的还是 AOP 本来就有这种设定，特意检测一下

> 场景重现:
> service A 有两个 method01，method02 并且 method01 会调用 method02. 创建 Aspect 同时覆盖这两个方法，当 method01 执行时，method02 并不会被检测到。
> 
> 搜了一下 stackoverflow, 有人指出这个现象底层原理已经在 5.8.1 中写了。。。。proxy 之后，对自己的调用将会失效
> Spring 4.3 之后可以通过 class 中注入本身来绕过这个问题 
> [stackoverflow exp + solution](https://stackoverflow.com/questions/13564627/spring-aop-not-working-for-method-call-inside-another-method)
