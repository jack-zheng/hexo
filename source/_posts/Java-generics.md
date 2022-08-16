---
title: Java generics
date: 2022-08-10 15:59:15
categories:
- java
tags:
- 泛型
---

泛型类有什么用，格式

```java
// class 类名称 <泛型标识:可以是任意标识符>{
//   private 泛型标识 var; 
//   .....
//   }
// }
public class ArrayList<E> extends AbstractList<E> {}
```

泛型接口有什么作用，格式

```java
public interface Iterable<T> {}
```

泛型方法有什么作用，格式

```java
E elementData(int index) {
    return (E) elementData[index];
}

@SuppressWarnings("unchecked")
static <E> E elementAt(Object[] es, int index) {
    return (E) es[index];
}
```

