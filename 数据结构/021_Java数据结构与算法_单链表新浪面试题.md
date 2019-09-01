---
title: 021_Java数据结构与算法_单链表新浪面试题
date: 2019-07-28 17:49:20
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 021_Java数据结构与算法_单链表新浪面试题

## 单链表面试题（新浪、百度、腾讯）

单链表的常见面试题有如下：

1. 求单链表中有效结点的个数；
2. 查找单链表中的倒数第k个结点【新浪面试题】；
3. 单链表的反转【腾讯面试题】；
4. 从尾到头打印单链表【百度，要求方式1：反向遍历。方式2：Stack栈】；
5. 合并两个有序单链表，合并之后的链表依然有序【课后练习】；

求单链表中的有效结点的个数：

```java
/**
     * 获取到单链表结点的个数（如果是带头结点的链表，不需要统计头结点）
     * @param head 链表的头结点
     * @return 返回的就是有效结点的个数
*/
public static int getLinkedListLength(Node head){
    int length = 0;
    if(head.next == null){
        return length;
    }
    // 定义一个辅助变量
    Node temp = head.next;
    while (temp!=null){
        length++;
        temp = temp.next;
    }
    return length;
}
```

查找单链表中的倒数第k个结点【新浪面试题】：

```java
/**
     * 查找单链表中的倒数第k个结点【新浪面试题】：
     * 思路：
     * 1. 编写一个方法，接收head结点以及index
     * 2. index表示倒数第index个结点
     * 3. 先把链表从头到尾遍历一次，得到这个链表的总长度getLinkedListLength
     * 4. 得到size之后，我们从链表的第一个开始遍历（size-index）个，就可以得到
     * 5. 如果找到，则返回该结点，否则返回null
*/
public static Node findLastIndexNode(Node head,int index){
    // 判断为空则返回null
    if(head.next==null){
        //没有找到
        return null;
    }
    // 得到长度
    int size = getLinkedListLength(head);
    // 遍历到size-index位置，就得到倒数第index个
    // 先做一个index的校验
    if(index<=0 || index>size){
        return null;
    }
    Node cur = head.next;
    // 比如要找倒数第一个 总共3个 那就要从head.next再往下找3-1次
    for (int i = 0; i < size - index; i++) {
        cur = cur.next;
    }
    return cur;
}
```

