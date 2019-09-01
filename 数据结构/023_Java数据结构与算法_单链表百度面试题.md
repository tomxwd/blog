---
title: 023_Java数据结构与算法_单链表百度面试题
date: 2019-07-28 20:06:36
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 023_Java数据结构与算法_单链表百度面试题

从尾到头打印链表【百度，要求方式1：反向遍历。方式2：Stack栈】

方式1：若用逆序，再打印，会破坏原来链表的结构（不建议）

方式2：利用栈这个数据结构，将各个节点压入到栈中，然后利用栈的**先进后出**的特点，就实现了逆序打印的效果。



![从尾到头打印单链表](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/023%E4%BB%8E%E5%B0%BE%E5%88%B0%E5%A4%B4%E6%89%93%E5%8D%B0%E5%8D%95%E9%93%BE%E8%A1%A8.png)



栈的基本使用演示：

![栈的基本使用](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/023%E6%A0%88%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8.png)

```java
/**
 * 演示栈的基本使用
 * @author xieweidu
 * @createDate 2019-07-28 20:10
 */
public class TestStack {

    public static void main(String[] args) {
        Stack<String> stack = new Stack<String>();
        // 入栈
        stack.add("jack");
        stack.add("tom");
        stack.add("smith");
        // 出栈
        while (stack.size()>0){
            // pop就是将栈顶的数据取出
            System.out.println(stack.pop());
        }
    }

}
```



方式2实现代码：

```java
/**
     * 方式2：
     * 可以利用栈这个数据结构，将各个结点压入到栈，然后利用栈的先进后出的特点，就实现了逆序打印的效果
     * @param head
*/
public static void reversePrint(Node head){
    if(head.next==null){
        System.out.println("空链表无法打印");
    }
    // 创建一个栈，将各个节点压入栈中
    Stack<Node> stack = new Stack<Node>();
    Node cur = head.next;
    // 将链表的所有节点压入到栈中
    while (cur!=null){
        stack.push(cur);
        // cur后移，压入下一个结点
        cur = cur.next;
    }
    // 将栈中的结点进行打印，pop出栈
    while(stack.size()>0){
        System.out.println(stack.pop());
    }
}
```



合并两个有序的单链表，合并之后的链表依然要保持有序【课后练习】