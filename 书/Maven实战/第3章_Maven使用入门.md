---
title: 第3章_Maven使用入门
date: 2019-09-05 22:07:11
tags: 
 - Java
 - Maven
categories:
 - Maven
---

# 第3章_Maven使用入门



## maven-compiler-plugin插件

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```



## maven-shade-plugin插件

默认打包生成的jar不能直接运行，因为带有main方法的类信息不会添加到manifest中（jar文件中META-INF/MANIFEST.MF文件，将无法看到Main-Class一行）

借助maven-shade-plugin插件解决：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>top.tomxwd.maven03.HelloWorld</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

