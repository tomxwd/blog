---
title: 01_IDEA的安装、配置与使用
date: 2019-06-16 ‏‎11:03:39
tags: 
 - IDEA
 - Java
categories:
 - 工具
 - IDEA使用
---

# 01_IDEA的安装、配置与使用

> [视频链接](https://www.bilibili.com/video/av30080993)

## IDEA介绍

#### JetBrains公司介绍

[IDEA](https://www.jetbrains.com/idea/)是JetBrains公司的产品，公司旗下还有其他产品，比如：

- WebStorm：用于开发JavaScript、HTML5、CSS3等前端技术；
- PyCharm： 用于开发Python；
- PhpStorm：用于开发PHP；
- RubyMine：用于开发Ruby/Rails；
- AppCode：用于开发Objective-C/Swift；
- CLion：用于开发C/C++；
- DataGrip：用于开发数据库和SQL；
- Rider：用于开发.NET；
- GoLand：用于开发Go语言；
- Android Studio：用于开发android（google基于IDEA社区版进行迭代，因此也是与JetBrains公司相关）



---

## IntelliJ IDEA介绍

IDEA，全称IntelliJ IDEA，是Java语言的集成开发环境，IDEA在业界被公认为是最好的Java开发工具之一，尤其是在智能代码助手、代码自动提示、重构、J2EE支持、Ant、JUnit、CVS整合、代码审查、创新的GUI设计等方面的功能可以说是超常的。

IntelliJ IDEA在2015年的官网上这样介绍自己：

IntelliJ IDEA主要用于支持Java、Scala、Groovy等语言的开发工具，同时具备支持目前主流的技术和框架，擅长于企业应用、移动应用和Web应用的开发。



---

## IDEA的主要功能介绍

语言支持上：

| 安装插件后支持 |   SQL类    | 基本JVM |
| :------------: | :--------: | :-----: |
|      PHP       | PostgreSQL |  Java   |
|     Python     |   MySQL    | Groovy  |
|      Ruby      |   Oracle   |         |
|     Scala      | SQL Server |         |
|     Kotlin     |            |         |
|    Clojure     |            |         |

其他支持：

|  支持的框架  | 额外支持的语言代码提示 | 支持的容器 |
| :----------: | :--------------------: | :--------: |
|  SpringMVC   |         HTML5          |   Tomcat   |
|     GWT      |          CSS3          |   TomEE    |
|    Vaadin    |          SASS          |  WebLogin  |
|     Play     |          LESS          |   JBoss    |
|    Grails    |       JavaScript       |   Jetty    |
| Web Services |      CoffeeScript      | WebSphere  |
|     JSF      |        Node.js         |            |
|    Struts    |      ActionScript      |            |
|  Hibernate   |                        |            |
|     Fiex     |                        |            |



---

## IDEA的主要优势：（相较于Eclipse而言）

1. 强大的整合能力。比如：Git、Maven、Spring等。
2. 提示功能的快速、便捷。
3. 提示功能的范围更广。
   - 比如在Mapper.xml文件中编写SQL语句的时候有提示。
4. 好用的快捷键和代码模块。
5. 精准搜索。



---

## IDEA的下载地址：（官网）

> [官网windows版本下载地址](https://www.jetbrains.com/idea/download/#section=windows)

IDEA分为两个版本：旗舰版和社区办。

旗舰版为收费（30天免费试用），社区办免费。

![两个版本之间的区别](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea1.png)

这里提供了不同操作系统下的两个不同版本的安装文件。

两个不同版本的详细对比，可以参照[官网](http://www.jetbrains.com/idea/features/editions_comparison_matrix.html)。



---

## 官网提供的IDEA详细使用文档

> [文档地址](https://www.jetbrains.com/help/idea/meet-intellij-idea.html)

#### 安装前准备：

硬件要求：个人建议配置，内存8G或以上，CPU最好i5以上，最好安装固态硬盘（SSD），将IDEA安装在固态硬盘中，这样流畅度会高很多。

软件要求：IDEA本身绑定了JRE1.8，但是要自己另外安装JDK1.8进行开发。



---

## 卸载

除了安装路径之外，在C盘，用户下有个.IntelliJIdeaXxx目录。里面有两个文件（config，system）。

config：用户的一些设置：代码提示、字体风格、插件等等。

system：包括一些缓存



---

## 安装

**路径**：全英文路径，保留版本号信息等。

**安装选项**：

![安装选项](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea2.png)

**安装总结**：从安装上看，IDEA对硬件要求似乎不是很高，但是实际在开发中其实并不是这样的，因为IDEA执行时会有**大量缓存、索引文件**。所以如果如果你正在使用Eclipse，想通过使用IDEA来解决电脑的卡顿等问题，基本不可能，本质上应该对自己的硬件设备进行升级。

**安装目录**：

1. bin目录下：启动文件，虚拟机配置信息，idea基本属性信息
   - idea64.ext.vmoptions文件：可以更改下面的信息。
     - Xms128m：初始的内存数，可以改成500。
     - Xmx750m：最大的内存数，增加到1500，可以降低垃圾回收的频率，提高性能。
     - XX:ReservedCodeCacheSize=240m：代码缓存的大小，可以改为500。
2. help：帮助文档
3. jre64：自带了JRE1.8
4. lib：依赖的类库
5. license：相关插件的许可信息
6. plugins：插件目录

**查看设置目录结构**：

没启动过的话，没有这个目录，启动后会在C盘用户目录下多出一个.IntelliJIdeaXxx，里面包括了相关配置信息（config），系统目录（system）。

config主要包括代码提示、模板、字体风格、插件等。**重要！**

system包括缓存等信息。



---

## 激活

直接搜百度idea激活，获取激活码激活即可。



---

## 创建工程/打开工程

创建新的工程（跟eclipse创建工作空间一样），导入工程同理。

工程下的.idea和PROJECT01.iml文件都是IDEA工程特有的。类似于Eclipse工程下的.setttings、.classpath、.project等。



---

## 创建模块（Module）

在Eclipse中我们有Workspace（工作空间）和Project（工程）的概念，在IDEA中只有Project（工程）和Module（模块）的概念。这里的对应关系是：

IDEA中的Project是Eclipse中的Workspace。

IDEA中的Module是Eclipse中的Project。

**在IDEA中**，Project是最顶级的级别，次级是Module。一个Project可以有多个Module。因为目前主流的大型项目都是分布式部署的，结构都是类似这种多Module结构。

这类项目一般这样划分，如：coreModule、webModule、pluginModule、solrModule等等。模块之间彼此可以相互依赖。通过这些Module的命名可以看出来，他们之间都是处于同一个项目业务下的模块，彼此之间是有不可分割的业务关系的。

如果说项目比较小，如单体项目，是可以在Project下直接编写src代码的。



---

## 删除模块（Module）

需要先关闭该模块（右键Open Module Settings-->减号移除该模块，此时相当于关闭了项目），再进行右键删除（磁盘上的也删除）。



#### 基本设置：

1. 打开一些工具栏：

   ![添加工具栏](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea3.png)

2. File->Settings进行配置![设置界面](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea4.png)

   在这里可以设置一些主题等。相关主题可以到网上下载然后导入即可。

3. 鼠标移动，显示快速的文本提示

   ![快速的文本提示](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea5.png)

4. 设置自动导包

   ![设置自动导包](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea6.png)

5. 设置显示行号和方法间的分隔符

   ![显示行号和方法间的分隔符](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea7.png)

6. 忽略大小写的提示

   ![忽略大小写的提示](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea8.png)

   新版本直接去掉这个选项即可。

7. **设置取消单行显示tabs的操作**

   ![设置取消单行显示tabs的操作](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea9.png)

8. 设置默认的字体、字体大小、字体行间距等

   ![设置默认的字体、字体大小、字体行间距等](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea10.png)

9. 设置头部信息

   ![设置头部信息](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea11.png)

   ```
   /**
       @author xieweidu
       @createDate ${YEAR}-${MONTH}-${DAY} ${TIME}
   */
   ```

10. 设置文件编码

    ![设置文件编码](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea12.png)

11. **设置自动编译**

    ![设置自动编译](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea13.png)

    IDEA默认状态下不自动编译，Eclipse都是自动编译的。

12. 关闭IDEA自动更新

    Settings->Appearance & Behavior->System Settings->Updates里面的Automatically check updates for前面的选项取消。



## 常用模板

| 模板      | 作用                                       |
| --------- | ------------------------------------------ |
| psvm      | main方法                                   |
| sout      | System.out.println()                       |
| soutp     | 快速打印出形参                             |
| soutm     | 打印出方法名                               |
| stouv     | 打印变量                                   |
| xxx.sout  | sout一样                                   |
| fori      | fro(int i=0;i<  ;i++){<br />}              |
| iter      | for (String s : arr) {     <br />}         |
| itar      | fro(int i=0;i<**arr.length** ;i++){<br />} |
| list.for  | 增强for循环                                |
| list.fori | 普通for循环                                |
| list.forr | 普通逆序for循环                            |
| ifn       | if(xxx==null)                              |
| inn       | if(xxx!=null)                              |
| xxx.null  | if(xxx==null)                              |
| xxx.nn    | if(xxx!=null)                              |
| prsf      | private static final                       |
| psf       | public static final                        |
| psfi      | public static final int                    |
| psfs      | public static final String                 |
| thr       | throw new                                  |



---

## 修改、自定义模板、

修改：Editor-->Live Templates修改Abbreviation值即可。

自定义：右边加号添加。