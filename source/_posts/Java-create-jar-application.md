---
title: 创建可运行 jar 应用
date: 2021-11-11 15:15:50
categories:
- java
tags:
- jar
---

gradle, maven 都提供了现成的打 jar 功能

## Maven 

这里以 maven 为例，创建工程如下

```text
.
├── java
│   └── com
│       └── jk
│           └── Main.java
└── resources
```

```java
package com.jk;

public class Main {
    public static void main(String[] args) {
        for (int i = 0; i < args.length; i++) {
            System.out.println("Index: " + i + ", Value: " + args[i]);
        }
    }
}
```

在 pom 文件中添加配置

```xml
<build>
    <plugins>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>2.5.5</version>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>com.jk.Main</mainClass>
                    </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </plugin>

    </plugins>
</build>
```

终端输入命令：`mvn package assembly:single`, 运行后在 src 同级目录下会生产 target 文件夹，其中包含了对应的 jar 包, 选择带 with-dependencies 的那个

```text
java -jar app-jar-1.0-SNAPSHOT-jar-with-dependencies.jar hello world 
Index: 0, Value: hello
Index: 1, Value: world
```

PS: 直接选 plugins -> assembly -> assembly:single 不 work，不是很了解为啥，需要系统学一下 maven 才知道
