---
title: 07_IDEA中生成javadoc、清理缓存和索引、插件的使用
data: 2019-06-16 18:19:53
tags: 
 - IDEA
 - Java
categories:
 - 工具
 - IDEA使用
---

# 07_IDEA中生成javadoc、清理缓存和索引、插件的使用



## 生成javadoc

![javadoc第一步](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea34.png)

![javadoc第一步](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea35.png)



---

## 缓存和索引的清理

**IDEA首次加载项目的时候，都会创建索引**，而创建索引的时间跟项目的文件多少成正比。在IDEA创建索引的过程中即使你编辑了代码也是编译不了、运行不起来的，所以还是要安安静静等待IDEA索引创建完成。

**IDEA的缓存和索引主要是用于加快文件查询，从而加快各种查找、代码提示等操作的速度**，所以IDEA的索引的重要性可想而知。

但是，IDEA的索引和缓存并不是一直很良好的支持IDEA的，某些特殊条件下，IDEA的缓存和索引也是会损坏的，比如断电、蓝屏引起的强制关机，当你重新打开IDEA的时候，很可能会报很多莫名其妙的错误，甚至打不开项目，IDEA主题还原为默认状态。即便是没有发生断电、蓝屏等问题，也会有时候遇到奇怪的问题，也有可能是IDEA的索引或者缓存出现了问题，这种情况还挺常见，不过遇到了也不必担心，我们可以手动对IDEA缓存和索引进行清理。**如下：**

![清理](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea36.png)

会提醒本地的历史记录将会被清理，因为除了Git之外，IDEA还会保存历史的修改记录，如果要保存这个信息，需要进行备份（.IntelliJIdeaXxx/system/LocalHistory）。

再次打开IDEA就会重新进行创建索引操作。

第二种方式是直接删除.IntelliJIdeaXxx里的system文件，再次打开也会重新建立索引等。



---

## 插件使用：

[官网插件库地址]("https://plugins.jetbrains.com")

例如：GsonFormat

或者在Settings->Plugins中下载