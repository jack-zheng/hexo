---
title: Java 操作符优先级
date: 2020-10-22 15:14:01
categories:
- 编程
tags:
- java
- 优先级
---

记录一下工作终于到的操作符优先级的问题，总体的逻辑是：从左到右依次计算

[Java Operator Precedence](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html)

|      Operators       |                Precedence                |
| :------------------: | :--------------------------------------: |
|       postfix        |              expr++ expr--               |
|        unary         |      ++expr --expr +expr -expr ~ !       |
|    multiplicative    |                  * / %                   |
|       additive       |                   + -                    |
|        shift         |                << >> >>>                 |
|      relational      |           < > <= >= instanceof           |
|       equality       |                  == !=                   |
|     bitwise AND      |                    &                     |
| bitwise exclusive OR |                    ^                     |
| bitwise inclusive OR |                    \|                    |
|     logical AND      |                    &&                    |
|      logical OR      |                   \|\|                   |
|       ternary        |                   ? :                    |
|      assignment      | = += -= *= /= %= &= ^= \| = <<= >>= >>>= |


## 逻辑与 和 逻辑非

```java
if (null != params && params.isFeatureExist(FeatureEnum.FEATURE_01) && !(ElementTypeEnum.ADDRESS.equals(Element.getElementTypeEnum()) || ElementTypeEnum.BUSINESS_ADDRESS.equals(Element.getElementTypeEnum())) && !ElementTypeEnum.PERSON_GLOBAL_INFO.getElementId().equals(Element.getId()))
```

从左到右依次判断就完事儿啦。。。感觉上面表格中的 逻辑与 > 逻辑非 的表示还混淆了我的判断（；￣ェ￣）
