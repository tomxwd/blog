---
2019-08-27 15:02:41

---

#

把指定的maven仓库位置配置到环境变量中：

| 变量             | 值                                                   |
| ---------------- | ---------------------------------------------------- |
| GRADLE_USER_HOME | D:\environment\maven-repository【本地maven仓库位置】 |

![1566891649819](../数据结构/数据结构图解/1566891649819.png)

可以看到已经生效；

```groovy
/*指定所使用的仓库，
这里mavenCentral()表示使用中央仓库，
此刻项目中所需要的jar包都会默认从中央仓库下载到本地指定目录*/
/*加上mavenLocal()，表示先从本地仓库去找依赖，如果没有再从中央仓库下载
* 如果只配置mavenCentral()，表示直接从中央仓库下载jar包，但是如果指定下载的位置已经有了就不会再次下载了*/
repositories {
    mavenLocal()
    mavenCentral()
}
```



> [官方文档说明仓库的问题](https://docs.gradle.org/4.10-rc-1/userguide/repository_types.html#sub:maven_local)

这里需要注意，配完之后发现，貌似没有用到以前的maven仓库，根据官方文档说明，需要把USER_HOME里面的.m2文件夹中添加settings.xml配置文件即可，但是其实会发现，gradle的仓库还是跟maven的仓库

坑很多，导入的spring包找不到context.support包等；