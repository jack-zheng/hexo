---
title: TestNG Jmockit 使用案例
date: 2020-07-07 14:15:56
categories:
- 编程
tags:
- java
- testng
- jmockit
---

记录一下工作中常用到的 TestNG, Jmockit 使用案例

## DataProvider

### 单个参数

```java
    @DataProvider(name = "singleParam")
    public Object[][] singleParam() {
        return new Object[][]{
                {"Jerry"},
                {"Tom"},
        };
    }

    @Test(dataProvider = "singleParam")
    public void single_data(String username) {
        System.out.println("Get username: " + username);
    }
```

### 多个参数

```java
    @DataProvider(name = "multiParam")
    public Object[][] multiParam() {
        return new Object[][]{
                {"Jerry", 12},
                {"Tom", 11},
        };
    }

    @Test(dataProvider = "multiParam")
    public void single_data(String username, int age) {
        System.out.println("Get username: " + username + ", age: " + age);
    }
```

## Mocked 作用域

如果是 global 参数，那么所有 class 内的 case 都会有影响，如果是 method level 的那只有对应的 case 有影响

```java
class Teacher{
  public String name = "unnamed";

  public Teacher(String name) {
    this.name = name;
  }
}

@Mocked Teacher teacher;
@Test
public void test() { System.out.println(teacher.name); } //output: null

@Test
public void test02 () { System.out.println(new Teacher("Jack").name); } //output: null
```

如果做 method level 的 mock, 只作用 case 本身

```java
@Test
public void test(@Mocked Teacher teacher) { System.out.println(teacher.name); } // null

@Test
public void test02 () { System.out.println(new Teacher("Jack").name); } // Jack
```

## Jmockit 和 TestNG 兼容性问题

TestNG 6.9.11+ 和 Jmockit 有兼容性问题，将 @Mocked 通过参数方式传入会抛 Exception

```java
public class CompatibleTest {
  @Test
  public void test(@Mocked UserBean userBean) {}
}

//output:
// org.testng.internal.reflect.MethodMatcherException:
// Data provider mismatch
// Method: test([Parameter{index=0, type=com.objects.UserBean, declaredAnnotations=[@mockit.Mocked(stubOutClassInitialization=false)]}])
// Arguments: []
```

修复方法：将 @Mocked 部分提取改为 global 的变量即可

```java
public class CompatibleTest {
  @Mocked UserBean userBean;
  
  @Test
  public void test() {}
}
```

如果我还想保留这种 case level 的使用，需要做点什么？这种 case level 的使用在作用域控制上更好

TODO

## Mock 不带默认构造函数的对象

构建一个测试对象时，如果他没有模式构造函数的话需要为参数声明 @Injectable

```java
// 测试对象
class Dog{
  private String name;

  public Dog(String name) {
    this.name = name;
  }

  public String getName() {
    return name;
  }
}

public class CompatibleTest {
  @Tested Dog dog;
  @Injectable String name;
  
  @Test
  public void test() { dog.getName(); }
}

// 如果没加的话抛出异常
// java.lang.IllegalArgumentException: No constructor in tested class that can be satisfied by available injectables
//   public com.successfactors.legacy.service.provisioning.impl.Dog(String)
//     disregarded because no injectable was found for parameter "name"
```

## Mockup 工厂方法

```java
/**
* new object + mockup, new object 发生在 mock 之后，所以 mock 生效
*/
@Test
public void mock_factory_using_mockup() {
    new MockUp<NPCFactory>() {
        @Mock
        public Person getNPC() {
            return new Person("mock", 1);
        }
    };

    ClassRoom classRoom = new ClassRoom();
    assertEquals("mock", classRoom.getNPCName());
}

public class ClassRoom {
    private Person npc = NPCFactory.getNPC();
    public String getNPCName() { return npc.getName(); }
}
```

## 使用 Deencapsulation 设置私有变量，高版本已经 deprecated

```java
/**
* new object + expectations, new object 发生在 mock 之后，所以 mock 生效
*/
@Test
public void mock_factory_using_deencapsulation(@Mocked final Person person) {
    new Expectations() {{
        person.getName();
        result = "deenMock";
    }};

    Deencapsulation.setField(room, "npc", person);
    assertEquals("deenMock", room.getNPCName());
}
```

## 通过 Expectations case level mock 静态方法

```java
/**
* new object + expectations, new object 发生在 mock 之后，所以 mock 生效
*/
@Test
public void mock_factory_using_expectations() {
new Expectations(NPCFactory.class) {{
    NPCFactory.getNPC();
    result = new Person("expMock", 2);
}};

ClassRoom classRoom = new ClassRoom();
assertEquals("expMock", classRoom.getNPCName());
}
```

## 部分 mock/PartialMock

```java
@Tested Person person;

@Test
public void person_name_jack() {
    new Expectations(person) {{
        person.getName();
        result = "jack";
    }};

    assertEquals("jack", person.getName());
    assertEquals(0, person.getAge());
}
```

partial 对非修饰类型有效吗？有效

`new Expectations(ClassA.class)` 会对这个 class 的所有实例生效，`new Expectations(instance)` 则只会对当前这个 instance 起作用，范围更精确

## 获取 Logger 引用做验证

如果你在 UT 中想要验证某条 log 有没有打印出来，你可以使用 `@Capturing` annotation。

> 相比于 @Mocked 而言，@Capturing 最大的特点是，他用于修饰 父类或者接口，那么他的所有实现类都会被 mocked 掉。对 log 的案例来说，我们为 Logger 这个 interface 加上这个注释之后，后续所有的实现都被 mock 掉，然后我们再做验证

```java
// Tested Class
public class MySubscriber {

  private static final Logger LOGGER = LogManager.getLogger(MySubscriber.class);

  @Override
  public void onEvent(Event event) {
    if (LOGGER.isInfoEnabled()) {
      LOGGER.info("Start Process MySubscriber...");
      LOGGER.info("End...");
    }
  }
}

// In UT
@Capturing
private Logger logger;

@Test
public void test_capturing_anno() {
  new Expectations(ReadAuditSwitchHelper.class) {{
    logger.isInfoEnabled();
    result = true;
  }};

  subscriber.onEvent(context, event);

  new Verifications() {{
    logger.isInfoEnabled(); times=1;
    List<String> capturedInfos = new ArrayList<>();
    logger.info(withCapture(capturedInfos));

    capturedInfos.stream().forEach(System.out::println);
  }};
}
```

## 获取方法参数

```java
// 如果是单个参数
new Verifications() {{
  double d;
  String s;
  mock.doSomething(d = withCapture(), null, s = withCapture());

  assertTrue(d > 0.0);
  assertTrue(s.length() > 1);
}};

// 如果是多个参数
new Verifications() {{
  List<DataObject> dataObjects = new ArrayList<>();
  mock.doSomething(withCapture(dataObjects));

  assertEquals(2, dataObjects.size());
  DataObject data1 = dataObjects.get(0);
  DataObject data2 = dataObjects.get(1);
  // Perform arbitrary assertions on data1 and data2.
}};
```

## @Mocked 导致 equals 方法失效

今天写 UT 的时候遇到一个问题，当我使用 @Mocked 修饰一个类时，这个累的所有引用都会被 mock 掉，虽然知道有这种特性，但是以前都没有碰到问题，忽视了，debug 花了好久。

示例如下：

准别两个简单的 MyBean 和 MyField, MyField 是 MyBean 的一个属性，并在声明时就做了初始化。

对应的 UT 可以 work，但当我对 MyField 添加 @Mocked 注解时，对应的 equals 方法会被抹去，UT 就挂了。

解决方案有两种：1. 不用 @Mocked; 2. 只做方法层面的 mock

对于第二种方法，testng 升级到 6.1 之后需要配合 @DataProvider 使用，变得麻烦了，也不知道后面的版本会不会修复这个问题

```java
public class MyBean {
    private MyField field = new MyField();

    // getter/setter

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        MyBean myBean = (MyBean) o;
        return Objects.equals(field, myBean.field);
    }

    @Override
    public int hashCode() {
        return Objects.hash(field);
    }
}

public class MyField {
    private String name;

    // getter/setter

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        MyField myField = (MyField) o;
        return Objects.equals(name, myField.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}

public class TestMockedAnno01 {
  // @Mocked MyField field;

  @Test
  public void test() {
    MyBean bean1 = new MyBean();
    MyBean bean2 = new MyBean();

    Assert.assertEquals(bean1, bean2);

  }
}
```