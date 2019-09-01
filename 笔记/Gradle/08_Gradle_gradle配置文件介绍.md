---
2019-08-27 14:28:50

---

#
没配置本地仓库就会放到以下路径：

![1566887701820](../数据结构/数据结构图解/1566887701820.png)

```groovy
plugins {
    id 'java'
}

/*项目信息*/
group 'top.tomxwd'
version '1.0-SNAPSHOT'

/*java版本*/
sourceCompatibility = 1.8

/*指定所使用的仓库，
这里mavenCentral()表示使用中央仓库，
此刻项目中所需要的jar包都会默认从中央仓库下载到本地指定目录*/
repositories {
    mavenCentral()
}

/*gradle工程所有的jar包的坐标都在dependencies属性内
* 每一个jar包的坐标都有三个基本元素组成
* group，name，version
* testCompile表示该jar包在测试的时候起作用，该属性为jar包的作用域
* 我们在gradle里面添加坐标的时候都要加上jar包的作用域*/
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    compile group: 'org.springframework', name: 'spring-core', version: '5.1.5.RELEASE'
}
```



