---
title: Python 使用 json 序列化
date: 2020-06-13 16:11:08
categories:
- python
tags:
- 序列化
---

## dump Vs dumps

这两个函数都可以用来做序列化，唯一的区别是 dump 需要指定一个 io，比如打开的文件作为输出的地方，而 dumps 默认是以 stdout 做为输出端的，也就是打印在终端

```python
import json
a = {'name': 'jack'}
json.dump(a)
# Out[6]: '{"name": "jack"}'

with open('data.json', 'w') as file:
    json.dump([a, a], file)
# 当前目录下会生产名为 data.json 的文件，内容为 [{"name": "jack"}, {"name": "jack"}]
```

## load Vs loads

有了前面的基础，理解 load 和 loads 也是一个套路，一个直接从你指定的 string 加载，一个从你指定的文件加载

```python
ret = json.loads('{"name": "jack"}')
ret, type(ret)
# Out[11]: ({'name': 'jack'}, dict)

with open('data.json') as file:
    ret = json.load(file)
print(ret)
# [{'name': 'jack'}, {'name': 'jack'}]
```

## 支持中文

写入文件是指定 encoding 和 ensure_ascii 参数，读取时指定 encoding 就可以了

```python
me = {'name': '我'}
with open('dump3.json', 'w', encoding='utf-8') as file:
    json.dump(me, file, ensure_ascii=False)

with open('dump3.json', encoding='utf-8') as file:
    ret = json.load(file)
    print(ret)
# {'name': '我'}
```

## 序列化 Object

序列化对象时可以在 dump(s) 的方法中指定一个自己的序列化规则类, 一种是通过 cls 参数，一种是通过 default 参数。不过有一个需要注意的点是，使用时并不指代整个对象的序列化逻辑，而是对那些不知道怎么序列化的部分给出逻辑，这块挺绕的

```python
# 该例子中，Person 是自定义的类，所以调用 dumps 时，如果直接传入，会抛 exception: TypeError: Object of type Person is not JSON serializable
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

# 可以通过指定 default 参数，给出转化规则

def PersonConvert(person):
    if isinstance(person, Person):
        return person.__dict__
    else:
        raise TypeError

p = Person('jack',30)
json.dumps(p, default=PersonConvert)
# Out[28]: '{"name": "jack", "age": 30}'

# 也可以通过指定 cls 参数，给出转化规则
class PersonEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Person):
            return obj.__dict__
        else:
            return json.JSONEncoder.default(self, obj)

json.dumps(p, cls=PersonEncoder)
# Out[30]: '{"name": "jack", "age": 30}'

# 如果此时我们对 Person 做一下升级，添加一个 datetime 属性
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
        self.create_date = datetime.now()

# 那么之前的函数就不够用了，我们除了要处理 Person 的逻辑，还要处理 datetime 的逻辑
def PersonConvertV2(obj):
    if isinstance(obj, Person):
        return obj.__dict__
    elif isinstance(obj, datetime):
        return obj.timestamp()
    else:
        raise TypeError

p2 = Person('Tom', 31)
json.dumps(p2, default=PersonConvertV2)
# Out[46]: '{"name": "Tom", "age": 31, "create_date": 1592802400.657711}'

# 网上有给出比较多经典的转化方式，在转化过程中会携带 class, module 的信息，为反序列化做准备
def obj_to_dict(obj):
    if isinstance(obj, Person):
        d = {}
        d['__class__'] = obj.__class__.__name__
        d['__module__'] = obj.__module__
        d.update(obj.__dict__)
            return d
        elif isinstance(obj, datetime):
            return obj.timestamp()
        else:
            raise TypeError

json.dumps(p2, default=obj_to_dict)
# Out[54]: '{"__class__": "Person", "__module__": "__main__", "name": "Tom", "age": 31, "create_date": 1592802400.657711}'
```

理解了 encode 的逻辑，decode 也差不多。不过逻辑稍微有点区别，他是在遇到 dict 的时候去做判断的。而且从他的输出看，应该是由内而外的进行解析的。

```python
 def dict_to_obj(d):
    if "level01" in d:
        print("l1: %s" % d)
        return d
    elif "level02" in d:
        print("l2: %s" %d)
        return d
    else:
        raise TypeError

json.loads(jstr, object_hook=dict_to_obj)
# l2: {'level02': 'true', 'age': 30}
# l1: {'level01': 'true', 'name': 'jack', 'info': {'level02': 'true', 'age': 30}}     
# Out[13]: {'level01': 'true', 'name': 'jack', 'info': {'level02': 'true', 'age': 30}}
```

```python

def dict_to_obj(our_dict):
    """
    Function that takes in a dict and returns a custom object associated with the dict.
    This function makes use of the "__module__" and "__class__" metadata in the dictionary
    to know which object type to create.
    """
    if "__class__" in our_dict:
        # Pop ensures we remove metadata from the dict to leave only the instance arguments
        class_name = our_dict.pop("__class__")
        # Get the module name from the dict and import it
        module_name = our_dict.pop("__module__")
        # We use the built in __import__ function since the module name is not yet known at runtime
        module = __import__(module_name)
        # Get the class from the module
        class_ = getattr(module,class_name)
        # Use dictionary unpacking to initialize the object
        obj = class_.__new__(class_)
        for key, value in our_dict.items():
            if key == 'create_date':
                value = datetime.fromtimestamp(value)
            setattr(obj, key, value)
    else:
        obj = our_dict
    return obj

jstr = '{"__class__": "Person", "__module__": "__main__", "name": "Jack", "age": 30, "create_date": 1592805275.55762}'
jstr = '{"name":"jack", "info":{"level02": "true", "age":30}}'
o = json.loads(jstr, object_hook=dict_to_obj)
print(o.create_date)
print(type(o.create_date))
# 2020-06-22 13:54:35.557620
# <class 'datetime.datetime'>
```

## 其他的一些收获

* 在 class 的方法中可以有一个 toJSON 的方法快速得到序列化的字符串
* 在 class 的构造函数里可以有一个 dict 参数用来快速构造对象

```python
class Person04:
    def __init__(self, name='', age=-1, pairs=None):
        self.name = name
        self.age = age
        if pairs:
            self.__dict__ = pairs

    def toJSON(self):
        return json.dumps(self,
                            default=lambda o: o.__dict__,
                            sort_keys=True,
                            indent=4)
```
