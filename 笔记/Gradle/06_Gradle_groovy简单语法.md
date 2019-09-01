---
2019-08-27 13:51:30

---

#





![1566885108576](../数据结构/数据结构图解/1566885108576.png)

编写：

```groovy
// 介绍groovy编程语言
println("hello groovy");
// groovy可以省略语句最末尾的分号
println("hello groovy")
// groovy可以省略括号
println"hello groovy"

// groovy中如何定义变量
// de是弱类型的，groovy会自动根据情况来给变量赋予对应的类型
def i = 18
println i

def s="xiaoming"
println s

// 定义一个集合类型
def list = ['a','b']
// 往list中添加元素
list<<'c'
// 取出list第三个元素
println list.get(2)

// 定义一个map
def map=['key1':'value1','key2':'value2']
// 向map中添加键值对
map.key3='value3'
// 打印出key3的值
println map.key3
println map.get('key3')
```

