---
2019-07-29 09:45:16
---



导包：

```xml
<dependencies>

    <dependency>
        <groupId>commons-beanutils</groupId>
        <artifactId>commons-beanutils</artifactId>
        <version>1.9.3</version>
    </dependency>

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

</dependencies>
```

1. Apache的BeanUtils包
2. 测试
3. lombok快速生成类信息



实体类信息：

1. Student：

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

   

2. Dog：

   ```java
   @Data
   @AllArgsConstructor
   @RequiredArgsConstructor
   public class Dog {
   
       private String name;
   
   }
   ```

   

3. ```java
   public class BeanUtilTest {
   
       @Test
       public void test1() throws Exception {
           Student stu1 = new Student(1,"tom",null);
           Student stu2 = new Student(2,"tim",null);
           Student stu3 = new Student();
           System.out.println("stu1 = " + stu1);
   
           // 拷贝属性
           BeanUtils.copyProperties(stu3,stu1);
           System.out.println("stu3 = " + stu3);
   
           System.out.println("----------------------------");
           // 设置属性的值
           BeanUtils.copyProperty(stu3,"no",3);
           BeanUtils.copyProperty(stu3,"name","Mary");
           System.out.println("stu3 = " + stu3);
   
           System.out.println("----------------------------");
           // JavaBean转为Map对象
           Map<String, String> map = BeanUtils.describe(stu1);
           System.out.println("map = " + map);
           for(Map.Entry<String,String> entry:map.entrySet()){
               System.out.println("K: "+entry.getKey()+"---V: "+entry.getValue());
           }
   
           System.out.println("----------------------------");
           // Map转JavaBean
           Map<String, Object> beanMap = new HashMap<String, Object>();
           beanMap.put("no","4");
           beanMap.put("name","Bob");
           beanMap.put("dog",new Dog("DOG"));
           Student stu4 = new Student();
           BeanUtils.populate(stu4,beanMap);
           System.out.println("stu4 = " + stu4);
           System.out.println(stu4.getDog().getName());
   
       }
   
   
   }
   ```



Dog类的存在只是为了测试Map转JavaBean。