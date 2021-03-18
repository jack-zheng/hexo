---
title: 链表反转
date: 2021-03-18 19:21:42
categories:
- 算法
tags:
- 链表
- 反转
---

反转一个单链表。

示例:

输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
进阶:
你可以迭代或递归地反转链表。你能否用两种方法解决这道题？

## 迭代法

想象的模型比较重要，我们准备两个空节点，一个用来拼接解析出来的节点，作为返回值，另一个用来持有后续节点，用来循环。

他这变有一个很奇特的点，就是平时我们在做链表操作的时候很多情况是**持有一个最后的节点**通过调用 node1.next = node2 的方式，在尾部拼接。但是这提的解体思路恰恰是相反的的，拿到一个新节点，通过 givenNode.next = node1 这种方式在头部拼接。第一次做的时候这个弯弯很难绕

```java
/**
 * input:  1->2->3->4->5->NULL
 * output: 5->4->3->2->1->NULL
 */
public class ReverseListSample {
    public static void main(String[] args) {
        ListNode fir = new ListNode(1, new ListNode(2, new ListNode(3, new ListNode(4, new ListNode(5)))));
        System.out.println(fir.toString());

        ListNode ret = reverseList(fir);
        System.out.println(ret.toString());
    }

    public static ListNode reverseList(ListNode head) {
        ListNode next;
        ListNode pre = null;
        while (Objects.nonNull(head)) {
            next = head.next;
            head.next = pre;
            pre = head;
            head = next;
        }
        return pre;
    }
}

class ListNode {
    int val;
    ListNode next;

    ListNode(int val) {
        this.val = val;
    }

    ListNode(int val, ListNode next) {
        this.val = val;
        this.next = next;
    }

    @Override
    public String toString() {
        return iterVal(this);
    }

    private String iterVal(ListNode node) {
        if (Objects.isNull(node)) {
            return "null";
        }

        return node.val + " -> " + iterVal(node.next);
    }
}

// 1 -> 2 -> 3 -> 4 -> 5 -> null
// 5 -> 4 -> 3 -> 2 -> 1 -> null
```

## 递归法

核心思想和前面的一致，具体到做表现形式上，我自己想出来的解法需要额外传入一个参数作为结果，貌似有点累赘

```java
public static ListNode reverseList(ListNode head, ListNode result) {
    if (Objects.isNull(head))
        return result;

    ListNode holder = head.next;
    head.next = result;
    return reverseList(holder, head);
}
```