---
title: Java HashMap things
date: 2021-03-09 15:16:54
categories:
- java
tags:
- collection
- HashMap
---

今天再看一个 defect 的时候涉及到 HashMap 存储的问题，回头想一下发现自己只对 HashMap 以 key 的 Hash 作为依存储依据这点比较清楚外，其他的印象很模糊，写这篇文章记录一下。想要了解的问题如下：

* [x] HashMap 再存储时是否只用 key 的 hash 做依据，和 value 有关系吗 - 只和 key 有关系，只用 key 的 hashCode 做 hash 后的结果作为判断依据
* [x] HashMap 底层使用什么数据结构存储的 - array + list/tree(红黑树)
* [x] HashMap 的类继承关系

以后对 HashMap 的知识点都可以考虑在这篇中做扩展，做成一个总集篇

## HashMap 图示

```txt
  +-------------------------------------------+ 
  |      |      |       |       |             | 
  | Node | Node | Tree  | Node  | ...         | 
  |      |      |       |       |             | 
  |      |      |       |       |             | 
  |------|------|       |------ |             | 
  | next | next |       | next  |             | 
  +------|------|-------|-------|-------------+ 
     .                                          
     .                                          
     .                                          
 +------+                                       
 | Node |                                       
 |      |                                       
 |      |                                       
 |----- |                                       
 | next |                                       
 +------+                                       
```

## HashMap 的类继承关系

```txt

   I               I                                                                                                                               
  +--------+      +----------------------------+                                                                                                   
  |  Map   |      | Map/Cloneable/Serializable |                                                                                                   
  +--------+      +----------------------------+                                                                                                   
       ^                    ^                                                                                                                      
       |                    |                                                                                                                      
  C    |                    |                                                                                                                      
 +-------------+            |                                                                                                                      
 | AbstractMap |            |                                                                                                                      
 +-------------+            |                                                                                                                      
       ^                    |                                                                                                                      
       |                    |                                                                                                                      
       |                    |                                                                                                                      
       |                    |                                                                                                                      
       |                    |                                                                                                                      
    +-------------------------------+                                                                                                              
    |           HashMap             |                                                                                                              
    |                               |                                                                                                              
    +-------------------------------+ 
```

JDK8 中对 HashMap 的实现做了改动，原先是 Array + link list, 时间复杂度 O(1)+O(n) 当哈希冲突严重时，性能就取决于后者了。新的实现采用 Array + list/tree, 当 list 长度大于 8 时就会将 list 转化为红黑树，即 O(1)+O(logn) 比原先会有提升

## 基本数据结构

Map.Entry: 定义了基本 get/set 方法的接口

HashMap.Node: 单项可延伸的链表结构

```java
final int hash;
final K key;
V value;
Node<K,V> next;
```

LinkedHashMap.Entry: 增加了 before，after 属性，但是没有调用，好奇怪

HashMap.TreeNode: 红黑树实现，extends LinkedHashMap.Entry, 但是在我看来直接继承 HashMap.Node 不是更好？

## put 方法实现

```txt
                +----------------+                             
                |   Start        |                             
                | Give node info |                             
                +----------------+                             
                        |                                      
                        |                                      
                        v                                      
                  +-----------+                                
                  |If Conflict|                                
                  +-----------+                                
              No   /         \   Yes                           
                  /           \                                
                 v             v                               
 +------------------+      +------------------+                
 |  Creae new node  |      | Check key & node |                
 +------------------+      +------------------+                
  |                            /      |      \                 
  |                           /       |       \                
  |                          /        |        \               
  |                         /         |         \              
  |                        /          |          \             
  |                       v           v           v            
  |       +---------------+ +------------------+ +-----------+ 
  |       | key same with | |Node is tree type | |List type  | 
  |       | first element | +------------------+ +-----------+ 
  |       +---------------+    |                  /            
  |                |           |                 /             
  |                |           |                /              
  |                v           v               v               
  |           +-----------------------------------+            
  |-------->  |          Check if resize          |            
              +-----------------------------------+            
                          |                                    
                          |                                    
                          v                                    
                      +--------+                               
                      |  End   |                               
                      +--------+                               
```

```java
// 代码实现

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0) // 如果 tab 是空的，给一个初始 size
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null) // 如果 bucket 位置没有值，直接填充一个
            tab[i] = newNode(hash, key, value, null);
        else { // 如果 bucket 位置上有值，则再看
            Node<K,V> e; K k;
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) // 如果 key 已经存在，将值放入 e 在后面做 value 替换
                e = p;
            else if (p instanceof TreeNode) // 如果是 tree, 则在 tree 后面添加节点
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else { // 处理 linked list 的情况
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) { // 如果是末尾节点，直接 append
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // 如果达到转化阀值，将链表转化为树
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) // 如果 key 重复，跳出循环
                        break;
                    p = e; // 给 p 赋值，继续循环
                }
            }
            if (e != null) { // 对已经存在的 node 做值替换
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

## get 方法实现

```java
/**
* Implements Map.get and related methods.
*
* @param hash hash for key
* @param key the key
* @return the node, or null if none
*/
final Node<K,V> getNode(int hash, Object key) {
   Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
   if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
      if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k)))) // always check first node
            return first;
      if ((e = first.next) != null) {
            if (first instanceof TreeNode) // 如果是 tree 类型，调用 tree 的 get 方法
               return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do { // 否则遍历链表
               if (e.hash == hash &&
                  ((k = e.key) == key || (key != null && key.equals(k))))
                  return e;
            } while ((e = e.next) != null);
      }
   }
   return null;
}
```

## Idea 调试优化

当调试 HashMap 时，默认设置下 Node 只显示 K，V 值，对其他细节，比如静态变量，Node 的 next 都是忽略的，可以通过以下方式查看

**临时方案**

在底部 debug 界面，选中需要查看的 entry, 右键 View as -> Object 即可，但是下次调试是会重制，需要再次设置

**永久方案**

Debug 是选中 tab 下的元素，右键 View as -> Create... -> Apply render to object of type 中输入 `java.util.HashMap$Node` 再 Apply 以下就行了。我这边是自动填充好了的

**查看静态变量**

Customize Data Views -> 勾选 static fields, static final fields

PS: 我本地设置了貌似没什么效果 （；￣ェ￣）

## 参考

* [Debug 设置](https://blog.devwu.com/2018/06/07/IntelliJ%20IDEA%20%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95%E6%97%B6%E6%9F%A5%E7%9C%8B%E6%89%80%E6%9C%89%E5%8F%98%E9%87%8F/)
* [HashMap 原理](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)
* [官方针对 7-8 HashMap 实现修改的说明](http://openjdk.java.net/jeps/180)