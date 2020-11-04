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
* EntityResolver: 提供获取外部文件的方法， Spring 在介些 xml 的时候也定义过这哥方法，可以参考下
* DTDHandler: 这个类都没有使用例子，是不是一个很冷门的类啊 （；￣ェ￣） 以后有机会看到再记录把
* ContentHandler: 负责处理 xml 节点的逻辑
* ErrorHandler: 结合 DTD 处理异常
* systemId: 外部资源(多半是DTD)的URI，比如本地文件 file:///usr/share/dtd/somefile.dtd 或者网络某个地址的文件 http://www.w3.org/somefile.dtd
* publicId: 和 systemId 类似，区别在于**间接性**
  * publicID 就相当于一个名字，这个名字代表了一个外部资源。比如，我们规定”W3C HTML 4.01″这个字符串对应 "http://www.w3.org/somedir/somefile.dtd" 这个资源。那么，publicID="W3C HTML 4.01" 和 systemID="http://www.w3.org/somedir/somefile.dtd" 是一样的，二者都引用了 http://www.w3.org/somedir/somefile.dtd 作为该文档的外部DTD。
* xmlReader.setFeature(url, flag): 用来表示某个特定的验证规则是否打开了
* XML schema, 就是我们在 Spring 项目中经常能看到的 `.xsd` 文件，他是 DTD 的替代品，支持的验证功能更多，格式和 XML 一致

基本套路：

1. 自定义一个 hander 继承 DefaultHandler, 重写其中的解析逻辑
2. 客户端代码中通过 SAXParserFactory 拿到 parser
3. parser 中传入要解析的文件和自定义 handler
4. parse 是 handler 中定义的 bean 被解析
5. parse 完成后重 handler 中拿到解析结果

下面展示的例子都存在 mybatis 的 repo 的

## TODO: 

* 我们现在工程中用到的解析 xml 的方式看上去很像官方文档中说的 Filter 模式，稍后仔细阅读一下
* add URL here
* SAX 是怎么做到事件触发的，光想想找不到思路。。。得看看源码

## 资源

* [官方文档](http://www.saxproject.org/quickstart.html)
* [Orelly 书集](https://docstore.mik.ua/orelly/xml/sax2/index.htm)

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

## SAXParser Vs XMLReader

SAXParser 和 XMLReader 的关系：SAXParser 隶属于 javax 包， XMLReader 是从 saxproject 这个项目拿过来的。他们都可以读取 xml, SAXParser 底层还是调用了 XMLReader, 前者调用简单，只提供常规用法，后者使用稍显繁琐，但是可以实现的定制化功能多。

类关系上：SAXParser 继承了 AbstractSAXParser, AbstractSAXParser 实现了 XMLReader 接口

```java
@Test
public void saxParser_vs_xmlReader() throws ParserConfigurationException, SAXException, IOException {
    String emp =
            "<Employee id=\"1\">\n" +
            "        <age>29</age>\n" +
            "        <name>Pankaj</name>\n" +
            "        <gender>Male</gender>\n" +
            "        <role>Java Developer</role>\n" +
            "    </Employee>";

    MyHandler handler = new MyHandler();

    // Parse with sax parser
    System.out.println("SaxParser result: ");
    SAXParserFactory factory = SAXParserFactory.newInstance();
    SAXParser saxParser= factory.newSAXParser();
    saxParser.parse(new InputSource(new StringReader(emp)), handler);
    System.out.println(handler.getEmpList());

    System.out.println("XMLReader result: ");
    XMLReader xmlReader = SAXParserFactory.newInstance().newSAXParser().getXMLReader();
    xmlReader.setContentHandler(handler);
    xmlReader.parse(new InputSource(new StringReader(emp)));
    System.out.println(handler.getEmpList());
}

//output, 第二次打印的时候
// SaxParser result: 
// [Employee(id=1, name=Pankaj, gender=Male, age=29, role=Java Developer)]
// XMLReader result: 
// [Employee(id=1, name=Pankaj, gender=Male, age=29, role=Java Developer), Employee(id=1, name=Pankaj, gender=Male, age=29, role=Java Developer)]
```

## ErrorHandler + DTD 验证 XML

在原来的基础上，我们想要在解析 xml 的时候添加一些限制，比如 Employee 元素必须包含 gender 不然抛错。这种功能可以通过添加 DTD 规则并且打开 xml 验证功能来实现。

定义 DTD 文件，DTD 中会包含 xml 各个 element 的从属关系，可以设置的属性值，属性数量等信息。如下方的例子中我们就规定 Employee 元素必须有至少一个的 gender 信息， 并且 Employee 必须包含 id 属性

```java
public class MyErrorHandler implements ErrorHandler {
    @Override
    public void warning(SAXParseException exception) throws SAXException {
        show("--Warning--", exception);
        throw (exception);
    }

    @Override
    public void error(SAXParseException exception) throws SAXException {
        show("--Error--", exception);
        throw (exception);
    }

    @Override
    public void fatalError(SAXParseException exception) throws SAXException {
        show("--Fatal Error--", exception);
        throw (exception);
    }

    private void show(String type, SAXParseException e) {
        System.out.println(type + ": " + e.getMessage());
        System.out.println("Line " + e.getLineNumber() + " Column " + e.getColumnNumber());
        System.out.println("System ID: " + e.getSystemId());
    }
}

@Test
public void test() throws ParserConfigurationException, SAXException, IOException {
    String str_with_dtd =
            "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n" +
            "<!DOCTYPE Employees [\n" +
            "        <!ELEMENT Employees (Employee)*>\n" +
            "        <!ELEMENT Employee (age?, name?, gender+, role*)>\n" +
            "        <!ATTLIST Employee\n" +
            "                id CDATA #REQUIRED\n" +
            "                >\n" +
            "        <!ELEMENT age (#PCDATA)>\n" +
            "        <!ELEMENT name (#PCDATA)>\n" +
            "        <!ELEMENT gender (#PCDATA)>\n" +
            "        <!ELEMENT role (#PCDATA)>\n" +
            "        ]>\n" +
            "<Employees>\n" +
            "    <Employee id=\"1\">\n" +
            "        <age>29</age>\n" +
            "        <name>Pankaj</name>\n" +
//                "        <gender>Male</gender>\n" +
            "        <role>Java Developer</role>\n" +
            "    </Employee>\n" +
            "</Employees>";

    XMLReader xmlReader = SAXParserFactory.newInstance().newSAXParser().getXMLReader();
    xmlReader.setFeature("http://xml.org/sax/features/validation", true);
    xmlReader.setErrorHandler(new MyErrorHandler());
    xmlReader.parse(new InputSource(new StringReader(str_with_dtd)));
}

// output:
// --Error--: The content of element type "Employee" must match "(age?,name?,gender+,role*)".
// Line 18 Column 16
// System ID: null

// org.xml.sax.SAXParseException; lineNumber: 18; columnNumber: 16; The content of element type "Employee" must match "(age?,name?,gender+,role*)".
```

**Note:** 在这个例子中我们只能用 XMLReader, 应为我们只实现了 ErrorHandler 接口。SaxParser 只能处理继承了 DefaultHandler 的类

## EntityResolver samples

Spring中使用DelegatingEntityResolver 类为 EntityResolver的实现类

```java
@Override
public InputSource resolveEntity(String publicId, String systemId) throws SAXException, IOException {
    if (systemId != null) {
        // 如果是DTD从这里开始
        if (systemId.endsWith(DTD_SUFFIX)) {
            return this.dtdResolver.resolveEntity(publicId, systemId);
        }
        // 如果是XSD从这里开始
        else if (systemId.endsWith(XSD_SUFFIX)) {
            // 通过调用META-INF/Spring.schemas解析
            return this.schemaResolver.resolveEntity(publicId, systemId);
        }
    }
    return null;
}
```

例二:

```java
public class MyEntityResolver implements EntityResolver {
    @Override
    public InputSource resolveEntity(String publicId, String systemId) {
        System.out.println(String.format("----- Call MyEntityResolver, PID: %s + SID: + %s", publicId, systemId));
        return null;
    }
}

@Test
public void test() throws ParserConfigurationException, SAXException, IOException {
    String str_with_dtd = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n" +
            "<!DOCTYPE succession-data-model PUBLIC \"Self_defined_plublic_name\" \"http://self/defined/public/name\">\n" +
            "<succession-data-model>\n" +
            "</succession-data-model>";

    XMLReader xmlReader = XMLReaderFactory.createXMLReader();
    xmlReader.setEntityResolver(new MyEntityResolver());
    xmlReader.parse(new InputSource(new StringReader(str_with_dtd)));
}

//output:
// ----- Call MyEntityResolver, PID: Self_defined_plublic_name + SID: + http://self/defined/public/name

// java.net.UnknownHostException: self
// at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:196)
// ...
```

在输出了 publicId 和 systemId 之后，他会试图通过 http 拿到 inputStream 中指定的文件数据，但是我随便写的，所以报错了，但是实验目的已经达到了

## External DTD/XSD sample

这部分我们可以等到以后看 spring 或者 mybatis 解析 xml 的时候看，直接是现成的例子， 他是通过 EntityResolver 指定的解析规则
