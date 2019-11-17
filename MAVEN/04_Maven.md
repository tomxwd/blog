---
title: MAVEN私服
date: 2019-06-07 21:43:20
tags: 
 - Java
 - Maven
categories:
 - Java环境
 - Maven
---

# MAVEN私服

---
## 私服安装
1. 下载安装包（**nexus-2.12.0-01-bundle.zip**）。
2. 解压到本地磁盘。
3. 使用管理员权限打开dos，在dos下执行命令安装私服服务器。
  - `nexus install`命令
4. 启动服务
  - `nexus start`命令
5. 找到私服访问的url
  - 在conf文件夹中找到nexus.properties文件，看`application-port`等属性。
  - 默认：http://localhost:8081/nexus/#welcome
  - 登录：admin/admin123
---
## 私服仓库类型
|Repository|Type|
|:-:|:-:|
|***Public Repositories***|group|
|3rd pary|hosted|
|Releases|hosted|
|Snapshots|hosted|
|Apache Snapshots|proxy|
|Central|proxy|
|Central M1 shadow|virtual|
1. Hosted：宿主仓库
  - 存放本公司开发的jar包（正式版本、测试版本、第三方：存在版权问题———Oracle）
2. Proxy：代理仓库
  - 代理中央仓库、Apache下测试版本的jar包。
3. Group：组仓库
  - 包含Hosted宿主仓库和Proxy代理仓库。
  - **将来连接组仓库即可**。
---
## 上传jar包到私服上（应用）
1. 在Maven目录下conf/setting.xml<br>配置认证：配置用户名和密码。
```
宿主仓库
<servers>
  <server>
    <id>releases</id>
    <username>admin123</username>
    <password>amdin123</password>
  </server>
  <server>
    <id>snapshots</id>
    <username>admin123</username>
    <password>amdin123</password>
  </server>
</servers>
```
2. 在将要上传的项目的pom.xml中配置jar包上传路径url
```
<distributionManagement>
  <repository>
    <id>releases</id>
    <url>http://localhost:8081/nexus/content/repositories/releases/</url>
  </repository>
  <snapshotRepository>
    <id>snapshots</id>
    <url>http://localhost:8081/nexus/content/repositories/snapshots/</url>
  </snapshotRepository>
</distributionManagement>
```
3. 执行命令发布项目到私服（上传）<br>
maven项目右键RunAs->bulid的时候用`deploy`指令
4. 下载jar包到本地仓库（应用）
  - 在Maven目录下的conf/settings.xml中配置模板
  ```
  <profile>
    <!-- profile的id -->
    <id>dev</id>
    <repositories>
      <!-- 仓库id，repositories可以配置多个仓库，保证id不重复即可 -->
      <id>nexus</id>
      <!-- 仓库地址，即nexus仓库组的地址 -->
      <url>http://localhost:8081/nexus/content/groups/public/</url>
      <!-- 是否下载releases构件 -->
      <releases>
        <enabled>true</enabled>
      </releases>
      <!-- 是否下载snapshots构件 -->
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    </repositories>
    <!-- 插件仓库，maven的运行依赖插件，也需要从私服下载插件 -->
    <pluginRepository>
      <!-- 插件仓库的id不允许重复，如果重复后边配置会覆盖前边 -->
      <id>public</id>
      <name>Public Repositories</name>
      <url>http://localhost:8081/nexus/content/groups/public/<url>
    </pluginRepository>
  </profile>
  ```
  将来下载jar包可能是下载本公司开发的，也有可能是下载第三方的（中央仓库）。
  - 激活模板
  ```
  <activeProfiles>
    <activeProfile>dev</activeProfile>
  </activeProfiles
  ```
---
## 总结
1. 使用maven整合ssh框架（掌握）
2. 拆分maven工程（重点）
  - 将每一层代码和配置文件全部提取到一个独立的工程
3. 私服（了解）
