---
2019-07-26 17:25:02

---





1. ```java
   public static void main(String[] args) {
       SingleLinkedList linkedList = new SingleLinkedList();
       //        linkedList.add(new HeroNode(1,"宋江","及时雨"));
       //        linkedList.add(new HeroNode(2,"卢俊义","玉麒麟"));
       //        linkedList.add(new HeroNode(3,"吴用","智多星"));
       //        linkedList.add(new HeroNode(4,"林冲","豹子头"));
       linkedList.addByOrder(new HeroNode(1,"宋江","及时雨"));
       linkedList.addByOrder(new HeroNode(4,"林冲","豹子头"));
       linkedList.addByOrder(new HeroNode(4,"林冲","豹子头"));
       linkedList.addByOrder(new HeroNode(2,"卢俊义","玉麒麟"));
       linkedList.addByOrder(new HeroNode(3,"吴用","智多星"));
       linkedList.list();
   }
   ```

2. ```java
   //  第二种添加方式，需要排序，同样排名，则添加失败
   public void addByOrder(HeroNode node){
       // 因为头节点不能动，因此仍然用一个辅助指针
       // 因为单链表，所以我们找的temp是位于添加位置的前一个节点，否则插入不了
       HeroNode temp = this.head;
       // 添加的编号是否存在，默认false
       boolean flag = false;
       while(true){
           // 说明temp在链表的最后
           if(temp.next == null){
               // 不管找不找到都要break，说明要添加在最后
               break;
           }
           //位置找到，就在temp后面插入新节点
           if(temp.next.no > node.no){
               // 不能添加，相同序号
               break;
           } else if(temp.next.no==node.no){
               flag = true;
               break;
           }
           //后移
           temp = temp.next;
       }
       // 判断flag的值
       if(flag){
           // 已存在序号
           System.out.printf("编号%d已存在，不能添加\n",node.no);
       } else{
           // 插入到temp后
           node.next = temp.next;
           temp.next = node;
       }
   }
   ```

3. 增加了addByOrder方法

