---
title: Java xml parser compare
date: 2020-11-05 14:49:50
categories:
- 编程
tags:
- java
- xml
---

记录一下公司实际项目中新老两种 XML 解析框架的实现方式，作为以后类似问题的参考

## 老的实现方式

用的是最传统的 SAX 解析模式，优点是很直观，在这个模块规模还小的时候挺好的，只要熟悉 SAX 的使用方式上手很快，但是不管什么功能，在经过十几年的反复堆砌之后都会变成一个难以维护的怪物。

实现的时候，最大的弊端应该是参杂了很多的业务逻辑到解析过程中，就我看来，解析就应该是纯粹的过程，业务相关的验证应该放到解析后做才对！

重构前：

* 最外层有 3k 的代码来解析 xml
* 最外层到 DefaultHandler 接口，中间还有 2 个父类继承关系存放各种公共变量和方法
* 所有的模块组都在一个 class 中维护代码，很臃肿，很多冗余
* 一些公共的 element, 比如 description, label 之类的每个模块都可能出一些奇葩用法，维护更难
* 在最外层的 Handler 实现中需要存储很多变量来存储中间状态的值

实现伪代码

```java
public class LegacyParser extends SuperParser3...SuperParser1 {

    private ParsedResultBean bean;
    private ModuleElement1 element1;
    private ModuleElement2 element1;
    ...
    private ModuleElementN elementn;


    private Map<String, IModuleElement> elementGroup1 = new HashMap();
    ...
    private Map<String, IModuleElement> elementGroupn = new HashMap();

    // 多种重载的构造函数包含 module 各自的 flag 参数，再定制后面 module 内容部的处理逻辑
    public  LegacyParser(ParsedResultBean bean) {...}
    public  LegacyParser(ParsedResultBean bean, boolean moduleFlag1) {...}
    public  LegacyParser(ParsedResultBean bean, boolean moduleFlag1, boolean moduleFlag2) {...}

    // 定义一个集合存储可用解析器
    private Map<String, ElementParser> availableParsers = new HashMap();

    private class ElementParser1 extends BaseElementParser...DefaultHander implements IElementParser {

        @Override
        public String getElementName() {
            return ElementName1;
        }

        @Override
        public void startElement(String uri, String localName, String qName, Attributes attrs) throws SAXException {
          // module logic
          elementGroup1.put(...);
        }

        @Override
        public void endElement(String uri, String localName, String qName) throws SAXException {
            // do something
        }

        private void moduleMehtods(...) {
            // do something
        }
    }

    private class ElementParser2...N extends BaseElementParser...DefaultHander implements IElementParser {}

    // 最外部一个
    @Override
    public void startElement(String uri, String localName, String name, Attributes attrs) throws SAXException {
        if (availableParsers.containsKey(name)) {
            elementParsers.get(name).startElement(uri, localName, name, attrs);
        } else {
            throw new SAXException("Unexpected element " + name);
        }
    }

    @Override
    public void endElement(String uri, String localName, String name, Attributes attrs) throws SAXException {
        if (availableParsers.containsKey(name)) {
            elementParsers.get(name).endElement(uri, localName, name, attrs);
        } else {
            throw new SAXException("Unexpected element " + name);
        }
    }
}
```