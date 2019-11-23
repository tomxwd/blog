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



## 文件写入（fs文件系统）

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



#### 同步操作文件

1. 打开文件
   - fs.openSync(path,[,flags,mode]
     - path：要打开文件的路径
     - flags：打开文件要做的操作类型
       - r：只读的
       - w：只写的
     - mode：设置文件的操作权限，一般不传
     - 返回值：
       - 该方法会返回一个文件的描述符作为结果，我们可以通过该描述符来对文件进行各种操作
2. 向文件中写入内容
   -  fs.writeSync(fd, string[, position[, encoding\]])
     - fd：文件的描述符，需要传递要写入文件的描述符
     - string：要写入的内容
     - position：写入的起始位
     - encoding：编码
3. 保存并关闭文件
   - fs.closeSync(fd)
     - fd：要关闭的文件的描述符

```javascript
var fs = require("fs");
console.log(fs);
// 打开文件
var fd = fs.openSync("hello.txt","w")
console.log(fd);
// 写入内容
fs.writeSync(fd,"今天的天气真不错");
// 关闭文件
fs.closeSync(fd);
```



#### 异步操作文件

**异步方法不能有返回值，在回调函数里返回fd。**

1. 打开文件
   - fs.open(path[, flags[, mode]], callback)
     - callback：回调函数，里面包含了调用结果
       - callback参数err：错误对象，如果没有错误则为null
       - callback参数fd：文件描述符
2. 写内容
   - fs.write(fd, string[, position[, encoding]], callback)
3. 保存并关闭文件
   - fs.close(fd,callback)

```javascript
var fs = require("fs");
// 打开文件
fs.open("hello.txt","w",function(err, fd){
	// 判断是否出错
	if(!err){
		// 如果没有出错，就对文件进行写入操作
		fs.write(fd,"异步写入内容",function(err){
			if(!err){
				console.log("写入成功");
			}
			// 关闭文件
			fs.close(fd,function(err){
				if(!err){
					console.log("文件已关闭");
				}
			})
		})
	} else{
		console.log(err);
	}
});
```



### 简单文件写入（常用方式）

其实是对基本的文件操作进行了封装。

- 异步：fs.writeFile(file, data[, options\], callback)
  - file：要操作的文件的路径
  - data：要写入的数据
  - options：选项，可以对写入进行一些设置
    - encoding：默认utf-8
    - mode：默认0o666
    - flag：默认w（写入）
  - callback：当写入完成以后执行的函数

- 同步：fs.writeFileSync(file, data[, options\])

```javascript
// 引入fs模块
var fs = require("fs");

fs.writeFile("hello.txt","这是通过writeFile写入的内容",{flag:"a"},function(err){
	if(!err){
		console.log("写入成功");
	}
});
```



**追加内容，而不是覆盖内容**，把flag换成a：

| 模式 | 说明                                               |
| ---- | -------------------------------------------------- |
| r    | 读取文件，文件不存在则出现异常                     |
| r+   | 读写文件，文件不存在则出现异常                     |
| rs   | 在同步模式下打开文件用于读取                       |
| rs+  | 在同步模式下打开文件用于读写                       |
| w    | 打开文件用于写操作，不存在则创建，存在则截断       |
| wx   | 打开文件用于写操作，如果存在则打开失败             |
| w+   | 打开文件用于读写，如果不存在则创建，如果存在则截断 |
| wx+  | 打开文件用于读写，如果存在则打开失败               |
| a    | 打开文件用于追加，如果不存在则创建                 |
| ax   | 打开文件用于追加，如果路径存在则失败               |
| a+   | 打开文件进行读取和追加，如果不存在则创建该文件     |
| ax+  | 打开文件进行读取和追加，如果路径存在则失败         |



### 流式文件写入

无论是同步、异步、简单文件的写入，都不适合操作大文件：

- 性能较差
- 容易导致内存溢出

流失文件写入方法：

- fs.createWriteStream(path[,options])
- 可以通过监听流的open和close事件来监听流的打开和关闭
  - 由于open只需要一次，所以用onec来绑定
    - ws.once("open",function(){})
  - 而close需要绑定之后，手动调用end()
    - ws.once("close",function(){})
    - ws.end()

```javascript
// 引入fs模块
var fs = require("fs");

// 流式文件写入
// 创建一个可写流
var ws = fs.createWriteStream("hello.txt");

ws.once("open",function(){
	console.log("打开流");
})

ws.once("close",function(){
	console.log("流关闭");
})

// 通过ws向文件中输出内容
ws.write("通过可写流写入文件的内容1\n");
ws.write("通过可写流写入文件的内容2\n");
ws.write("通过可写流写入文件的内容3\n");
ws.write("通过可写流写入文件的内容4\n");
ws.write("通过可写流写入文件的内容5\n");
ws.write("通过可写流写入文件的内容6\n");

ws.end();
```



## 文件读取（fs文件系统）

同步文件读取和异步文件读取跟写入过程差不多，不做赘述。



### 简单文件读取

- fs.readFile(path[, options\], callback)
  - path：路径
  - options：读取的选项
  - callback：回调函数
    - err：错误对象
    - data：读取到的数据，会返回一个Buffer
- fs.readFileSync(path,[,options])

```javascript
// 引入fs模块
var fs = require("fs");

fs.readFile("hello.txt",function(err,data){
	if(!err){
		console.log(data.toString());
	}
});
```



### 流式文件读取

流式文件读取也适用于一些比较大的文件，可以分多次将文件读取到内存中

- fs.createReadStream(path[,options])
- 可以用once绑定开启和关闭流事件
- 如果要读取一个可读流中的数据，必须要为可读流绑定一个data事件，data事件绑定完毕，它会自动开始读取数据
  - rs.on("data",function(data){})
    - data：读取到的数据
  - 每次都是读取65536个字节

```javascript
// 引入fs模块
var fs = require("fs");

// 创建一个可读流
var rs = fs.createReadStream("hello.txt");
// 创建一个可写流
var ws = fs.createWriteStream("hello2.txt");
// 监听流的开启和关闭
rs.once("open",function(){
	console.log("可读流打开了");
});

rs.once("close",function(){
	console.log("可读流关闭了");
	// 数据读取完毕，关闭可写流
	ws.end();
});

ws.once("open",function(){
	console.log("可写流打开了");
});

ws.once("close",function(){
	console.log("可写流关闭了");
});


rs.on("data",function(data){
	console.log(data);
	ws.write(data);
});
```

以上是一个将可读流的数据写入到可写流中去。可以发现十分麻烦。这时候可以用pipe函数：

```javascript
// 引入fs模块
var fs = require("fs");

// 创建一个可读流
var rs = fs.createReadStream("hello.txt");
// 创建一个可写流
var ws = fs.createWriteStream("hello2.txt");
// 监听流的开启和关闭
rs.once("open",function(){
	console.log("可读流打开了");
});

rs.once("close",function(){
	console.log("可读流关闭了");
});

ws.once("open",function(){
	console.log("可写流打开了");
});

ws.once("close",function(){
	console.log("可写流关闭了");
});

// 可以将可读流中的内容写入到可写流中
rs.pipe(ws);
```

不需要绑定data函数，也不需要在读取流关闭的时候去调用可写流的end()方法关闭可写流，直接就是rs.pipe(ws)；



## fs中的其他方法

- 验证路径是否存在
  - ~~fs.exists(path,callback)~~
    - 由于验证是否存在需要当场就知道，并不需要用到异步，所以是弃用的方法。
  - fs.existsSync(path)
- 获取文件信息，返回一个对象，里面包含文件信息
  - fs.stat(path,callback)
    - err
    - stat
      - size：文件大小
      - isFile()：是否是一个文件
      - isDirectory()：是否是一个目录
  - fs.statSync(path)
- 删除文件
  - fs.unlink(path,callback)
  - fs.unlinkSync(path)
- 列出文件（读取一个目录的目录结构）
  - fs.readdir(path[, options],callback)
    - err：是否成功
    - files：是一个字符串数据，每一个元素就是一个文件夹或者文件的名字
  - fs.readdirSync(path[,options])
- 截断文件（将文件修改为指定大小）
  - fs.truncate(path,len,callback)
  - fs.truncateSync(path,len)
- 建立目录
  - fs.mkdir(path[, mode],callback)
  - fs.mkdirSync(path[, mode])
- 删除目录
  - fs.rmdir(path,callback)
  - fs.rmdirSync(path)
- 重命名文件和目录
  - fs.rename(oldPath, newPath, callback)
  - fs.renameSync(oldPath, newPath)
- 监视文件更改写入
  - fs.watchFile(filename[, options], listener)
    - options：
      - persistent：默认true
      - interval：默认间隔5007，也就是5秒左右一次
    - listener：
      - curr：当前文件的状态
      - prev：修改前文件的状态
      - 这两个对象都是stat对象