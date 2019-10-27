---
title: 01_jQuery_基本使用
date: 2019-09-15 22:14:32
tags: 
 - jQuery
categories:
 - jQuery
---

# 01_jQuery_基本使用

> [jQuery官网](https://jquery.com/)

- jQuery核心函数：$，jQuery
- jQuery核心对象：执行$、jQuery的函数返回的就是jQuery对象



## 1. 绑定文档加载完成的监听

```js
$(function(){
				
})
```



```javascript
jQuery(function(){
    
})
```



## 2. 获取的三种方式

- $("xxx")：里面直接写选择器
- $("#id")：id选择器
- $("\<p>123\</p>")：直接变成一个jQuery对象



## 3. 对象访问基本行为

- size()
- length
- [index]
- get(index)
- each()
- index()



## 4. 基本选择器

- #id：id选择器
- element：元素选择器
- .class：class选择器
- *：通配符
- selector1,selector2,selectorN：并集
- selector1selector2selectorN：交集



## 5. 层次选择器

查找子元素，后代元素，兄弟元素的选择器

- ancestor descendant：后代
- parent > child：子元素
- prev + next：后面的兄弟
- prev ~ siblings：prev之后所有的siblings元素



## 6. 过滤选择器

在原有的选择器匹配的元素中进一步进行过滤的选择器

基本都是以冒号（：）开头的



## 7. 表单选择器

- :input
- :text
- :password
- :radio
- :checkbox
- :submit
- :image
- :reset
- :button
- :file
- :hidden

表单对象属性：

- :enabled
- :disabled
- :checked
- :selected



## 8. 工具方法

- $.each(obj,function(key,value){}):遍历数组或对象中的数据
- $.trim():去除字符串两边的空格
- $.type(obj):得到数据的类型
- $.isArray(obj):判断是否为数组
- $.isFunction(obj):判断是否为函数
- $.parseJSON(json):解析json字符串转为js对象/数组
  - JSON.parse(jsonString):     json字符串---->js对象/数组
  - JSON.stringify(jsObj/jsArr):     js对象/数组---->json字符串
  - 以上两个后来js出的，基本用这两个



## 9. 属性

操作标签的属性

- attr(name)/attr(name,value): 读写非布尔值的标签属性
- prop(name)/prop(name,value): 读写布尔值的标签属性
- removeAttr(name)/removeProp(name): 删除属性
- addClass(classValue): 添加指定Class
- removeClass(classValue): 移除指定Class
- val()/val(value): 读写合一的value
- html()/html(htmlString): 读写合一的html



## 10. CSS

- css
- 位置
- 尺寸