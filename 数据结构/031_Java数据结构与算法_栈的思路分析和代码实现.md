---
title: 031_Java数据结构与算法_栈的思路分析和代码实现
date: 2019-07-30 20:25:27
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 031_Java数据结构与算法_栈的思路分析和代码实现

1. ![栈的快速入门](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/031%E6%A0%88%E7%9A%84%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.png)

2. ![数组模拟栈的思路分析图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/031%E6%95%B0%E7%BB%84%E6%A8%A1%E6%8B%9F%E6%A0%88%E7%9A%84%E6%80%9D%E8%B7%AF%E5%88%86%E6%9E%90%E5%9B%BE.png)

3. 代码：

   ```java
   /**
    * 定义一个栈结构
    */
   class ArrayStack{
   
       /**
        * top指针默认指向-1的位置
        */
       private Integer top = -1;
       /**
        * 栈的大小
        */
       private Integer maxSize = 0;
       /**
        * 模拟栈
        */
       private int[] stack;
   
       /**
        * 构造器，根据maxSize创建数组
        * @param maxSize
        */
       public ArrayStack(Integer maxSize){
           this.maxSize = maxSize;
           stack = new int[maxSize];
       }
   
       /**
        * 栈满
        * @return
        */
       public boolean isFull(){
           return top==maxSize-1;
       }
   
       /**
        * 栈空
        * @return
        */
       public boolean isEmpty(){
           return top==-1;
       }
   
       /**
        * 入栈
        * @param data
        */
       public void push(Integer data){
           //判断栈是否满了
           if (isFull()) {
               System.out.println("栈满");
               return;
           }
           stack[++top]=data;
       }
   
       /**
        * 出栈
        * @return
        */
       public Integer pop(){
           // 判断是否为空
           if (isEmpty()) {
               throw new RuntimeException("栈空");
           }
           return stack[top--];
       }
   
       /**
        * 遍历栈，显示栈
        */
       public void show(){
           if (isEmpty()) {
               System.out.println("栈空");
               return;
           }
           for (Integer i = top; i >= 0; i--) {
               System.out.printf("statck[%d]=%d\n",i,stack[i]);
           }
       }
   }
   ```

   