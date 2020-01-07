---
title: 05_Nginx_location指令说明
date: 2020-01-07 22:14:12
tags: 
 - Nginx
categories:
 - Nginx
---

# 05_Nginx_location指令说明

## location指令说明

该指令用于匹配URL

**语法：**

```shell
location [ = | ~ | ~* | ^~] uri {

}
```

1. =

   用于不含正则表达式的uri前，要求请求字符串与uri严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求；

2. ~

   用于表示uri包含正则表达式，并且区分大小写

3. ~*

   用于表示uri包含正则表达式，并且不区分大小写

4. ^~

   用于不含正则表达式的uri前，要求Nginx服务器找到标识uri和请求字符串匹配度最高的location之后，立即使用此location处理请求，而不再使用location块中的正则uri和请求字符串做匹配；

**注意：**如果uri包含正则表达式，则必须要有~或者~*标识；







