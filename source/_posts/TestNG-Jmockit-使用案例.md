---
title: 'TestNG, Jmockit 使用案例'
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
