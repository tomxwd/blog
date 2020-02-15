---
title: 03_IDEA关联数据库
date: 2019-06-16 16:28:12
tags: 
 - IDEA
categories:
 - IDEA
 - IDEA基础入门
---

# 03_IDEA关联数据库

首先点击右边：

![右侧菜单栏](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea19.png)

选择数据库类型：

![选择数据库类型](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea20.png)

连接数据库（当Test Connection的时候报错缺少包，Download就好了，会自动下载驱动包，如果驱动与MySQL服务版本不对，需要更改Driver）：

![连接数据库1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea21.png)

![连接数据库2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea22.png)

点开发现没有任何表或者数据库，这时候右键->Database Tools->Manage Shown Schemas...

![没显示任何数据库内容](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea23.png)

![同步数据操作](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/idea24.png)



---

## 总结

在IDEA中关联数据库并不是为了替代其他的数据库管理工具，而是为了方便生成对象。