---
2019-08-02 14:54:34
---















方法区

- 存储运行时常量池，已被虚拟机加载的**类信息**、常量、静态变量、即时编译器编译后的代码等数据

  类信息：

  - 类的版本
  - 字段
  - 方法
  - 接口

- 方法区和永久区

  - 很多人会把这两个东西说成同一样东西，但实际上不是，只是HotSpot是这样，HotSpot这样做是为了把垃圾回收扩展到方法区中，也就是说使用永久代来实现方法区可以省去对方法区编写内存管理代码的工作。
  - 而且在其他的虚拟机有的并不存在永久代的概念。

- 垃圾回收在方法区的行为

  - 回收效率比较低，成本高

- 异常的定义

  - OutOfMemoryError，申请不到内存的时候也会抛出OOM。