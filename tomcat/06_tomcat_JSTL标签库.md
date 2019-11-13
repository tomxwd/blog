---
title: 06_tomcat_JSTL标签库
date: 2019-11-10 10:17:22
tags: 
 - tomcat
categories:
 - tomcat
---

# 06_tomcat_JSTL标签库

## JSTL简介

- jsp虽然为我们提供了EL表达式来替代JSP表达式，但是由于EL表达式仅仅具有输出功能，而不替代页面中的jsp脚本片段。
- 为了解决这个问题，jsp为我们提供了可以自定义标签库的功能；
- 所谓自定义标签库就是指可以在jsp页面中以类似于HTML标签的形式调用java中的方法，使用方法和我们jsp动作标签类似。
- 而为了方便开发使用，sun公司又定义了一套通用的标签库名为JSTL（JSP Standard Tag Library），里面定义了很多我们开发中常用的方法，方便我们使用。
- JSTL的标准由sun公司定制，Apache中Jakarta小组负责实现。
- JSTL由5个不同功能的标签库组成。



## 使用JSTL

- 使用JSTL必须在项目中导入两个jar包

  - taglibs-standard-impl-1.2.1.jar
  - taglibs-standard-spec-1.2.5.jar

- 然后还需要在JSP页面中通过taglib标签引入标签库

  - `<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>`
  - prefix用来指定前缀名，我们可以通过该名来使用JSTL
  - uri相当于库的唯一标识，因为JSTL由多个不同的库组成，使用该属性指定要导入哪个库

- 使用JSTL

  - `<c:out value="hello"></c:out>`v

- JSTL由五个不同功能的标签库组成：

  | 功能范围         | URI                                    | 前缀 |
  | ---------------- | -------------------------------------- | ---- |
  | 核心             | http://java.sun.com/jsp/jstl/core      | c    |
  | 格式化           | http://java.sun.com/jsp/jstl/fmt       | fmt  |
  | 函数             | http://java.sun.com/jsp/jstl/functions | fn   |
  | 数据库（不使用） | http://java.sun.com/jsp/jstl/sql       | sql  |
  | XML（不使用）    | http://java.sun.com/jsp/jstl/xml       | x    |



## 核心标签（Core Tags）

- Core标签库，包括了我们最常用的标签

- 要使用Core标签库需要在JSP页面中引入：

  ```jsp
  <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
  ```



常用标签：

- **\<c:out>**

  - 用于计算一个表达式并将结果输出到当前页面
  - 类似JSP表达式\<%=>和EL表达式${}
  - 可以设置的属性：
    - value
      - 作用：要输出的值
      - 参数类型：Object
    - default
      - 作用：当value为null的时候显示的默认值
      - 参数类型：Object
    - escapeXml
      - 作用：是否对特殊字符进行转义
      - 参数类型：boolean
  - 例：
    - `<c:out value="${user.name}" default="" escapeXml="true"></c:out>`

- **\<c:set>**

  - 用于添加或修改域中的属性
  - 可以设置的属性：
    - value
      - 作用：要设置的值
      - 参数类型：Object
    - var
      - 作用：表示域中存放的属性名
      - 参数类型：String
    - scope
      - 作用：指定域（pageContext、request、session、application），若不指定则为pageContext
      - 参数类型：String
    - target
      - 作用：要修改的域对象的属性名（必须是JavaBean或Map）
      - 参数类型：Object
    - property
      - 作用：指定要修改的对象的属性名
      - 参数类型：String
  - 例：
    - 设置属性：
      - `<c:set var="key" value="value" scope="request"></c:set>`
    - 修改属性：
      - `<c:set property="name" target="${user}" value="孙悟空"></c:set>`

- \<c:remove>

  - \<c:remove>用于移除域中的属性
  - 可以设置的属性：
    - var
      - 作用：设置要移除的属性的名字
      - 参数类型：String
    - scope
      - 作用：设置要移除属性所在的域，**若不指定则删除所有域中的对应属性**
      - 参数类型：String
  - 例：
    - 移除所有域中key属性：`<c:remove var="key"/>`
    - 移除request中的key属性：`<c:remove var="key" scope="request"/>`

- \<c:if>

  - \<c:if>用于实现if语句的判断功能
  - 可设置的属性
    - test
      - 作用：设置if判断的条件，用于判断标签体是否被执行
      - 参数类型：boolean
    - var
      - 作用：用于指定接收判断结果的变量名
      - 参数类型：boolean
    - scope
      - 作用：指定判断结果保存到哪个域中
      - 参数类型：String
  - 例：
    - `<c:if test="${empty user}" var="isUserEmpty" scope="request">用户为空</c:if>`

- \<c:choose>、\<c:when>、\<c:otherwise>

  - \<c:choose>、\<c:when>、\<c:otherwise>三个标签配合使用，功能类似于Java中的switch/case

  - \<c:choose>是\<c:when>和\<c:otherwise>的父标签

  - \<c:when>的属性：

    - test
      - 作用：用于设置判断条件，若正确则c:when中的代码执行，否则不执行
      - 参数类型：boolean

  - \<c:otherwise>

    - 作用：如果所有的\<c:when>都没有执行则执行\<c:otherwise>的标签体

  - 例：

    ```jsp
    <c:choose>
        <c:when test="${param.age>18}">
            您已经成年
        </c:when>
        <c:when test="${param.age==18}">
            您刚好成年
        </c:when>
        <c:otherwise>
            您未成年
        </c:otherwise>
    </c:choose>
    ```

- \<c:forEach>

  - \<c:forEach>用于对多个对象的集合进行迭代，重复执行标签体，或者重复迭代固定的次数

  - 可设置属性：

    - var
      - 作用：设置遍历出对象的名称
      - 参数类型：String
    - items
      - 作用：指定要遍历的集合对象
      - 参数类型：数组、字符串和各种集合
    - varStatus
      - 作用：指定保存迭代状态的对象的名字，该变量引用的是一个LoopTagStatus类型的对象，通过该对象可以获得一些遍历的状态：
        - begin：开始位置
        - end：结束位置
        - count：第几个，从1开始
        - index：索引，从0开始
        - step：遍历的步长
        - first：是否为第一个元素
        - last：是否为最后一个元素
        - name
      - 参数类型：String
    - begin
      - 作用：指定遍历的开始位置
      - 参数类型：int
    - end
      - 作用：指定遍历的结束位置
      - 参数类型：int
    - step
      - 作用：迭代的步长
      - 参数类型：int

  - 例：

    ```jsp
    <c:forEach items="${list}" var="user" begin="0" end="4" step="2" varStatus="vs">
    	${vs.index} -- ${user.name} -- ${user.age} <br/>
    </c:forEach>
    ```

- \<c:url>

  - \<c:url>主要用来重写URL地址

    - 可设置的属性：
      - value
        - 作用：设置要处理的URI地址，注意这里要以/开头
        - 可接受参数：String
      - var
        - 作用：修改后存储到域对象中的uri属性名
        - 可接受参数：String
      - scope
        - 作用：设置修改后uri存放的域
        - 可接受参数：String

  - 例：

    - 使用相对路径：

      ```jsp
      <c:url value="index.jsp" var="uri" scope="request">
          <c:param name="name" value="张三"></c:param>
      </c:url>
      ```

      - 会生成地址：index.jsp?name=%E5%BC%A0%E4%B8%89

    - 使用绝对路径会自动在路径前面加上项目名：

      ```jsp
      <c:url value="/index.jsp" var="uri" scope="request">
          <c:param name="name" value="张三"></c:param>
      </c:url>
      ```

      - 会生成地址：/Test_JSTL/index.jsp?name=%E5%BC%A0%E4%B8%89

- \<c:redirect>

  - \<c:redirect>主要用于将请求重定向到另一个资源地址
  - 可设置的属性
    - uri
      - 作用：指定要重定向到的目标地址，注意这里指定绝对路径会自动加上项目名
      - 参数类型：String
  - 例：
    - `<c:redirect url="/target.jsp"></c:redirect>`



## JSTL函数（JSTL Functions）

- 函数标签库是在JSTL中定义的标准的EL函数集
- 函数标签库中定义的函数基本上都是对字符串的操作
- 引入：`<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions"%>`



### fn:contains和fn:containslgnoreCase

作用：用于判断字符串中是否包含指定字符串，containslgnoreCase忽略大小写。

语法：fn:contains(string,subString)返回boolean

| 参数      | 类型    | 作用                                             |
| --------- | ------- | ------------------------------------------------ |
| string    | String  | 源字符串                                         |
| subString | String  | 要查找的字符串                                   |
| 返回值    | boolean | 若string中包含subString则返回true，否则返回false |

例:

- ${fn:contains("hello","HE")}	false
- ${fn:containsIgnoreCase("hello","HE")}    true



### fn:startsWith和fn:endsWith

作用：判断一个字符串是否以指定字符开头（startsWith）或结尾（endsWith）

语法：fn:endsWith(string,suffix)返回boolean

| 参数           | 类型    | 作用                            |
| -------------- | ------- | ------------------------------- |
| string         | String  | 源字符串                        |
| prefix或suffix | String  | 要查找的前缀或后缀字符串        |
| 返回值         | boolean | 符合要求返回true，否则返回false |

例：

- ${fn:startsWith("hello","he")}	true
- ${fn:endsWith("hello","he")}    false



### fn:indexOf

作用：在一个字符串中查找指定字符串，并返回第一个符合的字符串的第一个字符的索引。

语法：fn:indexOf(string,subString)返回int

| 参数      | 类型   | 作用                                                         |
| --------- | ------ | ------------------------------------------------------------ |
| string    | String | 源字符串                                                     |
| subString | String | 要查找的字符串                                               |
| 返回值    | int    | 若在string中找到subString则返回第一个符合的索引，若没有符合的则返回-1 |

例：

- ${fn:indexOf("hello",'e')}	1



### fn:replace

作用：将一个字符串替换为另外一个字符串，并返回替换结果

语法：fn:replace(str,beforeSubString,afterSubString)返回String

| 参数            | 类型   | 作用             |
| --------------- | ------ | ---------------- |
| str             | String | 源字符串         |
| beforeSubString | String | 被替换的字符串   |
| afterSubString  | String | 要替换的新字符串 |
| 返回值          | String | 替换后的字符串   |

例：

- ${fn:replace("hello","llo",'e')}	hee



### fn:substring

作用：截取字符串

语法：fn:substring(str,beginIndex,endIndex)返回String

| 参数       | 类型   | 作用                       |
| ---------- | ------ | -------------------------- |
| str        | String | 源字符串                   |
| beginIndex | int    | 开始位置索引（包含该位置） |
| endIndex   | int    | 结束位置索引（不包含自身） |
| 返回值     | String | 返回截取的字符串           |

例：

- ${fn:substring("hello",1,3)}	el



### fn:substringBefore和fn:substringAfter





## 自定义标签

![1573370964913](06_tomcat_JSTL%E6%A0%87%E7%AD%BE%E5%BA%93/1573370964913.png)

1. 编写标签库的描述文件（描述标签的详细信息）

   - 库描述文件放在WEB-INF/tags
   - 创建xml的schema文件
   - 选择XML Catalog为http://xmlns.jcp.org/xml/ns/j2ee/web-jsptaglibrary_2_0.xsd
   - 去除j2ee前缀

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   
   <taglib xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd"
           version="2.1">
   
       <!-- 标签库的版本 -->
       <tlib-version>1.0</tlib-version>
       <!-- 指定下面所有当前标签库标签的前缀 -->
       <short-name>tomxwd</short-name>
       <!-- 标签库的唯一标识 -->
       <uri>http://www.tomxwd.top/tags/mytag</uri>
   
       <!-- Invoke 'Generate' action to add tags or functions -->
       <!-- 定义一个可以使用的标签 -->
       <tag>
           <!-- 定义标签名 -->
           <name>hello</name>
           <!-- 定义标签的实现类 -->
           <tag-class>top.tomxwd.tag.MyTag</tag-class>
           <!--
               empty       ：是一个空标签，就是没有标签体，代表当前是一个自结束标签，如<br/>
               scriptless  ：不可以传jsp表达式，只能穿el及其他正常内容
               JSP         ：跟scriptless比，还可以传入jsp表达式等等
               tagdependent：传入是啥就是啥
           -->
           <body-content>empty</body-content>
           <!-- 使用attribute定义属性 -->
           <attribute>
               <!-- name指定属性名 -->
               <name>msg</name>
               <!-- 表示必须写该属性 -->
               <required>true</required>
               <!-- rtexprvalue表示运行时表达式的值 -->
               <rtexprvalue>true</rtexprvalue>
           </attribute>
       </tag>
   
   </taglib>
   ```

2. 编写标签的实现类 （标签处理器类）

   ```java
   package top.tomxwd.tag;
   
   import javax.servlet.jsp.JspContext;
   import javax.servlet.jsp.JspException;
   import javax.servlet.jsp.PageContext;
   import javax.servlet.jsp.tagext.JspFragment;
   import javax.servlet.jsp.tagext.JspTag;
   import javax.servlet.jsp.tagext.SimpleTag;
   import java.io.IOException;
   
   public class MyTag implements SimpleTag {
   
       private String msg;
       private PageContext pc;
   
       public String getMsg() {
           return msg;
       }
   
       public void setMsg(String msg) {
           System.out.println("msg = " + msg);
           this.msg = msg;
       }
   
       /**
        * 执行标签功能
        * @throws JspException
        * @throws IOException
        */
       @Override
       public void doTag() throws JspException, IOException {
           System.out.println("MyTag.doTag");
           pc.getOut().write("<h1>"+msg+"</h1>");
       }
   
       /**
        * 设置父标签（服务器自动传进来）
        * @param parent
        */
       @Override
       public void setParent(JspTag parent) {
           System.out.println("MyTag.setParent");
       }
   
       /**
        * 获取父标签（只特指自定义标签）
        * @return
        */
       @Override
       public JspTag getParent() {
           System.out.println("MyTag.getParent");
           return null;
       }
   
       /**
        * 设置JspContext 就等于 pageContext（服务器自动传入）
        * @param pc pageContext
        */
       @Override
       public void setJspContext(JspContext pc) {
           System.out.println("MyTag.setJspContext");
           this.pc = ((PageContext) pc);
       }
   
       /**
        * 设置JsoBody（标签体内容，服务器自动传入）
        * @param jspBody
        */
       @Override
       public void setJspBody(JspFragment jspBody) {
           System.out.println("MyTag.setJspBody");
       }
   }
   ```

3. 在页面引入并使用

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <%@ taglib prefix="tomxwd" uri="http://www.tomxwd.top/tags/mytag" %>
   <html>
     <head>
       <title>HelloWorld</title>
       <base href="http://localhost:8080/test/">
     </head>
     <body>
     <tomxwd:hello msg="你好，这是msg"/>
     </body>
   </html>
   ```





