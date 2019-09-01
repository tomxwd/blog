---
title: 025_Java数据结构与算法_双向链表增删改查代码实现
date: 2019-07-28 21:00:11
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 025_Java数据结构与算法_双向链表增删改查代码实现

```java
/**
 * 创建双向链表的类
 */
class DoubleLinkedList{

    /**
     * 先初始化一个节点头，头结点不要动，不存放具体的数据
     */
    Node2 head = new Node2(0,"","");

    /**
     * 从双向链表中删除一个结点
     * 说明
     * 1. 对于双向链表，我们可以直接找到要删除的结点
     * 2. 找到后自我删除即可
     * @param no
     */
    public void delete(int no){
        // 判断当前链表是否为空
        if(head.next==null){
            System.out.println("空");
            return;
        }
        Node2 temp = head.next;
        boolean flag = false;
        while (true){
            if(temp==null){
                break;
            }
            if(temp.no == no){
                // 找到了待删除结点的前一个结点
                flag = true;
                break;
            }
            temp = temp.next;
        }
        if(flag){
            temp.pre.next = temp.next;
            // 删除最后一个结点有问题，temp.next为null，空指针异常
            if(temp.next!=null) {
                temp.next.pre = temp.pre;
            }
        }else{
            System.out.printf("找不到编号为%d的结点\n",no);
        }
    }

    /**
     * 修改节点信息，根据no编号来修改，即no编号不能改
     * @param node
     */
    public void update(Node2 node){
        if(head.next==null){
            System.out.println("链表为空");
            return;
        }else{
            // 找到需要修改的节点，根据编号查找
            Node2 temp = head.next;
            boolean flag = false; //表示是否找到该节点
            while(true){
                if(temp ==null){
                    break;// 到了最后
                }else if(temp.no==node.no){
                    flag = true;
                    break;
                }
                temp = temp.next;
            }
            if(flag){
                temp.name = node.name;
                temp.lastName = node.lastName;
            }else{
                System.out.printf("没有找到编号为%d的结点\n",node.no);
            }
        }
    }

    /**
     * 加节点(在后面加）
     * @param node
     */
    public void add(Node2 node){
        Node2 temp = head;
        while (true){
            if(temp.next == null){
                break;
            }else{
                temp = temp.next;
            }
        }
        // 形成双向链表
        temp.next = node;
        node.pre = temp;
    }

    public void show(){
        Node2 temp = head.next;
        while(temp!=null){
            System.out.println(temp);
            temp = temp.next;
        }
    }

}



class Node2 {
    int no;
    String name;
    String lastName;
    Node2 next;
    Node2 pre;

    public Node2(int no, String name, String lastName) {
        this.no = no;
        this.name = name;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return "Node2{" +
                "no=" + no +
                ", name=" + name +
                ", lastName=" + lastName +
                '}';
    }
}
```

