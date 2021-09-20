---
title: Ex15 Digester
date: 2021-09-17 15:24:45
categories:
- Tomcat
tags:
- How Tomcat Works
---

> **Chapter 15** explains the configuration of a web application through Digester, an exciting open source project from the Apache Software Foundation. For those not initiated, this chapter presents a section that gently introduces the digester library 
and how to use it to convert the nodes in an XML document to Java objects. It then explains the ContextConfig object that configures a StandardContext instance.

PS: 创建完 project 后将 lib 添加到项目的 classpath 中, 不然会缺少依赖

本章先介绍一个 xml 解析工具包 Digester，它是给予 SAX 的解析工具，然后介绍了 Tomcat 中解析配置文件的类 StandardContext 和 ContextConfig。最后实验部分，可以看到主函数中我们并没有声明 StandardWrapper 但是通过解析 xml 程序还是能正常运行。

## Digester

Digester 是 Apache 的 Jakarta 项目的一个子项目，是对 SAX 的封装和抽象，使用起来挺简单的，下面是几个小例子.

比如我们有一下 xml 文件并切想要将它解析成 employee 对象

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<employee firstName="Brian" lastName="May">
</employee>
```

我们可以使用如下代码, 先定义好 employee 的结构

```java
public class Employee {
    private String firstName;
    private String lastName;

    public Employee() {
        System.out.println("Creating Employee");
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        System.out.println("Setting firstName : " + firstName);
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        System.out.println("Setting lastName : " + lastName);
        this.lastName = lastName;
    }

    public void printName() {
        System.out.println("My name is " + firstName + " " + lastName);
    }
}
```

然后调用 Digester 方法解析，主要方法解释如下

* addObjectCreate: 添加一条创建对象的 rule，应该就是创建指定 element 的对象的意思
* addSetProperties: 添加一条 set properties 的 rule，应该就是将属性 set 到对象里的意思
* addCallMethod: 添加一条调用方法的 rule
* parse: 使用定制好的 rule 解析文件流

```java
public class Test01 {

    public static void main(String[] args) {
        String path = System.getProperty("user.dir") + File.separator  + "etc";
        File file = new File(path, "employee1.xml");
        Digester digester = new Digester();
        // add rules
        digester.addObjectCreate("employee", "com.jzheng.digestertest.Employee");
        digester.addSetProperties("employee");
        digester.addCallMethod("employee", "printName");

        try {
            Employee employee = (Employee) digester.parse(file);
            System.out.println("First name : " + employee.getFirstName());
            System.out.println("Last name : " + employee.getLastName());
        }
        catch(Exception e) {
            e.printStackTrace();
        }
    }
}

// Creating Employee
// Setting firstName : Brian
// Setting lastName : May
// My name is Brian May
// First name : Brian
// Last name : May
```

上面的例子是单个 node 的情况，如果 xml 是嵌套的话，可以如下处理

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<employee firstName="Freddie" lastName="Mercury">
    <office description="Headquarters">
        <address streetName="Wellington Avenue" streetNumber="223"/>
    </office>
    <office description="Client site">
        <address streetName="Downing Street" streetNumber="10"/>
    </office>
</employee>
```

根据实际情况，将对应的 Office 和 address 的定义写出来

```java
public class Office {
    private Address address;
    private String description;
    public Office() {
        System.out.println("..Creating Office");
    }
    public String getDescription() {
        return description;
    }
    public void setDescription(String description) {
        System.out.println("..Setting office description : " + description);
        this.description = description;
    }
    public Address getAddress() {
        return address;
    }
    public void setAddress(Address address) {
        System.out.println("..Setting office address : " + address);
        this.address = address;
    }
}

public class Address {
    private String streetName;
    private String streetNumber;

    public Address() {
        System.out.println("....Creating Address");
    }

    public String getStreetName() {
        return streetName;
    }

    public void setStreetName(String streetName) {
        System.out.println("....Setting streetName : " + streetName);
        this.streetName = streetName;
    }

    public String getStreetNumber() {
        return streetNumber;
    }

    public void setStreetNumber(String streetNumber) {
        System.out.println("....Setting streetNumber : " + streetNumber);
        this.streetNumber = streetNumber;
    }

    public String toString() {
        return "...." + streetNumber + " " + streetName;
    }
}
```

主体部分唯一的区别就是在嵌套对象的表示和 addSetNext 方法。当元素是嵌套在内部的元素时，通过斜杠(/)表示自元素

* addSetNext(String pattern, String method): pattern 表示处理的对象类型，method 表示触发的方法。如 `digester.addSetNext("employee/office", "addOffice");` 就表示当遇到 addOffice(Office office) 时，将两个对象关联起来

```java
public class Test02 {

    public static void main(String[] args) {
        String path = System.getProperty("user.dir") + File.separator  + "etc";
        File file = new File(path, "employee2.xml");
        Digester digester = new Digester();
        // add rules
        digester.addObjectCreate("employee", "com.jzheng.digestertest.Employee");
        digester.addSetProperties("employee");
        digester.addObjectCreate("employee/office", "com.jzheng.digestertest.Office");
        digester.addSetProperties("employee/office");
        digester.addSetNext("employee/office", "addOffice");
        digester.addObjectCreate("employee/office/address", "com.jzheng.digestertest.Address");
        digester.addSetProperties("employee/office/address");
        digester.addSetNext("employee/office/address", "setAddress");
        try {
            Employee employee = (Employee) digester.parse(file);
            ArrayList offices = employee.getOffices();
            Iterator iterator = offices.iterator();
            System.out.println("-------------------------------------------------");
            while (iterator.hasNext()) {
                Office office = (Office) iterator.next();
                Address address = office.getAddress();
                System.out.println(office.getDescription());
                System.out.println("Address : " +
                        address.getStreetNumber() + " " + address.getStreetName());
                System.out.println("--------------------------------");
            }

        }
        catch(Exception e) {
            e.printStackTrace();
        }
    }
}

// Creating Employee
// Setting firstName : Freddie
// Setting lastName : Mercury
// ..Creating Office
// ..Setting office description : Headquarters
// ....Creating Address
// ....Setting streetName : Wellington Avenue
// ....Setting streetNumber : 223
// ..Setting office address : ....223 Wellington Avenue
// Adding Office to this employee
// ..Creating Office
// ..Setting office description : Client site
// ....Creating Address
// ....Setting streetName : Downing Street
// ....Setting streetNumber : 10
// ..Setting office address : ....10 Downing Street
// Adding Office to this employee
// -------------------------------------------------
// Headquarters
// Address : 223 Wellington Avenue
// --------------------------------
// Client site
// Address : 10 Downing Street
// --------------------------------
```

当 rule 很多时，还可以将这些 rule 封装到一个类中

```java
public class EmployeeRuleSet extends RuleSetBase {
    public void addRuleInstances(Digester digester) {
        // add rules
        digester.addObjectCreate("employee", "com.jzheng.digestertest.Employee");
        digester.addSetProperties("employee");
        digester.addObjectCreate("employee/office", "com.jzheng.digestertest.Office");
        digester.addSetProperties("employee/office");
        digester.addSetNext("employee/office", "addOffice");
        digester.addObjectCreate("employee/office/address", "com.jzheng.digestertest.Address");
        digester.addSetProperties("employee/office/address");
        digester.addSetNext("employee/office/address", "setAddress");
    }
}
```

对应的主体简化为

```java
public class Test03 {

    public static void main(String[] args) {
        String path = System.getProperty("user.dir") + File.separator  + "etc";
        File file = new File(path, "employee2.xml");
        Digester digester = new Digester();
        digester.addRuleSet(new EmployeeRuleSet());
        try {
            Employee employee = (Employee) digester.parse(file);
            ArrayList offices = employee.getOffices();
            Iterator iterator = offices.iterator();
            System.out.println("-------------------------------------------------");
            while (iterator.hasNext()) {
                Office office = (Office) iterator.next();
                Address address = office.getAddress();
                System.out.println(office.getDescription());
                System.out.println("Address : " + address.getStreetNumber() + " " + address.getStreetName());
                System.out.println("--------------------------------");
            }

        }
        catch(Exception e) {
            e.printStackTrace();
        }
    }
}
```

## ContextConfig

StandardContext 必须要设置一个 ContextConfig 才能正常工作，它只要干几件事，设置 flag，设置 valve，解析 xml. ContextConfig 是以 listener 的形式出现的，监听 start/stop 事件。加载 xml 的逻辑在 defaultConig() 和 applicationConfig() 中。

### defaultConfig

解析 conf/web.xml 的内容

```java
private void defaultConfig() {

    // Open the default web.xml file, if it exists
    File file = new File(Constants.DefaultWebXml);
    if (!file.isAbsolute())
        file = new File(System.getProperty("catalina.base"),
                        Constants.DefaultWebXml);
    FileInputStream stream = null;
    try {
        stream = new FileInputStream(file.getCanonicalPath());
        stream.close();
        stream = null;
    } catch (FileNotFoundException e) {
        log(sm.getString("contextConfig.defaultMissing"));
        return;
    } catch (IOException e) {
        log(sm.getString("contextConfig.defaultMissing"), e);
        return;
    }

    // Process the default web.xml file
    synchronized (webDigester) {
        try {
            InputSource is =
                new InputSource("file://" + file.getAbsolutePath());
            stream = new FileInputStream(file);
            is.setByteStream(stream);
            webDigester.setDebug(getDebug());
            if (context instanceof StandardContext)
                ((StandardContext) context).setReplaceWelcomeFiles(true);
            webDigester.clear();
            webDigester.push(context);
            webDigester.parse(is);
        } catch (SAXParseException e) {
            log(sm.getString("contextConfig.defaultParse"), e);
            log(sm.getString("contextConfig.defaultPosition",
                                "" + e.getLineNumber(),
                                "" + e.getColumnNumber()));
            ok = false;
        } catch (Exception e) {
            log(sm.getString("contextConfig.defaultParse"), e);
            ok = false;
        } finally {
            try {
                if (stream != null) {
                    stream.close();
                }
            } catch (IOException e) {
                log(sm.getString("contextConfig.defaultClose"), e);
            }
        }
    }
}
```

### The applicationConfig Method

applicationConfig 的逻辑和 defaultConfig 基本一致，只是将解析的路径改了, 解析的路径为 `/WEB-INF/web.xml`

### Creating Web Digester

创建解析用的 digester

```java
private static Digester createWebDigester() {

    URL url = null;
    Digester webDigester = new Digester();
    webDigester.setValidating(true);
    url = ContextConfig.class.getResource(Constants.WebDtdResourcePath_22);
    webDigester.register(Constants.WebDtdPublicId_22,
                            url.toString());
    url = ContextConfig.class.getResource(Constants.WebDtdResourcePath_23);
    webDigester.register(Constants.WebDtdPublicId_23,
                            url.toString());
    webDigester.addRuleSet(new WebRuleSet());
    return (webDigester);

}
```

WebRuleSet 中就是解析的规则，比如下面这些是用来解析 filter 的规则

```java
digester.addObjectCreate(prefix + "web-app/filter", "org.apache.catalina.deploy.FilterDef");
digester.addSetNext(prefix + "web-app/filter", "addFilterDef", "org.apache.catalina.deploy.FilterDef");
```

整个程序段应该是解析之前，通过 digester.push(context) 将要解析的对象塞进去，然后通过 parse(is) 关联起来的

PS: Digester 和 SAX 的整合关系分析，有兴趣可以做一做