---
title: 链表反转
date: 2021-03-18 19:21:42
categories:
- 算法
tags:
- 链表反转
- interview
---

反转一个单链表。

示例:

输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
进阶: 你可以迭代或递归地反转链表。你能否用两种方法解决这道题？

## 思路整理

先想象一下最简单的模型，怎么将第一个元素反转？

```java
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
```

我们可以声明一个空节点，然后把原始节点的第一个节点的 next 引用指向这个空节点就是我们想要的效果了

```java
public class ReverseList {
    public static void main(String[] args) {
        ListNode head = new ListNode(1, new ListNode(2, new ListNode(3, new ListNode(4, new ListNode(5, null)))));

        ListNode empty = null;
        head.next = empty;
        System.out.println(head);
    }
}
// 1 -> null
```

为了遍历后续的节点，我们需要在 head.next = empty 之前用另一变量来持有这个 next 引用 `ListNode holder1 = head.next;`。处理第二个节点的时候，我们其实重复了之前的操作，将它单独拆下来，然后拼接到 head 前面即可。按照这个思路，我可以直接联想使用 while 循环处理这个问题

```java
public class ReverseList {
    public static void main(String[] args) {
        ListNode head = new ListNode(1, new ListNode(2, new ListNode(3, new ListNode(4, new ListNode(5, null)))));
        System.out.println(head);


        ListNode result = null;
        ListNode holder;

        while(head != null) {
            holder = head.next;
            head.next = result;
            result = head;
            head = holder;
        }

        System.out.println(result);
    }
}
// 1 -> 2 -> 3 -> 4 -> 5 -> null
// 5 -> 4 -> 3 -> 2 -> 1 -> null
```

把上面的实现精简一下就是迭代法了 (●°u°●)​ 」

## 迭代法

想象的模型比较重要，我们准备两个空节点，一个用来拼接解析出来的节点，作为返回值，另一个用来持有后续节点，用来循环。

他这边有一个很奇特的点，就是平时我们在做链表操作的时候很多情况是**持有一个最后的节点**通过调用 node1.next = node2 的方式，在尾部拼接。但是这里的解体思路恰恰是相反的的，拿到一个新节点，通过 givenNode.next = node1 这种方式在头部拼接。第一次做的时候这个弯弯很难绕

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

// 1 -> 2 -> 3 -> 4 -> 5 -> null
// 5 -> 4 -> 3 -> 2 -> 1 -> null
```

## 递归法

核心思想和前面的一致，具体到做表现形式上，我自己想出来的解法需要额外传入一个参数作为结果，有点累赘

```java
public static ListNode reverseList(ListNode head, ListNode result) {
    if (Objects.isNull(head))
        return result;

    ListNode holder = head.next;
    head.next = result;
    // 思路还是很清晰的， 先把传入的节点分离出来， 然后在作为 result 的节点前面一次添加分离出来的节点。当所有的节点都处理过后，返回
    return reverseList(holder, head);
}
```

搜索了一下单参数的解法

我们可以用 分治 的观点来看这个问题会更简单。终止条件为判空。否则，我们再次调用递归方法，拿到逆序结果。它骚就骚在一般我们的递归都是直接返回的，而它则是还要做一些操作。

但是值得注意的是，返回的结果是不能参与操作的，不然引用就会发生变化。必须通过现成的变量操作，不然循环就会被打破了。

```java
public static ListNode reverseList(ListNode current) {
    if (current == null || current.next == null)
        return current;

    ListNode next = current.next; // 拿到后续节点
    
    ListNode ret = reverseList(next); // 拿到计算结果
    current.next = null; // 打断当前节点, 防止闭环
    next.next = current; // 习惯性的写成 ret.next = current, 这样会打破迭代规律
    return ret;
}
```