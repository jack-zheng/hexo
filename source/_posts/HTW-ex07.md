---
title: Ex07 实现 Logger 机制
date: 2021-07-30 19:29:43
categories:
- Tomcat
tags:
- How Tomcat Works
---

**Chapter 7** covers loggers, which are components used for recording error messages and other messages.

分三节介绍 Catalina 中的 log 机制。第一部分介绍 Logger 接口，Catalina 中所有的 logger 都要实现的。第二节介绍 Tomcat 中的 loggers。第三节介绍本章中使用案例。

## The Logger Interface

理论上来说，Logger 可以 attach 到任何 container，但是实际操作中，我们基本只会 attach 到 Context level 以上的 container 中。

```java
public interface Logger {
    public static final int FATAL = Integer.MIN_VALUE; // 严重的
    public static final int ERROR = 1;
    public static final int WARNING = 2;
    public static final int INFORMATION = 3;
    public static final int DEBUG = 4;

    public Container getContainer();

    public void setContainer(Container container);

    public String getInfo();

    public int getVerbosity();

    public void setVerbosity(int verbosity);

    public void addPropertyChangeListener(PropertyChangeListener listener);

    // ------- 一系列重载的 log 方法，有些还支持 verbosity 设置，如果传入的 level 比当前的小就会被记载
    public void log(String message);

    public void log(Exception exception, String msg);

    public void log(String message, Throwable throwable);

    public void log(String message, int verbosity);

    public void log(String message, Throwable throwable, int verbosity);

    public void removePropertyChangeListener(PropertyChangeListener listener);
}
```

提供了很多 log 方法，最简单的只需要一个 string 参数即可。定义了五种 log 级别，还有 container 相关的方法将 logger 实体和 container 结合起来。

## Tomcat's Loggers

Tomcat 自带的 logger 有 FileLogger，SystemErrLogger 和 SystemOutLogger. 他们都继承自 LoggerBase 类， LoggerBase 实现了 Logger 接口

{% plantuml %}
interface Logger
Logger <|.. logger.LoggerBase

logger.LoggerBase <|-- logger.SystemOutLogger
logger.LoggerBase <|-- logger.SystemErrLogger
logger.LoggerBase <|-- logger.FileLogger
{% endplantuml %}

### The LoggerBase Class

Tomcat5 的 LoggerBase 由于结合了 MBeans 的功能，所以变得有些复杂，在 20 章会介绍。这里用 Tomcat4 的做例子。

Tomcat4 中的 LoggerBase 实现了除 log(string) 外的所有接口方法，这个方法的实现放在子类中完成。默认的 verbosity level 是 ERROR。可以通过 setVerbosity() 来改变这个值。

```java
public abstract class LoggerBase implements Logger {
    protected Container container = null;
    protected int debug = 0;
    protected static final String info = "org.apache.catalina.logger.LoggerBase/1.0";
    protected PropertyChangeSupport support = new PropertyChangeSupport(this);
    protected int verbosity = ERROR;

    public Container getContainer() {
        return (container);
    }

    public void setContainer(Container container) {
        Container oldContainer = this.container;
        this.container = container;
        support.firePropertyChange("container", oldContainer, this.container);
    }

    public int getDebug() {
        return (this.debug);
    }

    public void setDebug(int debug) {
        this.debug = debug;
    }

    public String getInfo() {
        return (info);
    }

    public int getVerbosity() {
        return (this.verbosity);
    }

    public void setVerbosity(int verbosity) {
        this.verbosity = verbosity;
    }

    public void setVerbosityLevel(String verbosity) {
        if ("FATAL".equalsIgnoreCase(verbosity))
            this.verbosity = FATAL;
        else if ("ERROR".equalsIgnoreCase(verbosity))
            this.verbosity = ERROR;
        else if ("WARNING".equalsIgnoreCase(verbosity))
            this.verbosity = WARNING;
        else if ("INFORMATION".equalsIgnoreCase(verbosity))
            this.verbosity = INFORMATION;
        else if ("DEBUG".equalsIgnoreCase(verbosity))
            this.verbosity = DEBUG;
    }

    public void addPropertyChangeListener(PropertyChangeListener listener) {
        support.addPropertyChangeListener(listener);
    }

    public abstract void log(String msg);

    public void log(Exception exception, String msg) {
        log(msg, exception);
    }

    public void log(String msg, Throwable throwable) {
        CharArrayWriter buf = new CharArrayWriter();
        PrintWriter writer = new PrintWriter(buf);
        writer.println(msg);
        throwable.printStackTrace(writer);
        Throwable rootCause = null;
        if (throwable instanceof LifecycleException)
            rootCause = ((LifecycleException) throwable).getThrowable();
        else if (throwable instanceof ServletException)
            rootCause = ((ServletException) throwable).getRootCause();
        if (rootCause != null) {
            writer.println("----- Root Cause -----");
            rootCause.printStackTrace(writer);
        }
        log(buf.toString());
    }

    public void log(String message, int verbosity) {
        if (this.verbosity >= verbosity)
            log(message);
    }

    public void log(String message, Throwable throwable, int verbosity) {
        if (this.verbosity >= verbosity)
            log(message, throwable);
    }

    public void removePropertyChangeListener(PropertyChangeListener listener) {
        support.removePropertyChangeListener(listener);
    }
}
```

### The SystemOutLogger Class

在它的实现中，所有 log 都会被输出到终端

```java
public class SystemOutLogger extends LoggerBase {
    protected static final String info = "org.apache.catalina.logger.SystemOutLogger/1.0";

    public void log(String msg) {
        System.out.println(msg);
    }
}
```

### The SystemErrLogger Class

与上面雷同，只不过用了 `System.err.println(msg);`

```java
public class SystemErrLogger extends LoggerBase {
    protected static final String info = "org.apache.catalina.logger.SystemErrLogger/1.0";

    public void log(String msg) {
        System.err.println(msg);
    }
}
```

### The FileLogger Class

FileLogger 是所有 tomcat 子类中最复杂的一个，他会将 log 记录到文件并附带时间戳信息。当初始化时他会创建一个附带当天日期的 log 文件，如果日期变了，他会创建一个新的文件保存日志。我们还可以自定义前缀后缀。

按天为单位常见新文件记录信息，允许指定前/后缀。Tomcat4 中这个类也实现了 Lifecycle 接口，这样我们实现了开启/停止服务的托管。在 Tomcat5 中，这个接口被移到父类去了。

PS: 这个类的 log 方法拿时间戳的方法挺有意思，以后同样的功能可以借鉴一下

```java
import java.sql.Timestamp;

public class TestFileLogger {
    @Test
    public void test() {
        Timestamp ts = new Timestamp(System.currentTimeMillis());
        System.out.println(ts);

        String tsString = ts.toString().substring(0, 19);
        System.out.println(tsString);

        String tsDate = tsString.substring(0, 10);
        System.out.println(tsDate);
    }
}

// 2021-09-03 15:20:49.025
// 2021-09-03 15:20:49
// 2021-09-03
```

实现了 Lifecycle 接口之后，stop/start 方法实现如下：

```java
public class FileLogger extends LoggerBase implements Lifecycle {
    public void start() throws LifecycleException {
        // Validate and update our current component state
        if (started)
            throw new LifecycleException(sm.getString("fileLogger.alreadyStarted"));
        lifecycle.fireLifecycleEvent(START_EVENT, null);
        started = true;
    }

    public void stop() throws LifecycleException {
        // Validate and update our current component state
        if (!started)
            throw new LifecycleException(sm.getString("fileLogger.notStarted"));
        lifecycle.fireLifecycleEvent(STOP_EVENT, null);
        started = false;
        close();
    }
}
```

主要方法 log(String msg) 实现如下

```java
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

通常情况下 FileLogger 会操作多个文件，在操作下一个时会把当前的关掉。

### The open method

先检查目录是否村子啊，没有就建一个。然后再创建 log 文件的 PrintWriter 并返回。

```java
private void open() {

    // Create the directory if necessary
    File dir = new File(directory);
    if (!dir.isAbsolute())
        dir = new File(System.getProperty("catalina.base"), directory);
    dir.mkdirs();

    // Open the current log file
    try {
        String pathname = dir.getAbsolutePath() + File.separator +
            prefix + date + suffix;
        writer = new PrintWriter(new FileWriter(pathname, true), true);
    } catch (IOException e) {
        writer = null;
    }

}
```

### The close method

很简单，flush IO 流并重制变量

```java
private void close() {
    if (writer == null)
        return;
    writer.flush();
    writer.close();
    writer = null;
    date = "";
}
```

### The log method

这小节就是将之前贴的这些方法穿起来描述一下，log 中的工作流如下：

* 调用 lang 包下的类和方法，拿到当前时间戳
* 比较当前时间，如有必要则创建新的 log 文件
* 根据 timestamp 这个 flag 决定是否在写 log 的时候加入时间戳
* 写入 log，打完收工

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

```xml
<!-- https://mvnrepository.com/artifact/tomcat/tomcat-util -->
<dependency>
    <groupId>tomcat</groupId>
    <artifactId>tomcat-util</artifactId>
    <version>4.1.31</version>
</dependency>
```

再后来，遇到了很多麻烦，最后将 Tomcat 源码整合到项目中了，所有问题都没了。。。
