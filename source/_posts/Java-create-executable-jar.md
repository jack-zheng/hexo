---
title: 创建可执行 jar
date: 2022-08-30 15:56:14
categories:
- java
tags:
- 可执行文件
- jar
---

今天在做 feature 的时候需要用到 SCIMono 这个项目里的 compliance test。看了一下它的执行逻辑，他是通过 jar 运行的，之前没有做过这种类型的东西，特别记录一下怎么创建类似的 jar 包。项目地址

* [executable-jar](https://github.com/jack-zheng/executable-jar)

## idea 创建创建可执行 jar

* [参考 CSDN](https://blog.csdn.net/ouyang111222/article/details/73105086) 

1. 创建测试 project，添加测试类 jk.Main 并添加测试方法 psvm 类型的即可
2. 右键 project -> Open Module Settings -> 选中 Artifacts
3. 在界面中间部分点击 '+' 号，选择 JAR -> From modules with dependencies
4. 'Main Class:' 中填入入口 class 的**全路径**, 也可以点文件夹 icon 选择，更方便
5. 点击 OK, idea 自动为你生成 MANIFEST.MF 文件。借助 idea 很多配置自动生成了，避免了很多错误
6. 在同一个界面的最右侧，勾选 'include in project build'，这个 jar 就会在 build 的时候自动创建，生产路径在同一界面给出了
7. 选择窗口顶部的 Build, 然后选择 rebuild project 或者 build artifacts 都行，可以看到目录树那边有生产 out 文件夹，里面就有对应的jar
8. 展开 out, 选中 jar 所在文件夹，右键 open in -> Terminal, 输入 `java -jar executable-jar.jar jack` 和 `java -jar executable-jar.jar` 测试

测试类代码如下

```java
public class Main {
    public static void main(String[] args) {
        if(args.length > 0) {
            System.out.println("Hello " + args[0]);
        } else {
            System.out.println("Hello world");
        }
    }
}
```

```MANIFEST.MF
Manifest-Version: 1.0
Main-Class: jk.Main
```

## 通过 maven shade 插件简化创建过程

* [官方文档](https://maven.apache.org/plugins/maven-shade-plugin/)
* [Git Repo](https://github.com/apache/maven-shade-plugin)

官方文档有挺详细的说明文档，还包括使用例子。基本流程可以概括为，在 pom 文件中添加配置，然后在项目中运行 `maven package` 即可, 不过我看好多文章都写的 `maven clean package` 也不知道哪里抄的。运行结束后可以在 target 下看到两个 jar 包，不以 origin 开头的就是我们需要的那个 jar 包。使用这个 plugin 省去了很多配置的工作。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.3.0</version>
    <executions>
        <execution>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <!-- 指定入口程序 -->
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                    <manifestEntries>
                        <Main-Class>jk.Main</Main-Class>
                    </manifestEntries>
                </transformer>
                    <!-- 整合 MF -->
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## 其他问题

在 SCIMono 项目中，他将控制台收到的参数存到 System.properties 中，case 中从系统配置中那这些变量来使用。本地测试的时候可以在 UT 的 configuration 中添加 `-Dkey=value` 的方式添加这些配置
