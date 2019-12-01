---
title: 01_Java设计模式_案例
date: 2019-12-01 23:23:53
tags: 
 - 设计模式
categories:
 - 设计模式
---

# 01_Java设计模式_案例

## 原型设计模式问题

1. 用UML类图画出原型模式核心角色；

2. 原型设计模式的深拷贝和浅拷贝是什么，并写出深拷贝的两种方式的源码：

   - 重写clone方法实现深拷贝
   - 使用序列化来实现深拷贝

3. 在Spring框架中哪里使用到原型模式，并对源码进行分析：

   - beans.xml中有个`<bean id="id01" class="top.tomxwd.spring.bean.Monster" scope="prototype"/>`，这里的scope="prototype"就是；

4. Spring中原型bean的创建，就是原型模式的应用

5. 代码分析+Debug源码

   代码：

   ```java
   public static void main(String[] args){
       ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
       //获取monster[通过id获取monster]
       Object bean = applicationContext.getBean("id01");
       System.out.println("bean"+bean);
   }
   ```

   源码：

   ```java
   @Overrid
   public Object getBean(String name) throws BeansException{
       return doGetBean(name,null,null,false);
   }
   ```

   ```java
   eles if(mbd.isPrototype()){
       ......
   }
   ```



## 设计模式的七大原则

要求：

1. 七大设计原则核心思想
   - 单一职责原则
   - 接口隔离原则
   - 依赖倒转原则
   - 里式替换原则
   - 开闭原则（ocp原则）
   - 迪米特法则
   - 合成复用原则
2. 能够以类图的形式说明设计原则
3. 在项目实际开发中，在哪里使用到了ocp原则【工厂设计模式】

