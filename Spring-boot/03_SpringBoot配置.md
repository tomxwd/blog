---
title: 03_SpringBoot配置
date: 2019-06-17 22:52:35
tags: 
 - Java
 - SpringBoot
categories:
 - Spring
 - SpringBoot初学
---

# 03_SpringBoot配置

SpringBoot使用一个全局的配置文件，名字是固定的（application)

- application.properties
- application.yml

配置文件的作用：修改SpringBoot自动配置的默认值；

SpringBoot在底层都给我们自动配置好了。

**配置文件都放在src/main/resources目录或者类路径/config下**



## YAML(YAML Ain't Markup Lanuage)

#### 1、YAML简介

- YAML A Markup Lanuage : YAML是一个标记语言
- YAML isn't Markup Lanuage：YAML不是一个标记语言

标记语言：

以前的配置文件，大多都使用xxx.xml文件；

yml是YAML语言的文件，是**以数据为中心**，比json、xml等更适合做配置文件

[参考语法规范](http://www.yaml.org/)

**配置例子：**

YAML：

```yaml
server:
  port: 8081
```

XML：

```xml
<server>
	<port>8081</port>
</server>
```

#### 2、YAML语法

###### 1.基本语法

k:(空格)v：表示一对键值对（空格必须有）

以空格的缩进来控制层级关系；只要是左对齐的数据都是同一层级的

```yaml
server:
	port: 8081
	path: /hello
```

属性和值也是大小写敏感的。



###### 2.值的写法

**字面量：普通的值（数字，字符串，布尔）**

k:v :字面量直接写，字符串默认不用加上单双引号；

- "" : 双引号，不会转义字符串里面的特殊字符；特殊字符会作为本身想表达的意思
  
- name: "zhangsan\n lisi" 输出：zhangsan换行lisi
  
- '' : 单引号，会转义特殊字符，特殊字符最终只是一个普通的字符串数据

  - name: "zhangsan\n lisi" 输出：zhangsan\n lisi

  

**对象、Map（属性和值）（键值对）；**

k: v :对象还是k: v的方式，在下一行来写对象的属性和值的关系；注意缩进

```yaml
friends:
	lastName: zhangsan
	age: 20
```

行内写法：

```yaml
friends: {lastName: zhangsan,age: 18}
```



**数组（List、Set）；**

用- 值来表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法：

```yaml
pets: [cat,dog,pig]
```



## 配置文件值注入（@ConfigurationProperties方式）

**Person类：**

```java
package com.github.tomxwd.com.github.tomxwd.bean;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.List;
import java.util.Map;

/**
 * @author xieweidu
 * @createDate 2019-07-02 23:02
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties:告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定
 *  prefix = "person" 配置文件中哪个下面的所有属性进行一一映射
 *
 *  只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能
 *
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;

    private List<Object> lists;

    private Dog dog;

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getBoss() {
        return boss;
    }

    public void setBoss(Boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getLists() {
        return lists;
    }

    public void setLists(List<Object> lists) {
        this.lists = lists;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", lists=" + lists +
                ", dog=" + dog +
                '}';
    }
}
```

使用到了@ConfigurationProperties注解，这里由于这个注解是容器提供的，所以需要把Person注入到容器中，也就是给Person用@Component注解，把Person注入到容器中。



**还需要导入一个依赖：**

```xml
<!-- 导入配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

编写yaml文件会有自动提示。



**application.yaml配置文件：**

```yaml
server:
  port: 8081

person:
  lastName: zhangsan
  age: 18
  boss: false
  birth: 2019/12/12
  maps: {k1: v1,k2: 12}
  lists:
    - lisi
    - zhaoliu
  dog:
    name: 小狗
    age: 2
```



**application.properties配置文件**

需要注意编码格式，否则乱码

```properties
server.port=8082

person.last-name=张三
person.age=19
person.birth=2019/12/12
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=狗
person.dog.age=18 
```



**测试：**

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootBoot02ConfigApplicationTests {

    @Autowired
    private Person person;

    @Test
    public void contextLoads() {

        System.out.println(person);

    }

}
```



**输出结果：**

```java
Person{lastName='zhangsan', age=18, boss=false, birth=Thu Dec 12 00:00:00 CST 2019, maps={k1=v1, k2=12}, lists=[lisi, zhaoliu], dog=Dog{name='小狗', age=2}}
```



## 配置文件值注入（@Value方式）

```java
@Value("${person.last-name}")
private String lastName;
@Value("#{11*11}")
private Integer age;
```

可以使用SpEL表达式。



## @Value和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties                 | @Value       |
| -------------------- | ---------------------------------------- | ------------ |
| 功能                 | 批量注入配置文件中的属性                 | 一个个指定   |
| 松散绑定（松散语法） | 如大写的可以用-来表示 lastName=last-name | 不支持       |
| SpEL表达式语言       | 不支持                                   | 支持#{}和${} |
| JSR303数据校验       | 支持                                     | 不支持       |
| 复杂类型封装         | 支持                                     | 不支持       |

在类上加`@Validated`注解，表示这个类下的属性需要校验。

在属性上加校验属性，如`@Email`，如果不是email则会报错。

**数据校验**

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    @Email
    private String lastName;
```

配置文件yaml还是properties他们都可以获取到值。

**如果说，**我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

**如果说，**我们专门编写了一个javabean来和配置文件进行映射，我们直接使用@ConfigurationProperties;



**注意**：@ConfigurationProperties读取的是全局配置文件。



## @PropertySource&@ImportResource

**@PropertySource** : 加载指定的配置文件。

```java
@Component
@ConfigurationProperties(prefix = "person")
//@Validated
@PropertySource(value = "classpath:person.properties")
public class Person {
```

**@ImportResource** : 导入Spring的配置文件，让配置文件里面的内容生效；

SpringBoot里面没有Spring的配置文件，我们自己编写的配置文件也不能自动识别；

想让Spring的配置文件生效，加载进来，就需要@ImpotResource标注在一个配置类上。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class="com.github.tomxwd.service.impl.HelloServiceImpl"></bean>

</beans>
```

以前的配置文件。

```java
@SpringBootApplication
@ImportResource(locations = {"classpath:beans.xml"})
public class SpringbootBoot02ConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootBoot02ConfigApplication.class, args);
    }

}
```

在配置类上写`@ImportResource(locations = {"classpath:beans.xml"})`，导入Spring的配置文件让其生效。



SpringBoot推荐给容器中添加组件的方式：推荐使用全注解的方式。

`@Configuration` 指名当前类是一个配置类，就是来取代之前的Spring配置文件。 

1. 配置类====Spring配置文件

2. 使用@Bean来注入组件。

   ```java
   /**
    * @author xieweidu
    * @createDate 2019-07-03 21:40
    * @Configuration 指名当前类是一个配置类,就是来取代之前的Spring配置文件。
    */
   @Configuration
   public class MyAppConfig {
   
   
       // 将方法的返回值添加到容器中，容器中这个组件默认的id就是方法名
       @Bean
       public HelloService helloService(){
           return new HelloServiceImpl();
       }
   
   }
   ```

   将方法的返回值添加到容器中，容器中这个组件默认的id就是方法名;



## 配置文件占位符

#### RandomValuePropertySource:配置文件中可以使用随机数

${random.value}、${random.int}、${random.long}、${random.int(10)}、${random.int[1024,65536]}、${random.uuid}

#### 属性配置占位符

可以在配置文件中引用前面配置过的属性（优先级前面配置过的这里都能用）。

${app.name:默认值}来指定找不到属性时的默认值。

配置文件：

```properties
server.port=8082

person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2019/12/12
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.last-name}狗
person.dog.age=18
```

有随机数，有引用前面的配置属性值。

结果：

```java
Person{lastName='张三294e3109-2381-4bbe-b93d-54bad4c9eef1', age=9826389, boss=false, birth=Thu Dec 12 00:00:00 CST 2019, maps={k1=v1, k2=14}, lists=[a, b, c], dog=Dog{name='张三0f1f0bba-9a6e-4689-b5e5-695f00b1db7f狗', age=18}}
```

如果说引用${person.hello}（不存在的属性值）则会保留原样如：`name='${person.hello}狗'`;

但是如果你用${person.hello:123}来替代，冒号:后面的值为默认值，如果没有就用后面的值。会得到`name='123狗'`；



## Profile

Profile是Spring对不同环境提供不同配置功能的支持，可以通过激活、指定参数等方式快速切换环境：

1. 多profile文件形式

   - 格式：application-{profile}.properties/yaml
     - application-dev.properties
     - application-prod.properties

   默认是使用application.properties文件

2. 多profile文档块模式（yaml文件支持）

   ```yaml
   server:
     port: 8081
   spring:
     profiles:
       active: test
   ---
   server:
     port: 1234
   spring:
     profiles: dev
   ---
   server:
     port: 1235
   spring:
     profiles: prod
   
   ---
   server:
     port: 1236
   spring:
     profiles: test
   ```

   在第一个文档块用spring.profiles.active来指定哪个环境

3. 激活方式

   - 命令行 --spring.profiles.active=dev；优先级高。
     - 打包完成之后，jar包启动的时候可以使用这个。例如：java -jar Xxx.jar --spring.profiles.active=dev;
     - 在IDEA中，即是修改启动环境中的Program arguments:为--spring.profiles.active=Xxx即可。
     - 其他的工具也一样，配置Program arguments命令行参数即可。
   - 配置文件 spring.profiles.active=dev；优先级低
   - jvm参数 -Dspring.profiles.active=dev；
     - 修改启动环境中的VM options:为 -Dspring.profiles.active=dev



## 配置文件的加载位置

SpringBoot启动会扫描以下位置的application.properties或者application.yml文件作为SpringBoot的默认配置文件

- file : ./config/	这里指，当前项目的根目录下
- file : ./
- classpath : /config/    
- classpath : /               这里指，在classpath下，即是默认的那个文件

以上是按照**优先级从高到低**的顺序，所有位置的文件都会被加载，**高优先级配置内容会覆盖低优先级配置内容**。

并不是说最高级的有了，就不看低级的了，SpringBoot会从这四个位置全部加载配置文件，是一种互补配置。

我们也可以通过配置**spring.config.location**来改变默认配置。**项目打包好以后，我们可以用使用命令行参数的形式，启动项目的时候来指定配置文件的新位置（磁盘上的位置），这个指定的配置文件也会跟默认加载的配置文件共同起作用形成互补配置。**`java -jar Xxx.jar --spring.config.location=F:/application.properties`。



## 外部配置加载顺序

SpringBoot支持多种外部配置方式，高优先级覆盖低优先级，**所有的配置会形成互补配置。**

这些方式的优先级如下：

[官网手册，有十七种优先级](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/htmlsingle/#boot-features-external-config)

1. 命令行参数

   - java -jar Xxx.jar **--server.port=8087**
   - java -jar Xxx.jar **--server.context-path=/boot**
   - java -jar Xxx.jar --server.port=8087 --server.context-path=/boot
   - 修改了端口、项目上下文路径等再启动，即是用命令行来控制，多个参数用空格来分隔

2. 来自java:comp/env的JNDI属性

3. Java系统属性（System.getProperties()）

4. 操作系统环境变量

5. RandomValuePropertySource配置的random.*属性值

   

   **以下：**

   **优先加载带profile的**

6. jar包外部的application-{profile}.properties或application.yml（带spring.profile）配置文件。

7. jar包内部的application-{profile}.properties或application.yml（带spring.profile）配置文件。

   

   **再来加载不带profile的**

8. jar包外部的application.properties或application.yml（不带spring.profile）配置文件。

9. jar包内部的application.properties或application.yml（不带spring.profile）配置文件。

   

   **由jar包外向jar包内进行搜找；**jar包外，即是在jar的同级目录下新建application.properties文件，可以覆盖jar包内的配置。

   

10. @Configuration注解类上的@PropertySource

11. 通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源，参考[官方文档。](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/htmlsingle/#boot-features-external-config)



## 自动配置原理

1. 可以查看HttpEncodingAutoConfiguration
2. 通用模式
   - XxxAutoConfiguration：自动配置类
   - XxxProperties：属性配置类
   - yml/properties文件中能配置的值就来源于[属性配置类]
3. 几个重要注解
   - @Bean
   - @Conditional
4. Debug查看详细的自动配置报告

配置文件到底能写什么？怎么写？自动配置原理是什么？

配置文件能配置的属性参照[官网文档](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/htmlsingle/#common-application-properties)

**自动配置原理：**

1. SpringBoot启动的时候加载主配置类，开启了自动配置功能**@EnableAutoConfiguration**；

2. @EnableAutoConfiguration 作用：

   - 利用AutoConfigurationImportSelector.class给容器中导入一些组件

   - 可以查看selectImports()方法的内容；

   - AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,      annotationMetadata);获取候选的配置
     - 再看里面调用的getCandidateConfigurations()方法，再看里面的List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),      getBeanClassLoader());再里面的loadSpringFactories()方法中，发现他获取了一个`FACTORIES_RESOURCE_LOCATION`（"META-INF/spring.factories"）的资源。即是**扫描所有jar包类路径下的"META-INF/spring.factories"**这个文件。然后把扫描到的文件的内容包装成properties对象。再从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加到容器中，点开其中一个META-INF/spring.factories文件，看有没有key为EnableAutoConfiguration对应的值，等号后面的所有全限定名的类，都是要扫描为组件的。
     - 总结就是一句话，把META-INF/spring.factories里面配置的所有EnableAutoConfiguration的值加入到容器中。
     - 每一个XxxAutoConfiguration类都是容器中的一个组件，都加入到容器中，用他们来做自动配置。

3. 每一个自动配置类进行自动配置功能。

4. 以HttpEncodingAutoConfiguration（Http编码自动配置）为例解释自动配置的原理。

   ```java
   @Configuration//表示这是一个配置类，以前编写的配置类作用一样，也可以给容器中添加组件
   @EnableConfigurationProperties(HttpProperties.class)//启用指定类的ConfigurationProperties功能；将配置文件中对应的值和HtppProperties绑定起来；并把HttpProperties加入到IOC容器中
   @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)//Spring底层@Conditonal注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；判断当前应用是否是Web应用，如果是，当前配置类生效
   @ConditionalOnClass(CharacterEncodingFilter.class)// 判断当前项目有没有这个类，SpringMVC中进行乱码解决的过滤器
   @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)//判断配置文件中，是否存在某个配置，spring.http.encoding.enabled;如果不存在，判断也是成立的，就是说如果不配置 就是true，配置了就是配置的属性值
   public class HttpEncodingAutoConfiguration {
       
       // 已经和springboot的配置文件映射了
   	private final HttpProperties.Encoding properties;
       
       // 只有一个有参构造器的情况下，参数的值就会从容器中拿
       public HttpEncodingAutoConfiguration(HttpProperties properties) 	{
   		this.properties = properties.getEncoding();
   	}
       
       @Bean//给容器中，添加一个组件，这个组件的某些值，需要从properties中获取
   	@ConditionalOnMissingBean
   	public CharacterEncodingFilter characterEncodingFilter() {
   		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
   		filter.setEncoding(this.properties.getCharset().name());
   		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
   		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
   		return filter;
   	}
   -------------------------------------------
       @ConfigurationProperties(prefix = "spring.http")//从配置文件中获取指定的值和bean的属性进行绑定
       public class HttpProperties {
   ```

   根据当前不同的条件判断，决定这个配置类是否生效？

   一旦这个配置类生效；这个配置类就会给容器中添加各种组件，这些组件的属性是从对应的properties类中获取的，这些类里的每一个属性又是和配置文件绑定的。

   @EnableConfigurationProperties(HttpProperties.class)

   ```java
   @ConfigurationProperties(prefix = "spring.http")//从配置文件中获取指定的值和bean的属性进行绑定
   public class HttpProperties {
   ```

   所有在配置文件中能配置的属性都是在XxxProperties类中封装着，配置文件能配置什么就可以参照某个功能对应的这个属性类

5. **@Conditional**派生注解（Spring注解版原生的@Conditional作用）

   - 作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置里面的所有内容才生效；

     | @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
     | ------------------------------- | ------------------------------------------------ |
     | @ConditionalOnjava              | 系统的Java版本是否符合要求                       |
     | @ConditionalOnBean              | 容器中存在指定Bean                               |
     | @ConditonalOnMissingBean        | 容器中不存在指定Bean                             |
     | @ConditionalOnExpression        | 满足SpEL表达式指定                               |
     | @ConditionalOnClass             | 系统中有指定的类                                 |
     | @ConditionalOnMissingClass      | 系统中没有指定的类                               |
     | @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
     | @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
     | @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
     | @ConditionalOnWebApplication    | 当前是web环境                                    |
     | @ConditionalOnNotWebApplication | 当前不是web环境                                  |
     | @ConditionalOnJndi              | JNDI存在指定项                                   |

     自动配置类必须在一定的条件下才能生效，所以spring.factories并不是所有的组件都会加载进来。比如此时的AspectJ，并不生效，没导入包。

   - 在application.properties中加上`debug=true`，开启SpringBoot的Debug模式。会打印CONDITIONS EVALUATION REPORT条件评估报告（自动配置报告），可以看到哪些自动配置类生效了。

6. **SpringBoot的精髓**

   - SpringBoot启动会加载大量的自动配置类

   - **我们看我们需要的功能有没有SpringBoot默认写好的自动配置类**
   - **我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件已经有了，我们就不需要再自己配置了）**
   - *给容器中自动配置类添加组件的时候，会从properties类中获取某些属性，我们就可以在配置文件中指定这些属性的值。**

   XxxAutoConfiguration：自动配置类，会给容器中添加组件

   XxxProperties：封装配置文件中相关属性 ；