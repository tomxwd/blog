---
title: 01_Node.js_简介
date: 2019-11-22 22:17:15
tags: 
 - node.js
categories:
 - node.js
---

# 01_Node.js_简介

- Node.js是一个能够在服务器端运行JavaScript的开放源代码、跨平台JavaScript运行环境。
- Node采用Google开发的V8引擎运行js代码，使用**事件驱动、非阻塞、异步I/O模型**等技术来提高性能，可优化应用程序的传输量和规模
- Node大部分基本模块都用JavaScript编写，在Node出现之前，JS通常作为客户端程序设计语言使用，以JS写出的程序常在用户的浏览器上运行。



## 模块化简介

CommonJS规范

- 一个js文件就是一个模块；

- 在node中，通过require()函数来引入外部的模块，require()中可以传递一个文件的路径来引入外部模块，如果是相对路径，需要用.或..开头；
- 使用require()引入模块以后，该函数会返回一个对象，这个对象代表的是引入的模块。
- 在node中每个js的代码都是独立运行在一个函数中，因此是不可以直接访问到里面的变量，但是可以通过exports来向外部暴露变量和方法。
  - exports.x = "hello"；
  - exports.func = function(){};



### 模块标识

使用require()引入外部模块的时候，使用的就是模块标识，可以通过模块标识找到指定的模块

模块分为两大类：

- 核心模块
  - 由node引擎提供的模块
  - 核心模块的标识就是模块的名字
    - var fs = requird("fs");
- 文件模块
  - 由用户自己创建的模块 
  - 文件模块的标识就是文件的路径（绝对路径，相对路径）
    - 一般用相对路径，用.或..开头



- global
  - 在node中有一个全局对象——global，作用和网页中的window类似，在全局中创建的函数都会作为global的方法保存
- arguments
  - 每个模块其实都是运行在一个function中，所以会有arguments属性
  - arguments.callee
    - 这个属性保存的是当前执行的函数对象
    - 那么直接`console.log(arguments.callee+"")`即可看到这个函数。

从上面的打印结果可知，当node在执行模块中的代码的时候，会首先在代码的最顶部添加如下代码：

```javascript
function (exports, require, module, _filename, _dirname){
```

最尾部添加一个大括号}；因此就知道模块是怎么相互独立，变量是隔离的。

在node执行的时候，其实就是执行这段函数，而传入的参数就是上面五个：

- exports
  - 该对象用来将变量或函数暴露到外部
- require
  - 函数，用来引入外部的模块
- module
  - module代表的是当前模块本身
  - exports就是module的属性`exports == module.exports`为true
- _filename
  - 当前模块的完整路径
- _dirname
  - 当前模块所在文件夹的完整路径，也就是filename的上一级



**module.export和export的区别：**

module.exports=一个json串：

```javascript
module.exports={
    name: "猪八戒",
    age: 28,
    sayName: function(){
        console.log("我是猪八戒");
    }
}
```

可以从外部拿到所有变量以及函数。

用exports=一个json串：

```javascript
exports={
    name: "猪八戒",
    age: 28,
    sayName: function(){
        console.log("我是猪八戒");
    }
}
```

会报错。

原因 ：就是一个引用数据类型导致的：

```javascript
// 声明一个a
var a = {}
// 给a塞一个b
a.b = {}
// 声明c指向a里的b
var c = a.b;
// 修改c的值
c = "123";
// 这时候a.b也变成123
// 如果c = new Object()
// 那么c指向了一个新的引用，因此对c进行的修改都不会对a.b产生影响
```

所以通过module.exports可以通过.的形式暴露，也可以通过直接赋值的形式进行暴露，而exports只能通过.来进行暴露。