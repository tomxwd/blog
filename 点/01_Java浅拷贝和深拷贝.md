---
title: 01_Java浅拷贝和深拷贝
date: 2019-07-29 13:50:43
tags: 
 - Java
 - 点
categories:
 - 点
---

# 01_Java浅拷贝和深拷贝

## 准备工作：

1. 引入依赖:

   ```xml
   <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.12</version>
   </dependency>
   
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <version>1.18.8</version>
   </dependency>
   ```

2. 实体类

   Student:

   ```java
   @Data
   @RequiredArgsConstructor
   @AllArgsConstructor
   public class Student {
   
       private Integer no;
       private String name;
       private Dog dog;
       
   }
   ```

   Dog:

   ```java
   @Data
   @AllArgsConstructor
   @RequiredArgsConstructor
   public class Dog {
   
       private String name;
   
   }
   ```



## 直接赋值

测试代码：

```java
@Test
public void testAssign(){
    // 直接赋值
    Student stu1 = new Student(0,"Tom",new Dog("dog"));
    Student stu2 = stu1;
    stu2.setNo(2);
    System.out.println(System.identityHashCode(stu1));
    System.out.println(System.identityHashCode(stu2));
}
```

结果：

```
326549596
326549596
```

可以看到，直接赋值是将引用传给了stu2，所以stu1和stu2的hashCode是相同的，也就是说指向同一对象。

因此无论是stu1做改变还是stu2做改变，该对象都会被影响。



## 浅拷贝

浅拷贝是按位拷贝对象，会创建新的对象出来，这个新创建的对象有被复制对象所有属性值的一份**精确拷贝**。

- 如果是基本数据类型，就会拷贝一份基本数据类型的值。
- 如果是引用数据类型，拷贝的就是地址（**复制的是引用，而不是复制引用的对象**），所以其中一个对象改变了这个地址的对象内容，会影响到另一个对象。

**要实现对象拷贝，就要实现Cloneable接口并重写clone方法。**

改造Student类：

```java
@Data
@RequiredArgsConstructor
@AllArgsConstructor
public class Student implements Cloneable{

    private Integer no;
    private String name;
    private Dog dog;

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

改造Dog类：

```java
@Data
@AllArgsConstructor
@RequiredArgsConstructor
public class Dog implements Cloneable{

    private String name;

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

测试代码：

```java
@Test
public void testShallowClone() throws CloneNotSupportedException {
    // 浅拷贝
    Student stu1 = new Student(0,"Tom",new Dog("dog"));
    Student stu2 = (Student) stu1.clone();
    // 测试基本数据类型
    System.out.println("------基本数据类型------");
    System.out.println(System.identityHashCode(stu1.getName()));
    System.out.println(System.identityHashCode(stu2.getName()));
    System.out.println("改值");
    stu1.setName("TTT");
    System.out.println(System.identityHashCode(stu1.getName()));
    System.out.println(System.identityHashCode(stu2.getName()));
    System.out.println(stu1);
    System.out.println(stu2);
    // 测试引用数据类型
    System.out.println("-------引用数据类型------");
    System.out.println(System.identityHashCode(stu1.getDog()));
    System.out.println(System.identityHashCode(stu2.getDog()));
    System.out.println("-----------1------------");
    stu2.getDog().setName("newName");
    System.out.println(stu1);
    System.out.println(stu2);
    System.out.println("-----------2------------");
    stu2.setDog(new Dog("newDog"));
    System.out.println(System.identityHashCode(stu1.getDog()));
    System.out.println(System.identityHashCode(stu2.getDog()));
}
```

输出结果：

```
------基本数据类型------
326549596
326549596
改值
1364335809
326549596
Student(no=0, name=TTT, dog=Dog(name=dog))
Student(no=0, name=Tom, dog=Dog(name=dog))
-------引用数据类型------
458209687
458209687
-----------1------------
Student(no=0, name=TTT, dog=Dog(name=newName))
Student(no=0, name=Tom, dog=Dog(name=newName))
-----------2------------
458209687
233530418

```

可以看到，基本数据类型是拷贝了一份，而引用数据类型则是赋值引用，当修改引用数据类型对象的内容时，会互相影响，除非重新new一个对象。



## 深拷贝

**如果要把引用数据类型指向的对象也进行拷贝，就要用到深拷贝。**

深拷贝的实现同样是需要都实现Cloneable接口，并重写clone()方法，不同的是clone()的内容；

Dog类保持不变，Student类改变：

```java
@Data
@RequiredArgsConstructor
@AllArgsConstructor
public class Student implements Cloneable{

    private Integer no;
    private String name;
    private Dog dog;

    @Override
    public Object clone() throws CloneNotSupportedException {
        Student newStudent = (Student) super.clone();
        Dog newDog = (Dog) this.dog.clone();
        newStudent.setDog(newDog);
        return newStudent;
    }
}

```

测试代码：

```java
@Test
public void TestDeepClone() throws CloneNotSupportedException{
    // 深拷贝
    Student student = new Student(1, "TOM", new Dog("DOG"));
    Student clone = (Student) student.clone();
    System.out.println(System.identityHashCode(student.getDog()));
    System.out.println(System.identityHashCode(clone.getDog()));
}

```



## 其他：





### 可以用序列化来做拷贝

首先对象需要实现Serializable接口。

Student：

```java
@Data
@RequiredArgsConstructor
@AllArgsConstructor
public class Student implements Serializable {


    private Integer no;
    private String name;
    private Dog dog;

}

```

Dog：

```java
@Data
@AllArgsConstructor
@RequiredArgsConstructor
public class Dog implements Serializable {

    private String name;

}

```

测试代码：

```java
@Test
public void testSerializationCopy() throws Exception {
    Student student = new Student(0, "TIM", new Dog("DOG"));
    // 写入对象
    ByteArrayOutputStream bo = new ByteArrayOutputStream();
    ObjectOutputStream oo = new ObjectOutputStream(bo);
    oo.writeObject(student);
    // 读取对象
    ByteArrayInputStream bi = new ByteArrayInputStream(bo.toByteArray());
    ObjectInputStream oi = new ObjectInputStream(bi);
    Student clone = (Student) oi.readObject();
    System.out.println(System.identityHashCode(student));
    System.out.println(System.identityHashCode(clone));
    System.out.println("----改变前------");
    System.out.println("student = " + student);
    System.out.println("clone = " + clone);
    System.out.println("----改变后------");
    student.getDog().setName("TTTT");
    System.out.println("student = " + student);
    System.out.println("clone = " + clone);
}

```

注意：流操作最好管理好资源，上面没有做。

结果：

```
254413710
1724731843
----改变前------
student = Student(no=0, name=TIM, dog=Dog(name=DOG))
clone = Student(no=0, name=TIM, dog=Dog(name=DOG))
----改变后------
student = Student(no=0, name=TIM, dog=Dog(name=TTTT))
clone = Student(no=0, name=TIM, dog=Dog(name=DOG))

```

**最后：**对于不需要的属性，可以用transient关键字进行屏蔽，这样就不会对该属性进行序列化。



### 使用

这里用到了很多System.identityHashCode(Object x)，是用来得到对象地址的。

跟Object.hashcode()有区别，API上翻译过来是：

```
返回给定对象的哈希码，该代码与默认的方法 hashCode() 返回的代码一样，无论给定对象的类是否重写 hashCode()。null 引用的哈希码为 0。 

```

而如果重写了Object的hashCode()方法，用hashCode来得到的值是重写之后的，identityHashCode则是在没有重写的时候hashCode的值。

**可以说System.identityHashCode(Object x)打印的是“原生”的hashCode。**

