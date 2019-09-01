---
title: 019_Java数据结构与算法_单链表节点的修改
date: 2019-07-26 17:48:06
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 019_Java数据结构与算法_单链表节点的修改

新的版本：

```java
/**
 * @author xieweidu
 * @createDate 2019-07-28 14:16
 */
public class SingleLinkedListDemo {

    public static void main(String[] args) {
        SingleLinkedList list = new SingleLinkedList();
       /* list.add(new Node(1,"li","LI"));
        list.add(new Node(2,"si","SI"));
        list.add(new Node(6,"zhang","ZHANG"));
        list.add(new Node(4,"san","SAN"));*/
        list.addByOrder(new Node(1,"li","LI"));
        list.addByOrder(new Node(4,"lii","LII"));
        list.addByOrder(new Node(2,"si","SI"));
        list.addByOrder(new Node(6,"zhang","ZHANG"));
        list.addByOrder(new Node(4,"san","SAN"));

        list.show();
        
        list.update(new Node(4,"san","SAN"));

        list.show();
    }


}

class SingleLinkedList{
    /**
     * 头结点
     */
    Node head = new Node (0,"","");


    /**
     * 修改节点信息，根据no编号来修改，即no编号不能改
     * @param node
     */
    public void update(Node node){
        if(head.next==null){
            System.out.println("链表为空");
            return;
        }else{
            // 找到需要修改的节点，根据编号查找
            Node temp = head.next;
            boolean flag = false; //表示是否找到该节点
            while(true){
                if(temp ==null){
                    break;// 到了最后
                }else if(temp.no==node.no){
                    // 找到
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
     * 加节点，按no顺序加
     * @param node
     */
    public void addByOrder(Node node){
        Node temp = head;
        boolean flag = false;
        while (true){
            if (temp.next==null) {
                break;
            }else{
                if(temp.next.no > node.no){
                    break;
                } else if (temp.next.no == node.no){
                    System.out.println("序号已存在");
                    flag = true;
                }
            }
            temp = temp.next;
        }
        if(!flag) {
            node.next = temp.next;
            temp.next = node;
        }
    }

    /**
     * 加节点(在后面加）
     * @param node
     */
    public void add(Node node){
        Node temp = head;
        while (true){
            if(temp.next == null){
                break;
            }else{
                temp = temp.next;
            }
        }
        temp.next = node;
    }

    public void show(){
       Node temp = head.next;
       while(temp!=null){
           System.out.println(temp);
           temp = temp.next;
       }
    }


}

class Node{
    int no;
    String name;
    String lastName;
    Node next;

    public Node(int no, String name, String lastName) {
        this.no = no;
        this.name = name;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return "Node{" +
                "no=" + no +
                ", name=" + name +
                ", lastName=" + lastName +
                '}';
    }
}
```

