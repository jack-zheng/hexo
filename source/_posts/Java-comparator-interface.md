---
title: Comparator 比较器
date: 2021-11-17 17:20:45
categories:
- java
tags:
- comparator
---

看 Spring 的 BeanFactoryProcessor 部分的时候，看到 sort processor 方法中用了 Comparator 做比较依据， 已经忘的差不多了，复习一下

创建一个简单的对象，有 name 和 age 属性，分别新建两个比较器，一个是 name 字母顺序，一个是 age 大小逆序。自定义的比较器实现 Comparator 接口并重写 compare 方法即可

```java
public class ComparatorTest {

	public static void main(String[] args) {
		A a1 = new A("a", 1);
		A a2 = new A("b", 2);
		List<A> list = new ArrayList<>();
		list.add(a1);
		list.add(a2);

		System.out.println("before sort: " + list);
		list.sort(new AgeSort());
		System.out.println("after sort age: " + list);
		list.sort(new NameSort());
		System.out.println("after sort name: " + list);
	}
}
// before sort: [A(name=a, age=1), A(name=b, age=2)]
// after sort age: [A(name=b, age=2), A(name=a, age=1)]
// after sort name: [A(name=a, age=1), A(name=b, age=2)]

/**
 * age 逆序
 */
class AgeSort implements Comparator<A> {

	@Override
	public int compare(A o1, A o2) {
		return o2.getAge() - o1.getAge();
	}
}

/**
 * name 顺序
 */
class NameSort implements Comparator<A> {

	@Override
	public int compare(A o1, A o2) {
		return o1.getName().compareTo(o2.getName());
	}
}

@Data
@AllArgsConstructor
class A {
	private String name;
	private int age;
}
```