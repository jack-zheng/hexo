---
title: JUnit5 快速入门
date: 2022-09-22 15:03:17
categories:
- java
tags:
- JUnit5
---

进度：最终只是草草的过了一遍官方文档，知道了他只用法，以后要用到的时候再深入看看吧

最近用到 SAP 的 scimono 这个 repo, 里面用到了 JUnit5，使用中发现和之前用的 TestNG 相比增加了很多有意思的特性，特意看下官方文档顺便写一些 demo 实践一下。

官方文档：[JUnit5](https://junit.org/junit5/docs/current/user-guide/)

PS: JUnit5 有一个相关的子项目 junit-platform-console-standalone，提供一个叫 ConsoleLauncher 的东西，可以通过命令行运行测试，给出的结果 UI 还挺好看

### 2.8.1 Operating System and Architecture Conditions

JUnit 中竟然还自带操作系统的删选，有点意思。下面的例子中，只有 MAC 相关的 case 能够执行。

PS: 标签的 compose 使用还是挺有意思的，就是在新的标签上添加已有标签实现功能继承，有机会可以深入看看他是怎么实现这个特性的。

```java
class QuickGuide {
    @Test
    @EnabledOnOs(MAC)
    void onlyOnMacOs() {
        System.out.println("onlyOnMacOs");
    }

    @TestOnMac
    void testOnMac() {
        System.out.println("testOnMac");
    }

    @Test
    @EnabledOnOs({ LINUX, MAC })
    void onLinuxOrMac() {
        System.out.println("onLinuxOrMac");
    }

    @Test
    @DisabledOnOs(WINDOWS)
    void notOnWindows() {
        System.out.println("notOnWindows");
    }

    @Test
    @EnabledOnOs(WINDOWS)
    void onWindows() {
        System.out.println("notOnWindows");
    }

    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    @Test
    @EnabledOnOs(MAC)
    @interface TestOnMac {
    }
}

// Disabled on operating system: Mac OS X
// notOnWindows
// testOnMac
// onlyOnMacOs
// onLinuxOrMac
```

除了操作系统，还有很多其他的筛选条件可供选择

* 根据芯片架构，5.7 可配 `@DisabledOnOs(architectures = "x86_64")`
* 根据JRE版本 `@DisabledForJreRange(max = JAVA_11)`
* 根据JVM删选 `@EnabledInNativeImage`
* 根据系统的 property 配置删选 `@EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")`
* 根据环境变量 `@EnabledIfEnvironmentVariable(named = "ENV", matches = "staging-server")`
* 自定义条件, 通过自己制定方法返回结果判断是否执行 UT, 示例如下，效果上和 ExecutionCondition 很像

```java
class QuickGuide3 {
    @Test
    @EnabledIf("customCondition")
    void enabled() {
        System.out.println("enabled");
    }

    @Test
    @DisabledIf("customCondition")
    void disabled() {
        System.out.println("disabled");
    }

    boolean customCondition() {
        return true;
    }
}
// enabled
// @DisabledIf("customCondition") evaluated to true
```

## 2.11 Test Instance Lifecycle

JUnit 会在每个 test case 执行的时候新建一个 test instance 来达到隔离的目的，如果想要每个 class 共用一个 test instance, 可以在 class 上添加 `@TestInstance(Lifecycle.PER_CLASS)` 注释

## 2.12. Nested Tests

通过 `@Nested` 注解可以将测试以一种更结构化的形式组织起来

## 2.13. Dependency Injection for Constructors and Methods

可以通过 TestInfo 作为测试或这 class 的参数，提供一些环境信息，比如 case 名字，method 名字，annotation 信息等。

## 2.14. Test Interfaces and Default Methods

介绍如何在不使用 mock lib 的情况下测试 interface，不是很明白，用到再看

## 2.16. Parameterized Tests

通过 @ParameterizedTest + @ValueSource 实现 TestNG 中 dataProvider 的效果

## 2.18. Dynamic Tests

动态生成测试用例
