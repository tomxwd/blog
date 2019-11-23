---
title: 03_Node.js_基础
date: 2019-11-23 12:35:48
tags: 
 - node.js
categories:
 - node.js
---

# 03_Node.js_基础

[官方文档](http://nodejs.cn/)

## Buffer（缓冲区）

- 从结构上看Buffer非常像一个数组，只不过它的元素为16进制的两位数（十六进制显示，实际上是二进制存储）。
  - 数组中不能存储二进制的文件，而Buffer就是专门用来存储二进制数据
- 实际上一个元素就标识内存中的一个字节。
  - 范围从00-ff
  - 一个字节(byte)8位(bit)
- 实际上Buffer中的内存不是通过JavaScript分配的，而是底层通过C++申请的。
- 也就是我们直接通过Buffer来创建内存中的空间。
- 使用Buffer不需要引入模块，直接使用即可。



字符串保存到Buffer中：

```javascript
var str = "HelloWorld";
// 将一个字符串保存到Buffer中
var buf = Buffer.form(str);
console.log(buf)
console.log(buf.length);//占用内存的大小
console.log(str.length);//字符串的长度
// 注意如果是中文，则占三个字节，所以字符串长度与占用内存长度不同
```



创建指定大小的Buffer【Buffer.alloc()】：

```javascript
var buf = new Buffer(10);//10个字节的buffer
console.log(buf.length);
```

**Buffer所有的构造函数都是不推荐使用的，都是废弃方法**

因此用推荐的方法：

```javascript
var buf = Buffer.alloc(10);
// 通过索引，操作buffer中的元素
buf[0] = 88;
buf[1] = 255;
buf[2] = 0xaa;
console.log(buf);
buf[3] = 556; // 输出2c，是因为超过了255，556转为二进制超过了8位，所以只截取了后面8位。如果是256，则为0
// 输出到控制台或页面，会转换为10进制显示
console.log(buf[2]) // 170
console.log(buf[2].toString(16));//转换为16进制的字符串，传入2则为2进制
```

**注意：**Buffer的大小一旦确定，无法修改，因为实际上是对底层内存的直接操作，因此分配了就确定了大小且空间连续；



创建不安全指定大小的Buffer【Buffer.allocUnsafe()】:

```javascript
var buf = Buffer.allocUnsafe(10);
console.log(buf);//打印了很多有值的Buffer
```

也就是allocUnsafe在分配空间的时候并不对空间进行清除，所以可能含有敏感数据。

性能好，不安全；



## 同步文件写入（fs文件系统）

- 在Node中，与文件系统的交互是非常重要的，服务器的本质就是将本地的文件发送给远程的客户端。
- Node通过fs模块来和文件系统进行交互
- 该模块提供了一些标准文件访问API来打开、读取、写入文件，以及与其交互。
- 要使用fs模块，首先需要对其进行加载
  - const fs = require("fs");



### 同步和异步调用

- fs模块中所有的操作都有两种形式可供选择
  - 同步（方法名带Sync的）
    - 同步文件系统会**阻塞**程序的执行，也就是除非操作完毕，否则不会向下执行代码。
  - 异步
    - 异步文件系统**不会阻塞**程序的执行，而是在操作完成的时候，通过回调函数将结果返回。



1. 打开文件
   - fs.openSync(path,[,flags,mode]
     - path：要打开文件的路径
2. 向文件中写入内容
3. 保存并关闭文件

