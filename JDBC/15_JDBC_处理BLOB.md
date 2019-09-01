---
title: 15_JDBC_处理BLOB
date: 2019-08-23 09:54:30
tags: 
 - Java
categories:
 - JDBC
---

# 15_JDBC_处理BLOB

## Oracle LOB

- LOB，即Large Objects（大对象），是用来**存储大量的二进制和文本数据的一种数据类型**（一个LOB字段可存储多达4GB的数据）；
- LOB分为两种类型：内部LOB和外部LOB：
  - 内部LOB将数据以字节流的形式存储在数据库的内部。因而，内部LOB的许多操作都可以参数事务，也可以像处理普通数据一样对其进行备份和恢复操作
  - Oracle支持三种类型的内部LOB：
    - BLOB（二进制数据）
    - CLOB（单字节字符数据）
    - NCLOB（多字节字符数据）
  - CLOB和NCLOB类型适用于存储超长的文本数据，**BLOB字段适用于存储大量的二进制数据，如图像、视频、音频、文件等**。
  - 目前只支持一种外部LOB类型，即BFILE类型。在数据库内，该类型仅存储数据在操作系统中的位置信息，而数据的实体以外部文件的形式存在于操作系统的文件系统中。因而，该类型所表示的数据是只读的，不参与事务。该类型可帮助用户管理大量的由外部程序访问的文件。



## MySQL BLOB类型介绍

- MySQL中，BLOB是一个二进制大型对象，是一个可以存储大量数据的容器，它能容纳不同大小的数据。

- MySQL的四种BLOB类型（除了在存储的最大信息量上不同之外，其他都是等同的）

  | 类型       | 大小（单位：字节） |
  | ---------- | ------------------ |
  | TinyBlob   | 最大255            |
  | Blob       | 最大65K            |
  | MediumBlob | 最大16M            |
  | LongBlob   | 最大4G             |

- 实际使用中根据需要存储的数据大小定义不同的BLOB类型，需要注意的是：**如果存储的文件过大，数据库的性能会下降**；



## 测试BLOB

### 准备工作：

修改数据库customers表：

```sql
CREATE TABLE `customers` (
  `ID` int(6) NOT NULL AUTO_INCREMENT,
  `NAME` varchar(25) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `EMAIL` varchar(25) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `BIRTH` date DEFAULT NULL,
  `PICTURE` mediumblob,
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

准备一张图片用于插入；



### 插入测试：

```java
/**
 * 插入BLOB类型的数据必须使用PreparedStatement
 * 因为BLOB类型的数据无法使用字符串拼接
 */
@Test
public void testInsertBlob(){
    try {
        baseDao.update("insert into customers(name,email,birth,picture) values(?,?,?,?)"
                       , "blob","blob@qq.com",new Date(new java.util.Date().getTime()),new FileInputStream("C:\\Users\\weidou.xie\\Desktop\\pic.png"));
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
}
```

setObject可以插入，setBlob(int index,InputStream inputStream)也可以。



### 取出测试：

```java
@Test
public void testGetBlob(){
    Connection conn = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    String sql = "select picture from customers where id = 12";
    try {
        conn = JDBCTools.getConnection();
        ps = conn.prepareStatement(sql);
        rs = ps.executeQuery();
        if(rs.next()){
            Blob picture = rs.getBlob(1);
            InputStream in = picture.getBinaryStream();
            System.out.println("in.available() = " + in.available());
            OutputStream out = new FileOutputStream("C:\\Users\\weidou.xie\\Desktop\\pic2.png");
            byte[] buffer = new byte[1024];
            int len = 0;
            while ((len=in.read(buffer))!=-1){
                out.write(buffer,0,len);
            }
            out.close();
            in.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(rs,ps,conn);
    }
}
```

