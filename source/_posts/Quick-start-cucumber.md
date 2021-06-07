---
title: Cucumber 弹射起步
date: 2021-05-11 15:39:11
categories:
- 弹射起步
tags:
- cucumber
---

## Scenario Outline

通过这个关键字，我们可以进行一组数据的测试，共能上类似于 TestNG 的 dataProvider

Scenario Outline 需要和 Examples 搭配使用，写了前者，IDEA 会给提示的

```feaure
Feature: user creation related tests

  Scenario Outline: Negative scenario, create user should be fail if field miss in body
    When call REST API to create user and field miss <fieldName> in body
    Then user should create failure, err msg contains <fieldInMsg> show in response
    Examples:
      | fieldName           | fieldInMsg           |
      | provisionerId       | Provisioner ID       |
      | provisionerName     | Provisioner name     |
      | provisionerPassword | Provisioner password |
      | provisionerEmail    | Provisioner email    |
```

对应的 Java 代码

```java
public class CreateUserImpl {

    @When("call REST API to create user and field miss {} in body")
    public void callRESTAPIToCreateProvisionerAndFieldMissFieldNameInBody(String fieldName) {
        System.out.println("fileName: " + fieldName);
    }

    @Then("user should create failure, err msg contains {} show in response")
    public void provisionerShouldCreateFailureErrMsgContainsFieldInMsgShowInResponse(String fieldInMsg) {
        String errMsg = fieldInMsg + "can not be null.";
        System.out.println(errMsg);
    }
}
```

这个 scenario 会运行 4 遍，每一行测试一次

## DataTable

和上面的概念很类似的还有一个叫 DataTable 的概念，他的作用是在一个 Step 中创建多组数据

格式上的区别：DataTable 是不需要 table header 的， 而且不需要 Examples 关键字，在 annotation 里面也不需要占位符

```feature
Scenario: Show my fruits
    When I have kinds of fruits
        | Apple     |
        | Banana    |
    Then show them
```

java 实现代码

```java
@When("I have kinds of fruits")
public void iHaveKindsOfFruits(DataTable dataTable) {
    List<String> list = dataTable.asList();
    System.out.println(Arrays.toString(list.toArray()));
}

@Then("show them")
public void showThem() {
}
```