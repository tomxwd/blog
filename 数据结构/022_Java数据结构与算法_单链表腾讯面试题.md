---
title: 022_Java数据结构与算法_单链表腾讯面试题
date: 2019-07-28 18:15:46
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 022_Java数据结构与算法_单链表腾讯面试题

单链表的反转【腾讯面试题，有一定难度】



本人的做法：

```java
public static void reverse(Node head){
    if(head.next==null){
        System.out.println("为空");
        return;
    }
    Node reverseHead = new Node(0,"","");
    Node cur = head.next;
    Node temp = null;
    while (cur!=null){
        temp = cur;
        cur = cur.next;
        temp.next = reverseHead.next;
        reverseHead.next = temp;
    }
    head.next = reverseHead.next;
}
```

​	大概思路是新定义一个头结点，遍历原来的链表，然后依次插入新的头结点之后，这样在前面的结点就会被移向后面，达到逆序的效果；

1. 定义一个新的头结点reverseHead
2. 用一个cur变量指向头结点的下一个结点信息。
3. 用cur遍历整个链表
4. 将需要插入的node存储到temp，然后立马让cur指向下一个结点（重点）
5. 再将temp的下一个结点指向头结点的下一个结点：temp.next = reverseHead.next;
6. 然后将头结点的下一个结点指向temp：reverseHead.next = temp;
7. 最后用原来的头结点指向逆序头结点的下一个结点：head.next = reverseHead.next;



视频做法：

原来跟我是一样的思路。

![单链表的反转思路](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/022%E5%8D%95%E9%93%BE%E8%A1%A8%E7%9A%84%E5%8F%8D%E8%BD%AC%E6%80%9D%E8%B7%AF.png)

```java
 // 将单链表反转
public static void reverseList(Node head){
    // 如果当前链表为空，或者只有一个结点，则无需反转，直接返回
    if(head.next == null||head.next.next==null){
        return;
    }
    // 定义一个辅助指针（变量），帮助我们遍历原来的链表
    Node cur = head.next;
    //指向当前结点【cur】的下一个结点
    Node next = null;
    Node reverseHead = new Node(0,"","");
    // 遍历原来的链表并完成 从头部插入操作
    while(cur!=null){
        //暂时保存当前结点的下一个结点，因为后面需要使用
        next = cur.next;
        // 将cur的下一个结点指向新的链表的最前端
        cur.next = reverseHead.next;
        // 将cur连接到新的链表上
        reverseHead.next = cur;
        // 当前结点后移
        cur = next;
    }
    // 将head.next指向reverse.next即可
    head.next = reverseHead.next;
}
```

思路相同，只是局部实现略有不同。

