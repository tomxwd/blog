---
title: Maven基础
date: 2019-06-04 ‏‎22:06:57
tags: 
 - Java
 - Maven
categories:
 - Java环境
 - Maven
---

# Maven基础

> B站黑马视频
>
> Maven是Apache公司的开源项目，是项目的构建工具，用来依赖管理

## maven的好处
1. 能够有效的减少项目体积，一台电脑上只需要下载一次相同版本的依赖即可。
2. 在项目需要jar包的时候，去仓库进行引用，所以Maven项目比传统的导入lib再build path所需要的内存小很多。

## maven的好处如何实现
1. 有个jar包仓库（Maven仓库）
2. 坐标：作为jar包拥有全球唯一的坐标
3. 举例： struts2-core-2.3.24.jar
4. Apache（公司名称）+Struts2（项目名）+2.3.24（版本信息）
5. 当Maven项目中需要一个jar包的时候，只需要在maven项目中配置需要的jar包的坐标信息，maven程序根据jar包坐标的信息去jar包仓库中查找jar包。
6. **maven的两大核心：**
  - **依赖管理：**对jar包的统一管理。
  - **项目构建：**项目在编码完成之后，对项目进行编译、测试、打包、部署等一系列操作都可以命令来实现。

## maven安装、配置本地仓库
1. maven的下载安装
  - 直接上官网下载zip包即可。
  - 解压在没有中文的路径。
  - 目录结构
    1. bin：可执行的脚本命令(mvn)
    2. conf：配置文件（settings.xml)
    3. lib：maven项目运行需要的jar包，因为maven是纯java开发的。
    4. boot：类加载器。
  - 配置环境变量
    1. 配置%MAVEN_HOME%到环境变量中，到bin的上一级。
    2. 配置path环境变量：%MAVEN_HOME%\bin;

    **注意**：maven需要java环境，JDK环境变量：%JAVA_HOME%
  - 使用mvn -v查看maven版本，以及是否安装成功。

2. Maven找jar包流程
  - Maven项目会在本地找仓库，如果有则依赖进去，但是随着jar包越来越多，项目越来越大，每个人的电脑里都需要很大的空间来做maven仓库，这时候就可以通过私服来解决，存在于本地的局域网内的一台服务器，存jar包，只需要在这个局域网里就可以依赖到。前提是安装了私服服务器。最后如果再找不到，就到中央仓库，在互联网上，存放了所有开源的jar包，由Maven团队来维护。
  - 步骤：
    1. 本地仓库：本机
    2. 私服：局域网
    3. 中央仓库：互联网

3. 配置本地仓库
  - 默认在.m2文件夹
  - 自定义
    1. 在maven目录的conf文件夹里有个settings.xml文件，更改里面的localRepository标签，指向自己的仓库位置。

## maven项目标准目录结构
1. **src**：项目的源码
  - main：主要的
    1. java：代码等。
    2. resources：配置文件等。
    3. webapp：页面的素材等。
  - test：测试的
    1. java：代码等。
    2. resources:配置文件等。
2. **pom.xml**：project object module，maven项目的核心配置文件。
3. **target**：src里的源码编译完后，存放在此。

## maven的常用命令
1. **clean：**清理target，即是class文件。
  ```
  由于Maven是JAVA来构建的一个项目，
  在maven的文件目录下有个lib，
  lib里是相关的jar包，
  有个插件是maven-clean-plugin，
  当用clean命令的时候其实是相当于调用哪个jar包。
  ```
2. **compile:**编译项目，即是生成target文件夹，class文件。背后也是调用lib里的一个jar包——maven-compile-plugin。
3. **test：**单元测试，即是把test下的测试都运行一遍。
  - 先compile编译，test/java里的java文件，再运行。
  - **有要求：**单元测试的类名：XxxTest.java
4. **package：**以往是通过Export来导出jar file或者war file，现在可以通过package来打包。
  - 先compile再test最后package。
  - 在target目录下生成jar/war包。
5. **install：**安装，本地多个项目公用一个jar包。
  - compile->test->package->install，最后在本地仓库可以找到。
  - **操作：**可以把一些工具类打包，放到本地仓库，其他项目可以直接去依赖，这样的好处是如果工具类更改了代码，其他项目只需要更新一下maven项目就可以了，不需要把jar包重新导入。

## maven项目的生命周期（了解）
- 在maven中存在三套生命周期，每一套生命周期相互独立，互不影响。
  1. CleanLifeCycle：清理生命周期：``clean``
  2. defaultLifeCycle：默认生命周期：``compile->test->package->install->deploy``
  3. siteLifeCycle：站点生命周期``site``描述这个项目的相关信息，如依赖了哪些jar包。
- 在一套生命周期里，执行后面的命令，前面的命令会自动执行。

## maven整合web项目案例
1. 配置eclipse中配置maven环境
  - 配置m2e插件，基本都会自带maven插件
  - 需要配置maven程序
    1. Perferences->Maven->Installations改为自己的Maven，不使用内嵌的Maven配置。
  - 配置User Settings：让eclipse知道maven仓库的位置。
    1. Perferences->Maven->User Settings改为自己Maven->conf下的settings.xml文件。
  - **构建索引：**show view->maven Repository选择本地仓库右键Rebuild Index。

## maven整合servlet
1. 创建一个Maven Project，普通的Maven项目或者父工程。
2. 创建一个简单的maven项目，create a simple project。
3. **Group Id：**组，公司名称。公司域名的倒序。
4. **Artifact Id：**项目名称。
5. **Version：**版本信息。*SNAPSHOT*：快照版本、测试版本。*RELEASES*：正式版本。
6. **Packaging：**jar则为jar包，war则为web工程，pom则为聚合工程父工程。
7. 简单maven工程需要自己去定义一些文件，如war工程需要自己去加WEB-INF下面的web.xml文件，JDK版本也需要自己去定义，否则Update项目会退回默认的JDK版本，指定了版本也没用。
8. 添加项目jdk编译插件：
```
  <build>
    <plugins>
      <!-- 设置版本为1.8 -->
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugins</artifactId>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF-8</encoding>
      </configuration>
    </plugins>
  </build>
```
9. 创建一个servlet，编译报错，因为缺失jar包：serlvet-api-xx.jar。
  - 查找依赖加在pom.xml文件中。
  - ``<dependencies>``标签下各种``<dependencie>``标签，各个``<dependencie>``标签就代表一个jar包。

## maven依赖范围（``<scope>``)
| 依赖范围 | 对于编译classpath有效 | 对于测试classpath有效 | 对于运行时classpath有效 | 例子|
| :-: | :-: | :-: | :-: || :-: |
| compile | Y | Y | Y | spring-core|
|test|-|Y|-|Junit|
|provided|Y|Y|-|servlet-api|
|runtime|-|Y|Y|JDBC驱动|
|system|Y|Y|-|本地的，Maven仓库之外的类库|
  - 默认是compile
  - 例如tomcat有servlet，则servlet在运行时不需要，因为tomcat里有。

## 运行以及调试maven项目
  1. 右键项目Run As：Maven build，Goals：tomcat:run
  2. 右键项目Debug As：Maven build,需要关联源码，否则无法跳到代码中。

## maven的概念模型
![maven的概念模型](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Maven/01maven%E7%9A%84%E6%A6%82%E5%BF%B5%E6%A8%A1%E5%9E%8B.jpg)
  1. 上半部分是核心之一：依赖管理。
  2. 下半部分是核心之二：项目构建，通过命令来构建项目，如打包、清理等操作。

## 总结
  1. 安装
  2. Maven标准的目录结构

    - ProjectName
    - src
      - main
        - java
        - resources
        - [webapp/WEB-INF/web.xml]
      - test
        - java
        - resources
    - pom.xml
  3. Maven的常用命令

    - clean
    - compile
    - test
    - package：项目的根目录target目录
    - install：本地仓库
  4. 使用eclipse开发maven项目

    - 区别：
      1. 不拷贝jar包
      2. 项目的目录结构不同
  5. pom.xml:项目对象模型

    - 本项目的坐标信息
    - 本项目jdk编译版本的信息
    - 本项目需要的依赖的坐标的信息
