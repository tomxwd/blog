---
title: 026_Java数据结构与算法_双线链表的功能测试及小结
date: 2019-07-28 21:16:09
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 026_Java数据结构与算法_双线链表的功能测试及小结

```java
public class DoubleLinkedListDemo {

    public static void main(String[] args) {

        System.out.println("双向链表测试");
        DoubleLinkedList list = new DoubleLinkedList();
        list.add(new Node2(1,"zhangsan","张三"));
        list.add(new Node2(2,"lisi","李四"));
        list.add(new Node2(3,"wangwu","王五"));
        list.add(new Node2(4,"zhaoliu","赵六"));
        list.show();
        // 修改
        System.out.println("双向链表修改测试");
        Node2 node2 = new Node2(4, "xinzhaoliu", "新赵六");
        list.update(node2);
        list.show();
        // 删除
        System.out.println("双向链表删除测试");
        list.delete(2);
        list.show();

    }

}
```



作业：

双向链表的第二种添加方式，按照编号顺序

![双向链表第二种添加方式示意图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/026%E5%8F%8C%E5%90%91%E9%93%BE%E8%A1%A8%E7%AC%AC%E4%BA%8C%E7%A7%8D%E6%B7%BB%E5%8A%A0%E6%96%B9%E5%BC%8F%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

