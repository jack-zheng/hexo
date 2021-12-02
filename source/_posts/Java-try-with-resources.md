---
title: try-catch-with-resources 语法中各分支的执行顺序
date: 2021-12-02 15:23:58
categories:
- java
tags:
- tree
---

Refer: [StackOverflow](https://stackoverflow.com/questions/24129088/are-resources-closed-before-or-after-the-finally)

结论：先执行 try 中内容，再执行关闭资源的行为，然后是 catch 最后再 finally

```java
public class CloseableDummy implements Closeable {
  public void close() {
    System.out.println("closing");
  }
}

public class CloseableDemo {
  public static void main(String[] args) {
    try (CloseableDummy closableDummy = new CloseableDummy()) {
      System.out.println("try exit");
      throw new Exception();
    } catch (Exception ex) {
      System.out.println("catch");
    } finally {
      System.out.println("finally");
    }
  }
}

// try exit
// closing
// catch
// finally
```