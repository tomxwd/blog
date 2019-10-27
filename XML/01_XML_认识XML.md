---
title: 01_XML_认识XML
date: 2019-10-26 12:57:43
tags: 
 - XML
categories:
 - XML
---

# 01_XML_认识XML

## xml简介

- xml，eXtensible Markup Language，可扩展标记语言，是一种标记语言。

- xml是一种非常灵活的语言，没有固定的标签，所有的标签都可以自定义；

- 通常，xml被用于信息的记录和传递，因此xml经常被用于充当配置文件；



## 格式良好的xml格式

- XML编辑器
  - 记事本
  - Dreamweaver
  - XMLSpy



- 格式良好的XML文档：遵循XML文档的基本规则
  - 必须有XML声明语句
  - 必须**有且仅有一个根元素**
  - 标签大小写敏感
  - 属性值用双引号
  - 标签成对
  - 元素正确嵌套
- 有效的XML文档
  - 首先必须是格式良好的
  - 使用DTD和XSD（XML Schema）定义语义约束



- 声明信息，用于描述xml的版本和编码方式：

  ```xml
  <?xml version="1.0" encoding="utf-8" ?>
  ```

- xml有且仅有一个根元素；

- xml是大小写敏感的；

- 标签是成对的，而且要正确嵌套；

- 属性值要使用双引号；

- 注释的写法：

  ```xml
  <!--注释-->
  ```




## XML解析

- dom：（Document Object Model，文档对象模型）是W3C组织推荐的处理XML的一种方式，下面有两个分支：jDom和dom4j，可以对XML文件进行增删改查的操作；
  - Dom4j：dom下的一个分支。我们常用dom4j替换原生dom解析。
- sax：（Simple API for XML）不是官方标准，但是XML社区事实上的标准，几乎所有的XML解析器都支持它，只能进行解析（查询）；
  - pull：Pull解析和Sax解析很相似，都是轻量级的解析，它是一个第三方开源的Java项目，但是在Android的内核中已经嵌入了Pull，只能进行解析（查询）；

![1572066727188](01_XML_%E8%AE%A4%E8%AF%86XML/1572066727188.png)



## 实例

```xml
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6.1</version>
</dependency>
```

1. 导入dom解析包dom4j
2. 创建有个车SAXReader（一个xml文档阅读器）
3. 使用reader读取文件，返回一个document对象
4. 获取到document以后就可以执行增删改查
5. 先要获取根节点
6. 使用根节点继续往下找



- getName()：获取元素名
- getText()：获取元素里面的文本值
- attributeValue("属性名")：获取元素的属性值
- elementText("标签名")：获取ele子元素为标签名的文本值
- element()：找到第一个子元素(可带参数)
- elements()：找到所有子元素



```java
@Test
public void test1() throws Exception{
    SAXReader saxReader = new SAXReader();
    Document document = saxReader.read(new File("classpath:student.xml"));


}

@Test
public void test2() throws DocumentException {
    SAXReader saxReader = new SAXReader();
    Document document = saxReader.read(new File("E:\\IDEA_PROJECT\\JAVA_WEB\\xml-test\\src\\main\\resources\\student.xml"));
    Element rootElement = document.getRootElement();
    System.out.println("rootElement.getName() = " + rootElement.getName());
    // 获取当前结点下的所有子节点
    List<Element> elements = rootElement.elements();

    elements.stream().forEach(e-> {
        System.out.println("e.getName() = " + e.getName());
        System.out.println("e.attributeValue(\"id\") = " + e.attributeValue("id"));
    });
}

/**
 * xml转对象
 * @throws Exception
 */
@Test
public void test3() throws Exception{
    SAXReader saxReader = new SAXReader();
    Document read = saxReader.read(new File("E:\\IDEA_PROJECT\\JAVA_WEB\\xml-test\\src\\main\\resources\\student.xml"));
    Element rootElement = read.getRootElement();
    // 遍历-->封装
    ArrayList<Student> students = new ArrayList<>();
    ((List<Element>)rootElement.elements()).stream().forEach(e->{
        Student student = new Student();
        student.setId(e.attributeValue("id"));
        student.setName(e.elementText("name"));
        student.setAge(e.elementText("age"));
        students.add(student);
    });
    students.forEach(e-> System.out.println("student = " + e));
}

/**
 * 修改
 * @throws Exception
 */
@Test
public void test4() throws Exception{
    SAXReader saxReader = new SAXReader();
    Document document = saxReader.read(new File("E:\\IDEA_PROJECT\\JAVA_WEB\\xml-test\\src\\main\\resources\\student.xml"));
    Element rootElement = document.getRootElement();
    // 获取到的是第一个student
    Element student = rootElement.element("student");
    // 改student的name为张三三
    Element name = student.element("name");
    System.out.println("name.getText() = " + name.getText());
    name.setText("张三三");
    name.addAttribute("name","zhangzhang");
    // 保存修改
    // OutputFormat输出格式化
    OutputFormat prettyPrint = OutputFormat.createPrettyPrint();
    XMLWriter writer = new XMLWriter(new FileOutputStream("E:\\IDEA_PROJECT\\JAVA_WEB\\xml-test\\src\\main\\resources\\student.xml"),prettyPrint);
    writer.write(document);
    writer.close();
}
```



## xPath

xPath是在XML文档中查找信息的语言。

xPath是通过元素和属性进行查找。

xPath简化了Dom4j查找节点的过程。

使用xPath必须导入jaxen-1.1-beta-6.jar

否则出现NoClassDefFoundError：org/jaxen/JaxenException

```xml
<dependency>
    <groupId>jaxen</groupId>
    <artifactId>jaxen</artifactId>
    <version>1.1.6</version>
</dependency>
```



语法：

| 使用                                | 说明                                      |
| ----------------------------------- | ----------------------------------------- |
| /student/student                    | 从根元素开始逐层找，以"/"开头             |
| 开头//name                          | 直接获取所有name元素对象，以"//"开头      |
| //student/*                         | 获取所有student元素的所有子元素对象       |
| //student[1] 或者 //student[last()] | 获取所有student元素的第一个或最后一个元素 |
| //student[@id]                      | 获取所有带id属性的student元素对象         |
| //student[@id='002']                | 获取id等于002的student元素对象            |
| //student[not(@*)]                  | 获取所有student元素下不包含属性的元素对象 |





