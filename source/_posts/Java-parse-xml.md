---
title: Java parse xml
date: 2020-11-03 18:57:40
categories:
- 编程
tags:
- java
- xml
---

简单记录一下 Java 解析 xml 的例子。

## 基本概念

* DefaulHandler: 为了简化代码将几个常用的 handler 合并为这个 DefaultHandler
* EntityResolver
* DTDHandler
* ContentHandler
* ErrorHandler
* systemId: 外部资源(多半是DTD)的URI，比如本地文件 file:///usr/share/dtd/somefile.dtd 或者网络某个地址的文件 http://www.w3.org/somefile.dtd
* publicId: 和 systemId 类似，区别在于**间接性**
  * publicID 就相当于一个名字，这个名字代表了一个外部资源。比如，我们规定”W3C HTML 4.01″这个字符串对应 "http://www.w3.org/somedir/somefile.dtd" 这个资源。那么，publicID="W3C HTML 4.01" 和 systemID="http://www.w3.org/somedir/somefile.dtd" 是一样的，二者都引用了 http://www.w3.org/somedir/somefile.dtd 作为该文档的外部DTD。

TODO: add URL here

基本套路：

1. 自定义一个 hander 继承 DefaultHandler, 重写其中的解析逻辑
2. 客户端代码中通过 SAXParserFactory 拿到 parser
3. parser 中传入要解析的文件和自定义 handler
4. parse 是 handler 中定义的 bean 被解析
5. parse 完成后重 handler 中拿到解析结果

下面展示的例子都存在 mybatis 的 repo 的

## Parse xml 并生成对应的实体类

```java
@Data
public class Employee {
    private int id;
    private String name;
    private String gender;
    private int age;
    private String role;
}

// 自定义 handler，解析 element 并被 emp bean 赋值
public class MyHandler extends DefaultHandler {

    // List to hold Employees object
    private List<Employee> empList = null;
    private Employee emp = null;
    private StringBuilder data = null;

    // getter method for employee list
    public List<Employee> getEmpList() {
        return empList;
    }

    boolean bAge = false;
    boolean bName = false;
    boolean bGender = false;
    boolean bRole = false;

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {

        if (qName.equalsIgnoreCase("Employee")) {
            // create a new Employee and put it in Map
            String id = attributes.getValue("id");
            // initialize Employee object and set id attribute
            emp = new Employee();
            emp.setId(Integer.parseInt(id));
            // initialize list
            if (empList == null)
                empList = new ArrayList<>();
        } else if (qName.equalsIgnoreCase("name")) {
            // set boolean values for fields, will be used in setting Employee variables
            bName = true;
        } else if (qName.equalsIgnoreCase("age")) {
            bAge = true;
        } else if (qName.equalsIgnoreCase("gender")) {
            bGender = true;
        } else if (qName.equalsIgnoreCase("role")) {
            bRole = true;
        }
        // create the data container
        data = new StringBuilder();
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        if (bAge) {
            // age element, set Employee age
            emp.setAge(Integer.parseInt(data.toString()));
            bAge = false;
        } else if (bName) {
            emp.setName(data.toString());
            bName = false;
        } else if (bRole) {
            emp.setRole(data.toString());
            bRole = false;
        } else if (bGender) {
            emp.setGender(data.toString());
            bGender = false;
        }

        if (qName.equalsIgnoreCase("Employee")) {
            // add Employee object to list
            empList.add(emp);
        }
    }

    @Override
    public void characters(char ch[], int start, int length) throws SAXException {
        data.append(new String(ch, start, length));
    }
}

// 测试用例
@Test
public void test_myhandler() {
    SAXParserFactory saxParserFactory = SAXParserFactory.newInstance();
    try {
        SAXParser saxParser = saxParserFactory.newSAXParser();
        MyHandler handler = new MyHandler();

        ClassLoader classLoader = getClass().getClassLoader();
        File file = new File(Objects.requireNonNull(classLoader.getResource("employees.xml")).getFile());

        saxParser.parse(file, handler);
        //Get Employees list
        List<Employee> empList = handler.getEmpList();
        //print employee information
        for (Employee emp : empList)
            System.out.println(emp);
    } catch (ParserConfigurationException | SAXException | IOException e) {
        e.printStackTrace();
    }
}
```

测试用 employees.xml

```xml
<Employees>
    <Employee id="1">
        <age>29</age>
        <name>Pankaj</name>
        <gender>Male</gender>
        <role>Java Developer</role>
    </Employee>
    <Employee id="2">
        <age>35</age>
        <name>Lisa</name>
        <gender>Female</gender>
        <role>CEO</role>
    </Employee>
    <Employee id="3">
        <age>40</age>
        <name>Tom</name>
        <gender>Male</gender>
        <role>Manager</role>
    </Employee>
    <Employee id="4">
        <age>25</age>
        <name>Meghna</name>
        <gender>Female</gender>
        <role>Manager</role>
    </Employee>
</Employees>
```

## TODO: add EntityResolver-ErrorHandler samples

## Internal dtd sample

## External dtd sample