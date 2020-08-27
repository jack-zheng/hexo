---
title: Equals in python
date: 2020-06-12 14:00:21
- 编程
tags:
- python
---

* `==` 和 `is` 的区别
* 怎么使得 Object 使用 `==` 比较相等
* Set 集合中判断相等

## == Vs is

`==` 用来判断值相等，`is` 用来判断引用相等

```python
a = [1, 2, 3]
b = a

a == b # true
a is b # true

b = a[:]
a == b # true
a is b # false
```

## 怎么使得 Object 使用 `==` 比较相等

你需要重写 class 的 __eq__ 方法

```python
class Person:
    def __init__(self, id, name):
        self.id = id
        self.name = name

    def __eq__(self, other):
        if not isinstance(other, Person):
            # don't attempt to compare against unrelated types
            return NotImplemented
        return self.id == other.id

p1 = Person(1, 'a')
p2 = Person(2, 'b')
p3 = Person(1, 'c')

p1 == p2 # false
p1 == p3 # true
```

Note: 重写 __eq__ 将会使对象变为 unhashable，在存到 Set， Map 等集合中会有影响，你可以重写 __hash__ 来定制

## Set 集合中判断相等

```python
class Person:
    def __init__(self, id, name):
        self.id = id
        self.name = name

    def __eq__(self, other):
        if not isinstance(other, Person):
            # don't attempt to compare against unrelated types
            return NotImplemented
        return self.id == other.id

    def __hash__(self):
        # necessary for instances to behave sanely in dicts and sets.
        return hash(self.id)

set([p1, p2, p3]) # only  p1, p2 will be stored
```

set 中并没有使用新的 object 代替旧的的方法，所以如果想要更新的话只能 remove + add 了
