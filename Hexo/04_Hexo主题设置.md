---
title: 04_Hexo主题设置
date: 2019-11-16 20:51:05
tags: 
 - Hexo
categories:
 - Hexo
---

# 04_Hexo主题设置

## 下载主题

这里用NexT主题，[官网](http://theme-next.iissnan.com/)；

[官网NexT主题发布页面（稳定版）](https://github.com/iissnan/hexo-theme-next/releases)

下载Source code（zip），解压到/themes目录下并更名为next；



## 启用主题

把全局配置文件的theme：指定为theme: next

然后执行：

```
hexo clean
hexo s
```



## 主题配置

参考官网给的[配置](http://theme-next.iissnan.com/getting-started.html)

```
---
title: Hexo Next如何在文章摘要展示图片
date: 2018-07-18 17:43:44
tags: Hexo
categories: Hexo
---
<img src="https://faithlove.github.io/pic/2018/RMTP_1/topPicPre.png" width=50% />
哇，漂亮的小姐姐(❤ ω ❤)
<!--more-->
```



## 配置导航

tag：`hexo new page tags`

categories：`hexo new page categories`

about：`hexo new page about`

archives：`hexo new page archives`

在主题配置中也开启这几个导航选项。

接着，编辑各个index.md页面，追加type: 各个标签名，即可。