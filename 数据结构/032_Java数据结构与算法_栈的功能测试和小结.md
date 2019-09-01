---
title: 032_Java数据结构与算法_栈的功能测试和小结
date: 2019-07-30 20:47:15
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 032_Java数据结构与算法_栈的功能测试和小结

```java
public class ArrayStackDemo {

    public static void main(String[] args) {
        // 测试ArrayStack是否正确
        ArrayStack stack = new ArrayStack(4);
        String key = "";
        // 是否退出菜单
        Scanner input = new Scanner(System.in);
        while (true){
            System.out.println("1: show显示栈");
            System.out.println("2: push入栈");
            System.out.println("3: pop出栈");
            System.out.println("4: exit退出");
            System.out.println("请输入选项：");
            int i = input.nextInt();
            if(i==1){
                stack.show();
            }else if(i==2){
                System.out.println("请输入数据：");
                int data = input.nextInt();
                stack.push(data);
            }else if(i==3){
                try{
                    stack.pop();
                }catch (RuntimeException e){
                    System.out.println(e.getMessage());
                }
            }else{
                input.close();
                break;
            }
        }
    }

}
```



课后作业：

链表实现栈功能！