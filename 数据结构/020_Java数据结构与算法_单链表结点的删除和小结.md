---
title: 020_Java数据结构与算法_单链表结点的删除和小结
date: 2019-07-28 17:43:43
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 020_Java数据结构与算法_单链表结点的删除和小结

![单链表结点的删除](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/020%E5%8D%95%E9%93%BE%E8%A1%A8%E7%BB%93%E7%82%B9%E7%9A%84%E5%88%A0%E9%99%A4.png)

```java
/**
     * 删除结点
     * 思路：
     * 1. head结点不能动，因此需要一个temp辅助结点找到待删除的前一个结点
     * 2. 说明在比较的时候是temp.next.no==no
     * @param no
*/
public void delete(int no){
    Node temp = head;
    boolean flag = false;
    while (true){
        if(temp.next==null){
            break;
        }
        if(temp.next.no == no){
            // 找到了待删除结点的前一个结点
            flag = true;
            break;
        }
        temp = temp.next;
    }
    if(flag){
        temp.next = temp.next.next;
    }else{
        System.out.printf("找不到编号为%d的结点\n",no);
    }
}
```

