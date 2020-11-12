---
title: Springboot add local jar into dependency
date: 2020-11-11 17:18:57
categories:
- 编程
tags:
- java
- 本地
---

Springboot 项目中怎么添加本地 jar 包记录

1. 在项目资源目录下创建一个文件夹，用来存放本地 jar 包
2. 在 pom.xml 中添加 本地 jar 包的引用，引用目录为第一步创建的文件目录
3. 在 pom.xml 的 plugins 中添加编译打包的目录，使本地jar包能打到项目中去

```txt
resources
├── application.properties
├── lib
│   └── tracesonar-0.1-SNAPSHOT.jar (本地包)
├── static
└── templates
```

```xml
<!-- add local lib reference -->
<dependency>
    <groupId>com.github.TraceSonar</groupId>
    <artifactId>TraceSonar</artifactId>
    <version>0.0.3</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/src/main/resources/lib/tracesonar-0.1-SNAPSHOT.jar</systemPath>
</dependency>

<plugins>
	<plugin>
		<artifactId>maven-compiler-plugin</artifactId>
		<configuration>
			<source>1.8</source>
			<target>1.8</target>
			<encoding>UTF-8</encoding>
			<compilerArguments>
				<extdirs>${project.basedir}/src/main/resources/lib</extdirs>
			</compilerArguments>
		</configuration>
	</plugin>
</plugins>
```