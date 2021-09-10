---
title: Ex08 类加载器
date: 2021-08-02 11:14:13
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 8** explains about loaders. A loader is an important Catalina module responsible for loading servlet and other classes that a web application uses. This chapter also shows how application reloading is achieved.

之前章节我们已经给出了一个简单的 loader 实现用于加载 servlet。这章我们将介绍 tomcat 的 standard web application loader. servlet container 必须实现自己的 loader，而不能使用系统自带的那个。因为它不信任运行的 servlets。如果它像我们之前的例子那样使用默认的类加载器，那么 servlet 将可以访问任何 JVM classpath 下的 class 和 lib，这和 security 的规则相违背。

一个 servlet 只允许加载 WEB-INF/classes 和 WEB-INF/lib 文件夹下的内容, 那个 web application(context) 需要有它自己的 loader。Catalina 中，org.apache.catalina.Loader 表示 loader 类。

另一个 tomcat 需要自己的 loader 的原因是它需要支持自动加载的功能。当 WEB-INF/classes 和 lib 下的内容发生改变时，这个 loader 需要自动检测并重新加载。Tomcat 新起一个线程完成这个功能, org.apache.catalina.loader.Reloader 即代表了 reload 这个行为。

本章第一部分介绍 Java 中的类加载机制。之后介绍 Loader 接口，最后演示 tomcat 的 loader 使用案例

本章中两个术语 repository 表示 class loader 会搜索的地方，resources 表示 DirContext，它指向 context 的 document 目录。

## Java Class Loader

查看 深入理解 Java 虚拟机 第 7，9 章节，说的很清楚了。

Java 允许你定制自己的 class loader，只需继承 java.lang.ClassLoader 即可。以下是 tomcat 需要定制 loader 的原因

* To specify certain rules in loading classes.
* To cache the previously loaded classes.
* To pre-load classes so they are ready to use.

## The Loader Interface

{% plantuml %}
title The Loader interface and its implementation
interface Loader
interface Reloader
class URLClassLoader
class WebappClassLoader

class WebappLoader

Loader <|.. WebappLoader
Reloader <|.. WebappClassLoader
URLClassLoader <|-- WebappClassLoader

WebappLoader "Uses".> WebappClassLoader
{% endplantuml %}

## The Reloader Interface

为了提供自动重载的功能，loader 必须实现 org.apache.catalina.loader.Reloader 接口

```java
public interface Reloader {

    public void addRepository(String repository);

    public String[] findRepositories();

    public boolean modified();
}
```

其中最重要的是 modified() 方法，当应用中的 servlet 或者 supporting classes 有改动时，他会返回 true。

## The WebappLoader Class

WebappLoader 是 Loader 接口的一个实现，他代表一个 web application 负责为这个应用加载 class。WebappLoader 会创建一个 WebappClassLoader 作为他的类加载器。和其他 Catalina 组件一样，WebappLoader 也实现了 Lifecycle 和 Runnable 接口。前者借由相关组件控制开启停止，后者可以通过多线程实现类的重载。class 重载由 Context 执行，而不是 WebappLoader， 细节将在 Chapter 12 的 StandardContext 介绍。

WebappLoader 中的主要方任务：

* Creating a class loader
* Setting repositories
* Setting the class path
* Setting permissions
* Starting a new thread for auto-reload

### Create A Class Loader

WebappLoader 将类加载委托给了一个内部的类加载器，外部并不能直接直接创建这个加载器。但是可以通过 getClassLoader() 拿到它。如果你想要指定自己的应用加载器，可以通过 setLoaderClass() 方法传入加载器全路径，需要注意的是，加载器最终是通过 createClassLoader() 方法创建的，所以自定义的类加载器必须继承自 WebappClassLoader 不然会抛异常。

```java
private WebappClassLoader createClassLoader() throws Exception {

    Class clazz = Class.forName(loaderClass);
    WebappClassLoader classLoader = null;

    if (parentClassLoader == null) {
        // Will cause a ClassCast is the class does not extend WCL, but
        // this is on purpose (the exception will be caught and rethrown)
        classLoader = (WebappClassLoader) clazz.newInstance();
    } else {
        Class[] argTypes = { ClassLoader.class };
        Object[] args = { parentClassLoader };
        Constructor constr = clazz.getConstructor(argTypes);
        classLoader = (WebappClassLoader) constr.newInstance(args);
    }

    return classLoader;
}
```

### Setting Repositories

WebappLoader 的 start() 方法中调用 setRepsitories 向 class loader 中添加 repositories。WEB-INF/classes 传给了 addRepository()，WEB-INF/lib 传给了 setJarPath()。

### Setting the Class Path

这个 task 通过在 start() 方法中调用 setClassPath() 实现。

### Setting a New Thread for Auto-Reload

当 WEB-INF/classes 或者 WEB-INF/lib 下的文件被修改了，修改的类需要在 Tomcat 不重启的情况下自动刷新。为了达到这个效果，WebappLoader 新启了一个线程，周期性的检查文件的时间戳，默认检查周期为 15s, 用户可以通过 get/setCheckinterval() 设置这个值。

```java
public void run() {
    if (debug >= 1)
        log("BACKGROUND THREAD Starting");

    // Loop until the termination semaphore is set
    while (!threadDone) {

        // Wait for our check interval
        threadSleep();

        if (!started)
            break;

        try {
            // Perform our modification check
            if (!classLoader.modified())
                continue;
        } catch (Exception e) {
            log(sm.getString("webappLoader.failModifiedCheck"), e);
            continue;
        }

        // Handle a need for reloading
        notifyContext();
        break;

    }

    if (debug >= 1)
        log("BACKGROUND THREAD Stopping");
}
```

PS: Tomcat5 中将这部分 task 移到 StandardContext 中的 backgroundProcess() 中去了

run() 中的核心方法是通过 while 循环定时检测 modified 的值，流程如下

* Sleep 一定时间
* 调用 modified() 查看 flag, 如果 false，continue
* 返回 true，调用 notifyContext() 方法，通过 Context 做 reload

```java
private void notifyContext() {
    WebappContextNotifier notifier = new WebappContextNotifier();
    (new Thread(notifier)).start();
}
```

notifyContext 并没有直接调用 Context 的 relaod 方法，而是通过启动内部类 WebappContextNotifier 线程做的。

```java
protected class WebappContextNotifier implements Runnable {
    public void run() {
        ((Context) container).reload();
    }
}
```

## The WebappClassLoader Class

WebappClassLoader 继承自 URLClassLoader，在实现时兼顾了性能和安全。性能方面，它会将之前还在的类 cache 一下，当需要加载类时先从 cache 中寻在，找不到在通过加载器加载。之前加载失败的也有对应的 cache.

安全方面，WebappClassLoader 有一个黑名单，阻止加载一些类

```java
private static final String[] triggers = {
    "javax.servlet.Servlet"
};

private static final String[] packageTriggers = {
    "javax",                                     // Java extensions
    "org.xml.sax",                               // SAX 1 & 2
    "org.w3c.dom",                               // DOM 1 & 2
    "org.apache.xerces",                         // Xerces 1 & 2
    "org.apache.xalan"                           // Xalan
};
```

### Caching

为了性能考虑，加载的类会被 cache 住，后面 class 加载时，会先从 cache 中拿。Caching 是通过 WebappClassLoader 中的 local cache 实现的，同时之前加载过的类通过 ClassLoader 中的 Vector 管理避免类被垃圾回收掉。

能被 WebappClassLoader 加载的类统称为 resource，通过 org.apache.catalina.loader.ResourceEntry 表示。ResourceEntry 会持有类的 byte 数据，最后修改日期，Manifest 等。

```java
public class ResourceEntry {
    public long lastModified = -1;
    public byte[] binaryContent = null;
    public Class loadedClass = null;
    public URL source = null;
    public URL codeBase = null;
    public Manifest manifest = null;
    public Certificate[] certificates = null;
}
```

cached resources 存在名为 resourceEntries 的 HashMap 中，以 resource name 作为 key. 没有找到的 resource 存在名为 notFoundResources 的 HashMap 中。

### Loading Classes

下面是 WebappClassLoader 加载 class 的规则

* All previously loaded classes are cached, so first check the local cache.
* If not found in the local cache, check in the cache, i.e. by calling the findLoadedClass of the java.lang.ClassLoader class. 
* If not found in both caches, use the system's class loader to prevent the web application from overriding J2EE class.
* If SecurityManager is used, check if the class is allowed to be loaded. If the class is not allowed, throw a ClassNotFoundException.
* If the delegate flag is on or if the class to be loaded belongs to the package name in the package trigger, use the parent class loader to load the class. If the parent class loader is null, use the system class loader.
* Load the class from the current repositories.
* If the class is not found in the current repositories, and if the delegate flag is not on, use the parent class loader. If the parent class loader is null, use the system class loader.
* If the class is still not found, throw a ClassNotFoundException.

## The Application

本章使用现成的 StandardContext 管理上下文，12 章会具体介绍，目前你只需要知道 StandardContext 会结合 listener 处理 event 即可。

本章自定义的 listener

```java
public class SimpleContextConfig implements LifecycleListener {

    public void lifecycleEvent(LifecycleEvent event) {
        if (Lifecycle.START_EVENT.equals(event.getType())) {
            Context context = (Context) event.getLifecycle();
            context.setConfigured(true);
        }
    }
}
```

我们创建 StandardContext 和 SimpleContextConfig 的实例并将 SimpleContextConfig 注册到 StandardContext 中。

同时我们复用前面章节的 SimplePipeline, SimpleWrapper 和 SimpleWrapperValve.

由于使用了 StandardContext 我们必须将测试 servlet 放到 WEB-INF/classes 下，这个例子中，我们创建一个新的目录 myApp 并创建对应的目录，通过 `System.setProperty("catalina.base", System.getProperty("user.dir"));` 指定 myApp 文件夹。

简单过一下 Bootstrap 的代码

```java
public final class Bootstrap {
    public static void main(String[] args) {

        //invoke: http://localhost:8080/Modern or  http://localhost:8080/Primitive

        System.setProperty("catalina.base", System.getProperty("user.dir"));
        Connector connector = new HttpConnector();
        Wrapper wrapper1 = new SimpleWrapper();
        // ... 设置 wrapper

        Context context = new StandardContext();
        // StandardContext's start method adds a default mapper
        context.setPath("/myApp");
        context.setDocBase("myApp");

        context.addChild(wrapper1);
        context.addChild(wrapper2);

        // context.addServletMapping()...
        // add ContextConfig. This listener is important because it configures
        // StandardContext (sets configured to true), otherwise StandardContext
        // won't start
        LifecycleListener listener = new SimpleContextConfig();
        ((Lifecycle) context).addLifecycleListener(listener);

        // here is our loader
        Loader loader = new WebappLoader();
        // associate the loader with the Context
        context.setLoader(loader);

        connector.setContainer(context);

        try {
            connector.initialize();
            ((Lifecycle) connector).start();
            ((Lifecycle) context).start();
            // now we want to know some details about WebappLoader
            WebappClassLoader classLoader = (WebappClassLoader) loader.getClassLoader();
            System.out.println("Resources' docBase: " + ((ProxyDirContext) classLoader.getResources()).getDocBase());
            String[] repositories = classLoader.findRepositories();
            for (int i = 0; i < repositories.length; i++) {
                System.out.println("  repository: " + repositories[i]);
            }

            // make the application wait until we press a key.
            System.in.read();
            ((Lifecycle) context).stop();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

整理一下思路：这张讲的内容调用点在 SimpleWrapper 的 loadServlet() 方法中。当访问页面时，最终到这个方法中，从 context 中拿到 class loader 并通过 classLoader.loadClass(cls) 加载类。这个 loader 和 classLoader 就是本章中的 WebappLoader 和 WebappClassLoader.

看完了感觉和 JVM 那边看到的有出入，很多自定义的 loader 都没讲到，热加载也没讲到，往后看看再说。