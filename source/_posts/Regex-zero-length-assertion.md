---
title: 零宽断言
date: 2021-04-28 15:11:39
categories:
- regex
tags:
- 零宽断言
---

## Positive Lookahead, 零宽度正预测先行断言: 匹配以 xx 结尾的, (?=XX)

特性： 这个断言是从右边开始匹配

```
e.g.01 
use .*(?=ing) as pattern to match 'cooking, doing' 
matched: 'cooking, do'

e.g.02
use [a-z]*(?=ing) as pattern to match 'cooking, doing, singing' 
matched: 'cook, do, sing'
```

## Negtive Lookahead, 负向零宽先行断言: 匹配不以 xx 结尾的, (?!XX)

特性： 这个断言是从右边开始匹配
```
e.g.01 
pattern: yolo(?!lo)
matched: yolo yololo yolo
result: 匹配到第一个 yolo 和最后一个 yolo
refer: https://regex101.com/r/UyC8Z1/1/

感觉上零宽断言适合做 词 这个level 的 match, 普通的 regex 经常会带进来空格
```

## Positive Lookbehind, 负向零宽先行断言: 匹配以 xx 开始的, (?<=XX)

特性： 这个断言是从左边开始匹配
```
e.g.01 
use (?<=abc).* to match abcdefgabc
result: defgabc, not abcdefg
```

## Negative Lookbehind, 负向零宽先行断言: 匹配不以 xx 开始的, (?<!XX)

特性： 这个断言是从左边开始匹配
```
e.g.01
use (?<!yo)lol to match lol yolol
result: the first lol on left
```

## Refer

* [Regex Online](https://regex101.com/)
* [Sample Blog Source, CN](http://www.ibloger.net/article/31.html)
* [Sample Blog Source](https://coderwall.com/p/5c7kjq/lookahead-and-lookbehind-regex)