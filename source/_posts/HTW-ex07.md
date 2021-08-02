---
title: HTW ex07
date: 2021-07-30 19:29:43
categories:
- Tomcat
tags:
- How Tomcat Works
---

**Chapter 7** covers loggers, which are components used for recording error messages and other messages.

分三节介绍 Catalina 中的 log 机制。第一部分介绍 Logger 接口，Catalina 中所有的 logger 都要实现的。第二节介绍 Tomcat 中的 loggers。第三节介绍本章中使用案例。

## The Logger Interface

提供了很多 log 方法，最简单的只需要一个 string 参数即可。定义了五种 log 级别，很常规 FATAL， ERROR，WARNING 等。

## Tomcat‘s Loggers

Tomcat 自带的 logger 有 FileLogger，SystemErrLogger 和 SystemOutLogger. 他本都继承自 LoggerBase 类， LoggerBase 实现了 Logger 接口

{ plantuml }
interface Logger
Logger <|.. logger.LoggerBase

logger.LoggerBase <|-- logger.SystemOutLogger
logger.LoggerBase <|-- logger.SystemErrLogger
logger.LoggerBase <|-- logger.FileLogger
{ endplantuml }

### The LoggerBase Class

LoggerBase 实现了除 log(string) 外的所有接口方法。这个方法的实现放在子类中完成。

### The SystemOutLogger Class

在它的实现中，所有 log 都会被输出到终端

```java
public void log(String msg) {
    System.out.println(msg);
}
```

### The SystemErrLogger Class

与上面雷同，只不过用了 `System.err.println(msg);`

### The FileLogger Class

将 log 记录到文件并附带时间戳信息。按天为单位常见新文件记录信息，允许指定前/后缀。Tomcat 4 中这个类也实现了 Lifecycle 接口，这样它就可以和容器同开同关了。在 Tomcat 5 中，这个接口被移到父类去了。

主要方法实现如下

```java
public void start() throws LifecycleException {
    // Validate and update our current component state
    if (started)throw new LifecycleException (sm.getString("fileLogger.alreadyStarted"));
    lifecycle.fireLifecycleEvent(START_EVENT, null);
    started = true;
}

public void stop() throws LifecycleException {
    // Validate and update our current component state
    if (!started)throw new LifecycleException (sm.getString("fileLogger.notStarted"));
    lifecycle.fireLifecycleEvent(STOP_EVENT, null);
    started = false;
    close();
}

public void log(String msg) {
    // Construct the timestamp we will use, if requested
    Timestamp ts = new Timestamp(System.currentTimeMillis());
    String tsString = ts.toString().substring(0, 19);
    String tsDate = tsString.substring(0, 10);

    // If the date has changed, switch log files
    if (!date.equals(tsDate)) {
        synchronized (this) {
            if (!date.equals(tsDate)) {
                close();
                date = tsDate;
                open();
            }
        }
    }
    // Log this message, timestamped if necessary
    if (writer != null) {
        if (timestamp) {
            writer.println(tsString + " " + msg);
        } else {
            writer.println(msg);
        }
    }
}
```

通常情况下 FileLogger 会操作多个文件，在操作下一个时会把前一个先关掉。

### The open method

先检查文件夹是否存在，没有就建一个。然后在创建对应的 log 文件再写内容。

### The close method

flush + close 流

### The log method

不说了，看代码，很简单的，不过它用系统时间生成如期的做法还是挺好的，以后项目可以借鉴一些。

### The Application

测试代码和第 6 章基本一样，只是在 Bootstrap 部分添加了对 log 文件的定制

```java
// ------ add logger --------
System.setProperty("catalina.base", System.getProperty("user.dir"));
FileLogger logger = new FileLogger();
logger.setPrefix("FileLog_");
logger.setSuffix(".txt");
logger.setTimestamp(true);
logger.setDirectory("webroot");
context.setLogger(logger);
```

访问 `http://localhost:8080/Modern` 后在 webroot 文件夹下面会生成定制的 log 文件 `FileLog_2021-08-02.txt`

这个案例运行的时候大概卡了快一分钟才执行完，看了 log 原来是有 class 找不到, 应该是我用的 jar 包已经是 Catalina 了，不是 tomcat 了，版本太高

```log
2021-08-02 10:38:13 HttpProcessor[8080][4] process.invoke
java.lang.NoClassDefFoundError: org/apache/tomcat/util/log/SystemLogHandler
	at org.apache.catalina.connector.RequestBase.recycle(RequestBase.java:562)
	at org.apache.catalina.connector.HttpRequestBase.recycle(HttpRequestBase.java:417)
	at org.apache.catalina.connector.http.HttpRequestImpl.recycle(HttpRequestImpl.java:195)
	at org.apache.catalina.connector.http.HttpProcessor.process(HttpProcessor.java:1101)
	at org.apache.catalina.connector.http.HttpProcessor.run(HttpProcessor.java:1151)
	at java.lang.Thread.run(Thread.java:836)
Caused by: java.lang.ClassNotFoundException: org.apache.tomcat.util.log.SystemLogHandler
	at java.net.URLClassLoader.findClass(URLClassLoader.java:444)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:480)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:384)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:413)
	... 6 more
```

这个类被放到 tomcat-util 中去了，添加对应的 reference 到 pom 中即可解决

```java
<!-- https://mvnrepository.com/artifact/tomcat/tomcat-util -->
<dependency>
    <groupId>tomcat</groupId>
    <artifactId>tomcat-util</artifactId>
    <version>4.1.31</version>
</dependency>
```
