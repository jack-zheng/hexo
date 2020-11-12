---
title: Java tree related code
date: 2020-11-10 15:27:57
categories:
- 编程
tags:
- java
- 树
---

最近在看 TraceSonar 的源码的时候，看到生成树相关的代码， 感觉我自己徒手写应该是没戏了，至少他的这个方案是可以 work 的，同时像收集一下网上能找到的生成树的一些优秀代码

## 实践

```java
import java.util.ArrayList;
import java.util.List;

public class Node<T> {
    private T data;
    private Node<T> parent = null;

    private List<Node<T>> children = new ArrayList<>();

    public Node(T data) {
        this.data = data;
    }

    public Node<T> addChild(Node<T> child) {
        child.setParent(this);
        this.children.add(child);
        return child;
    }

    public void addChildren(List<Node<T>> children) {
        children.forEach(each -> each.setParent(this));
        this.children.addAll(children);
    }

    public List<Node<T>> getChildren() {
        return children;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    private void setParent(Node<T> parent) {
        this.parent = parent;
    }

    public Node<T> getParent() {
        return parent;
    }

    public Node<T> getRoot() {
        if (parent == null) {
            return this;
        }
        return parent.getRoot();
    }
}

@Test
public void test() {
    Node<String> root = new Node<>("root");

    Node<String> node1 = root.addChild(new Node<>("node 1"));

    Node<String> node11 = node1.addChild(new Node<>("node 11"));
    node11.addChild(new Node<>("node 111"));
    node11.addChild(new Node<>("node 112"));

    node1.addChild(new Node<>("node 12"));

    Node<String> node2 = root.addChild(new Node<>("node 2"));

    node2.addChild(new Node<>("node 21"));
    node2.addChild(new Node<>("node 22"));

    printTree(root, " ");
}

private static <T> void printTree(Node<T> node, String appender) {
    System.out.println(appender + node.getData());
    node.getChildren().forEach(each -> printTree(each, appender + appender));
}

// output:
// root
//   node 1
//     node 11
//         node 111
//         node 112
//     node 12
//   node 2
//     node 21
//     node 22
```

这种解法，首先把树这种结构解析出来，单独作为一个载体，你可以根据自己的需求填充树中的内容，其次打印的时候用的 lambda 表达式也很简洁，很喜欢这个例子。

## 参考

[javagists](https://www.javagists.com/java-tree-data-structure)
