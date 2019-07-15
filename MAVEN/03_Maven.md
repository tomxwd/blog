---
title: MAVEN3
data: 2019-06-07 ‏‎‏‎20:35:02
tags: 
 - Java
 - Maven
categories:
 - Java环境
 - Maven
---

# MAVEN3

---
## 通过maven对项目进行拆分、聚合（**重点**）
对已有的maven SHH项目进行拆分。<br>
拆分思路：
  - 将dao层的代码以及配置文件全体提取出来到一个表现上独立的工程中。
  - 同样service、action拆分

拆分完成对拆分后的项目进行聚合，提出概念--父工程。
```
SSH-parent
  SSH-dao
  SSH-service
  SSH-web
```
1. 创建父工程需要选择创建maven工程。**packaging选择pom**。
  - 创建好的父工程只有一个pom.xml文件，不负责编码。作用：
    1. 项目需要的依赖信息，在父工程中定义，子模块不需要定义，直接进行继承。
    2. 将各个模块聚合到一起。
  - **将创建好的父工程发布到本地仓库！**
2. 创建dao项目
  - 需要在父工程下右键，否则需要输入父工程的项目坐标。建立Maven-Module工程。
  - 打包方式选择jar形式。
  - 创建好的dao项目，包含dao相关的代码、配置文件。
3. 创建service工程
  - 同样需要创建Maven-Module工程，jar形式。
  - 把service相关的配置文件以及代码也迁移到service工程中。
  - 由于需要到dao项目，所以要在**pom.xml中添加dao作为依赖**，否则会报错。
4. 创建web工程
  - 同样需要创建Maven-Module工程，但是需要打成war形式。
  - 同样需要把action相关的配置文件以及代码迁移到web工程中。
  - 由于需要调用多啊service项目，所以需要在**pom.xml中添加service作为依赖**，否则会报错。为什么没有引入dao？因为service已经引入了dao，所以不需要再引入。
5. 单元测试
  - 批量加载spring配置文件
    1. classpath:spring/applicationContext-*.xml
    2. classpath*:spring/applicationContext-*.xml:既要加载本项目中的配置文件，也要加载jar包中的配置文件。
  - 传递依赖的范围<br>
![maven的传递依赖范围](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Maven/03maven%E7%9A%84%E4%BC%A0%E9%80%92%E4%BE%9D%E8%B5%96%E8%8C%83%E5%9B%B4.png)
直接依赖：A跟B的范围<br>
传递依赖：B跟C的范围<br>
中间部分：A跟C的范围<br>
例如：A：SSH-service工程  B：SSH-dao工程 C:Junit单元测试
  - 总结：<br>
    当项目中需要某一个依赖没有传递过来的时候，在自己的工程中添加对应的依赖即可。
6. 运行方式<br>
  - Maven方式：
    - 方式1：运行父工程。<br>父工程将各个子模块聚合在一起。将ssh-web打war包发布到tomcat。
    - 方式2：直接运行web工程。
  - 其他方式：
    - 部署到tomcat中。

---
