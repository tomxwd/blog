---
title: 02_Spring_IOC
date: 2019-10-20 16:51:29
tags: 
 - spring
 - aop
categories:
 - spring源码
---

# 02_Spring_IOC

1. 为什么我们要使用Spring呢？

   - 为什么用IOC？

     在没有IOC之前，使用的是内部创建的方式来管理对象，这样会导致创建出来的对象很多，每次依赖到对象都要去new，而且依赖关系复杂。

   - 如何设计IOC容器？

     对于上面的**依赖对象多次创建问题**，可以用单例，工厂模式等设计模式来解决（beanFactory）

     容器要去统一对象的构建方式，通过反射、单例、工厂模式

     对于上面的**依赖关系复杂问题**，可以用外部传入对象方式解决依赖问题（构造器、方法传参、属性、反射）	

   - SpringIOC如何做的？

     **统一负责对象的创建，管理生命周期，自动维护对象之间的依赖关系。**

   - 依赖注入：

     - 构造器传参
     - 方法传参
     - 属性反射 field.set(x)

   - 依赖查找：

     - byType
     - byName

   - spring使用：

     - bean的装配方式：
       - xml
       - 注解：
         - @Component、@ComponentScan、@Service、@Controller...
         - @Bean、@Configuration...
       - FactoryBean
         - 接口：FactoryBean
       - @Import
         - 等同于xml中的import
         - 实现接口ImportBeanDefinitionRegistry、ImportSelector

2. BeanFactory和FactoryBean有什么区别？

   - FactoryBean是一个接口，实现里面的三个方法，可以往容器里添加bean，用byName的方式来取，如"myFactoryBean"，则会拿到其getObject方法返回的bean对象(User)，而如果用"&myFactoryBean"来取，则会取到真实的factoryBean对象。
   - 

3. Spring提供了几种方式管理bean的生命周期？有什么区别？



## IOC介绍

什么是IOC？

IOC也称为依赖注入（DI），它是一个对象定义依赖关系的过程，也就是说，对象只通过构造函数参数、工厂方法的参数或对象实例构造方法或从工厂返回在对象实例上设置的属性来定义他们所使用的其他对象。然后容器在创建bean时注入这些依赖项。这个过程本质上是bean本身的逆过程，因此称为控制反转（IOC），通过直接构造来控制其依赖项的实例化。



什么是bean？

在Spring中，构成应用程序主干并由Spring IOC容器管理的对象称为bean，bean是由SpringIOC容器实例化、组装和管理的对象。