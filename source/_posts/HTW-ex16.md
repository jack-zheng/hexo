---
title: Ex16 Shutdown Hook
date: 2021-09-20 14:56:16
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 16** explains the shutdown hook that Tomcat uses to always get a chance to do clean-up regardless how the user stops it (i.e. either appropriately by sending a shutdown command or inappropriately by simply closing the console.)

本章介绍了 Tomcat 如何优雅的执行 stop 方法，前面先通过两个例子介绍实现原理，后面分析 Tomcat 的实现方式

java 中，虚拟机可以通过两种方式 shut down

* 调用 System.exit 结束
* 非正常结束，比如 CTRL+C，直接关闭终端等

当虚拟机结束时，会一次执行下面两个步骤

* jvm 会执行所有注册的 shutdown hook. 这些 hook 被 attach 在 Runtime 对象中，结束时并行执行
* jvm 执行所有没有被调用的 finalizers 方法

本章中的例子都是针对第一点做实现的

创建一个 shutdown hook 的方式很简单，分一下几步

* 创建 class 继承 Thread
* run 方法中实现定制的逻辑
* 在目标应用中，实例话你的 hook class
* 注册 hook 到 Runtime

下面是一个示例程序, 不管怎么结束程序，ShutdownHook 的 run() 方法都会被调用到

```java
public class ShutdownHookDemo {
    public void start() {
        System.out.println("Demo");
        ShutdownHook shutdownHook = new ShutdownHook();
        Runtime.getRuntime().addShutdownHook(shutdownHook);
    }
    public static void main(String[] args) {
        ShutdownHookDemo demo = new ShutdownHookDemo();
        demo.start();
        try {
            System.in.read();
        }catch (Exception e) {}
    }
}

class ShutdownHook extends Thread {
    @Override
    public void run() {
        System.out.println("Shutting down...");
    }
}

// Demo
// Shutting down...
```

## A Shutdown Hook Example

这里通过一个 UI 组件做例子，应用启动时会创建一个临时文件，我们的目标是，推出后，这个文件必须被删除。原始实现如下，当点击 Exit 推出时，文件可以被删除，但是如果通过点击 x 按钮推出，则文件还会保留。

```java
public class MySwingApp extends JFrame {
    JButton exitButton = new JButton();
    JTextArea jTextArea1 = new JTextArea();
    String dir = System.getProperty("user.dir");
    String filename = "temp.txt";

    public MySwingApp() {
        exitButton.setText("Exit");
        exitButton.setBounds(new Rectangle(304, 248, 76, 37));
        exitButton.addActionListener(e -> exitButton_actionPerformed(e));
        this.getContentPane().setLayout(null);
        jTextArea1.setText("Click the Exit button to quit");
        jTextArea1.setBounds(new Rectangle(9, 7, 371, 235));
        this.getContentPane().add(exitButton, null);
        this.getContentPane().add(jTextArea1, null);
        this.setDefaultCloseOperation(EXIT_ON_CLOSE);
        this.setBounds(0, 0, 400, 330);
        this.setVisible(true);
        initialize();
    }

    private void initialize() {
        // create a temp file
        File file = new File(dir, filename);
        try {
            System.out.println("Creating temporary file");
            file.createNewFile();
        } catch (IOException e) {
            System.out.println("Failed creating temporary file.");
        }
    }

    private void shutdown() {
        // delete the temp file
        File file = new File(dir, filename);
        if (file.exists()) {
            System.out.println("Deleting temporary file.");
            file.delete();
        }
    }

    void exitButton_actionPerformed(ActionEvent e) {
        shutdown();
        System.exit(0);
    }

    public static void main(String[] args) {
        MySwingApp mySwingApp = new MySwingApp();
    }
}
```

改进方案是，将 shutdown() 的内容包装到单独的内部类中并实现 Thread 接口。在应用启动时，在 initialize() 方法中通过 hook 的方式注册到 Runtime。这样不管怎么退出，都能保证 shutdown() 方法会被执行

```java
public class MySwingAppWithShutdownHook extends JFrame {

    JButton exitButton = new JButton();
    JTextArea jTextArea1 = new JTextArea();
    String dir = System.getProperty("user.dir");
    String filename = "temp.txt";

    public MySwingAppWithShutdownHook() {
        exitButton.setText("Exit");
        exitButton.setBounds(new Rectangle(304, 248, 76, 37));
        exitButton.addActionListener(e -> exitButton_actionPerformed(e));
        this.getContentPane().setLayout(null);
        jTextArea1.setText("Click the Exit button to quit");
        jTextArea1.setBounds(new Rectangle(9, 7, 371, 235));
        this.getContentPane().add(exitButton, null);
        this.getContentPane().add(jTextArea1, null);
        this.setDefaultCloseOperation(EXIT_ON_CLOSE);
        this.setBounds(0, 0, 400, 330);
        this.setVisible(true);
        initialize();
    }

    private void initialize() {
        // add shutdown hook
        MyShutdownHook shutdownHook = new MyShutdownHook();
        Runtime.getRuntime().addShutdownHook(shutdownHook);
        // create a temp file
        File file = new File(dir, filename);
        try {
            System.out.println("Creating temporary file");
            file.createNewFile();
        } catch (IOException e) {
            System.out.println("Failed creating temporary file.");
        }
    }

    private void shutdown() {
        // delete the temp file
        File file = new File(dir, filename);
        if (file.exists()) {
            System.out.println("Deleting temporary file.");
            file.delete();
        }
    }

    void exitButton_actionPerformed(ActionEvent e) {
        shutdown();
        System.exit(0);
    }

    public static void main(String[] args) {
        MySwingAppWithShutdownHook mySwingApp = new MySwingAppWithShutdownHook();
    }

    private class MyShutdownHook extends Thread {
        public void run() {
            shutdown();
        }
    }
}
```

## Shutdown Hook in Tomcat

Tomcat 也通过同样的原理，在 Catalina 这个类中定义了一个 hook

```java
protected class CatalinaShutdownHook extends Thread {

    public void run() {

        if (server != null) {
            try {
                ((Lifecycle) server).stop();
            } catch (LifecycleException e) {
                System.out.println("Catalina.stop: " + e);
                e.printStackTrace(System.out);
                if (e.getThrowable() != null) {
                    System.out.println("----- Root Cause -----");
                    e.getThrowable().printStackTrace(System.out);
                }
            }
        }

    }
}
```

在 Catalina 运行 start() 方法的时候 attach 到 Runtime 对象中去

```java
Thread shutdownHook = new CatalinaShutdownHook();
//...
Runtime.getRuntime().addShutdownHook(shutdownHook);
```