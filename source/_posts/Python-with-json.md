---
title: Python 操作 json
date: 2021-04-28 16:34:03
categories:
- python
tags:
- json
---

## 基操

```python
import json

#### 从 string 加载
person = '{"name": "Bob", "languages": ["English", "Fench"]}'
person_dict = json.loads(person)

#### 从文件加载
with open('path_to_file/person.json') as f:
    data = json.load(f)

#### json 转 string
person_dict = {'name': 'Bob',
'age': 12,
'children': None
}
person_json = json.dumps(person_dict)
# Output: {"name": "Bob", "age": 12, "children": null}

person_dict = {"name": "Bob",
"languages": ["English", "Fench"],
"married": True,
"age": 32
}
#### json 写入文件
with open('person.txt', 'w') as json_file:
    json.dump(person_dict, json_file)


#### 格式化输出
person_string = '{"name": "Bob", "languages": "English", "numbers": [2, 1.6, null]}'

# Getting dictionary
person_dict = json.loads(person_string)

# Pretty Printing JSON string back
print(json.dumps(person_dict, indent = 4, sort_keys=True))
```