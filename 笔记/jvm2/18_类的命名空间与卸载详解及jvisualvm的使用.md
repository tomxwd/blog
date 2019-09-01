---
2019-08-20 14:39:13

---

#

![1566283174631](../数据结构/数据结构图解/1566283174631.png)

每一个类只能被加载一次这句话，是有个大前提：同一个命名空间；

如果是不同的命名空间，加载的两个不同的Class对象彼此是互相独立的。

下面看这个例子：

代码：

```java
public class MyTest16 extends ClassLoader{

    private String classLoaderName;

    private final String fileExtension = ".class";

    private String path;

    public void setPath(String path) {
        this.path = path;
    }

    public MyTest16(String classLoaderName){
        /**
         * 使用由方法getSystemClassLoader()所返回的类加载器作为他的parentClassLoader（双亲）
         */
        super();
        this.classLoaderName = classLoaderName;
    }

    /**
     * 会使用指定的parent做为这个类加载器的双亲
     * @param parent
     * @param classLoaderName
     */
    public MyTest16(ClassLoader parent,String classLoaderName){
        super(parent);
        this.classLoaderName = classLoaderName;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        System.out.println("findClass invoked：" + name);
        System.out.println("Classloadername：" + this.classLoaderName);
        byte[] data = this.loadClassData(name);
        return this.defineClass(name,data,0,data.length);
    }

    /**
     * 这个name就是用户传进来需要被加载的类的名字（二进制名 全限定名）
     * @param name
     * @return
     */
    private byte[] loadClassData(String name){
        InputStream is = null;
        byte[] data = null;
        ByteArrayOutputStream baos = null;

        name = name.replace(".","\\");

        try{
            is = new FileInputStream(new File(this.path+name+this.fileExtension));
            baos = new ByteArrayOutputStream();
            int ch = 0;
            while (-1 != (ch=is.read())){
                baos.write(ch);
            }
            data = baos.toByteArray();
        } catch (Exception e){
            e.printStackTrace();
        } finally {
            try {
                is.close();
                baos.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return data;
    }

    public static void main(String[] args) throws IllegalAccessException, InstantiationException, ClassNotFoundException {
        MyTest16 loader1 = new MyTest16("loader1");
//        loader1.setPath("E:\\IDEA-workspace\\jvm-test-01\\target\\classes\\");
        loader1.setPath("C:\\Users\\weidou.xie\\Desktop\\");
        Class<?> clazz = loader1.loadClass("top.tomxwd.classloader.MyTest1");
        System.out.println("clazz = " + clazz.hashCode());
        Object obj = clazz.newInstance();
        System.out.println("obj = " + obj);
        System.out.println("obj.getClass().getClassLoader() = " + obj.getClass().getClassLoader());
        System.out.println("----------------------------");
        MyTest16 loader2 = new MyTest16(loader1,"loader2");
        loader2.setPath("C:\\Users\\weidou.xie\\Desktop\\");
        Class<?> clazz2 = loader2.loadClass("top.tomxwd.classloader.MyTest1");
        System.out.println("clazz2 = " + clazz2.hashCode());
        Object obj2 = clazz2.newInstance();
        System.out.println("obj2 = " + obj2);
        System.out.println("obj2.getClass().getClassLoader() = " + obj2.getClass().getClassLoader());
    }

}
```

注意看loader2的实例，把loader1作为父加载器传入了。

输出：

```
findClass invoked：top.tomxwd.classloader.MyTest1
Classloadername：loader1
clazz = 356573597
obj = top.tomxwd.classloader.MyTest1@677327b6
obj.getClass().getClassLoader() = top.tomxwd.classloader.MyTest16@4554617c
----------------------------
clazz2 = 356573597
obj2 = top.tomxwd.classloader.MyTest1@14ae5a5
obj2.getClass().getClassLoader() = top.tomxwd.classloader.MyTest16@4554617c
```

记住前提是类路径下的MyTest1.class已经被删除，否则会被应用类加载器加载。

这时候输出结果很明显，由于loader2的父加载器是loader1，而loader1可以加载MyTest1，因此loader2并没有被执行加载。也就是说loader2加载的MyTest1是直接从loader1那边进行返回的，而loader1那边已经加载过了，所以是直接返回同一个Class对象。



修改main方法，添加loader3

代码：

```java
public static void main(String[] args) throws IllegalAccessException, InstantiationException, ClassNotFoundException {
    MyTest16 loader1 = new MyTest16("loader1");
    //        loader1.setPath("E:\\IDEA-workspace\\jvm-test-01\\target\\classes\\");
    loader1.setPath("C:\\Users\\weidou.xie\\Desktop\\");
    Class<?> clazz = loader1.loadClass("top.tomxwd.classloader.MyTest1");
    System.out.println("clazz = " + clazz.hashCode());
    Object obj = clazz.newInstance();
    System.out.println("obj = " + obj);
    System.out.println("obj.getClass().getClassLoader() = " + obj.getClass().getClassLoader());
    System.out.println("----------------------------");
    MyTest16 loader2 = new MyTest16(loader1,"loader2");
    loader2.setPath("C:\\Users\\weidou.xie\\Desktop\\");
    Class<?> clazz2 = loader2.loadClass("top.tomxwd.classloader.MyTest1");
    System.out.println("clazz2 = " + clazz2.hashCode());
    Object obj2 = clazz2.newInstance();
    System.out.println("obj2 = " + obj2);
    System.out.println("obj2.getClass().getClassLoader() = " + obj2.getClass().getClassLoader());
    System.out.println("-----------------------------");
    MyTest16 loader3 = new MyTest16("loader3");
    loader3.setPath("C:\\Users\\weidou.xie\\Desktop\\");
    Class<?> clazz3 = loader3.loadClass("top.tomxwd.classloader.MyTest1");
    System.out.println("clazz3 = " + clazz3.hashCode());
    Object obj3 = clazz3.newInstance();
    System.out.println("obj3 = " + obj3);
    System.out.println("obj3.getClass().getClassLoader() = " + obj3.getClass().getClassLoader());
}
```

输出：

```
findClass invoked：top.tomxwd.classloader.MyTest1
Classloadername：loader1
clazz = 356573597
obj = top.tomxwd.classloader.MyTest1@677327b6
obj.getClass().getClassLoader() = top.tomxwd.classloader.MyTest16@4554617c
----------------------------
clazz2 = 356573597
obj2 = top.tomxwd.classloader.MyTest1@14ae5a5
obj2.getClass().getClassLoader() = top.tomxwd.classloader.MyTest16@4554617c
-----------------------------
findClass invoked：top.tomxwd.classloader.MyTest1
Classloadername：loader3
clazz3 = 325040804
obj3 = top.tomxwd.classloader.MyTest1@45ee12a7
obj3.getClass().getClassLoader() = top.tomxwd.classloader.MyTest16@7f31245a
```



再次进行修改，把loader2作为loader3的父加载器

代码：

```java
public static void main(String[] args) throws IllegalAccessException, InstantiationException, ClassNotFoundException {
    MyTest16 loader1 = new MyTest16("loader1");
    //        loader1.setPath("E:\\IDEA-workspace\\jvm-test-01\\target\\classes\\");
    loader1.setPath("C:\\Users\\weidou.xie\\Desktop\\");
    Class<?> clazz = loader1.loadClass("top.tomxwd.classloader.MyTest1");
    System.out.println("clazz = " + clazz.hashCode());
    Object obj = clazz.newInstance();
    System.out.println("obj = " + obj);
    System.out.println("obj.getClass().getClassLoader() = " + obj.getClass().getClassLoader());
    System.out.println("----------------------------");
    MyTest16 loader2 = new MyTest16(loader1,"loader2");
    loader2.setPath("C:\\Users\\weidou.xie\\Desktop\\");
    Class<?> clazz2 = loader2.loadClass("top.tomxwd.classloader.MyTest1");
    System.out.println("clazz2 = " + clazz2.hashCode());
    Object obj2 = clazz2.newInstance();
    System.out.println("obj2 = " + obj2);
    System.out.println("obj2.getClass().getClassLoader() = " + obj2.getClass().getClassLoader());
    System.out.println("-----------------------------");
    MyTest16 loader3 = new MyTest16(loader2,"loader3");
    loader3.setPath("C:\\Users\\weidou.xie\\Desktop\\");
    Class<?> clazz3 = loader3.loadClass("top.tomxwd.classloader.MyTest1");
    System.out.println("clazz3 = " + clazz3.hashCode());
    Object obj3 = clazz3.newInstance();
    System.out.println("obj3 = " + obj3);
    System.out.println("obj3.getClass().getClassLoader() = " + obj3.getClass().getClassLoader());
}
```

输出：

```
findClass invoked：top.tomxwd.classloader.MyTest1
Classloadername：loader1
clazz = 356573597
obj = top.tomxwd.classloader.MyTest1@677327b6
obj.getClass().getClassLoader() = top.tomxwd.classloader.MyTest16@4554617c
----------------------------
clazz2 = 356573597
obj2 = top.tomxwd.classloader.MyTest1@14ae5a5
obj2.getClass().getClassLoader() = top.tomxwd.classloader.MyTest16@4554617c
-----------------------------
clazz3 = 356573597
obj3 = top.tomxwd.classloader.MyTest1@7f31245a
obj3.getClass().getClassLoader() = top.tomxwd.classloader.MyTest16@4554617c
```



## 卸载

![1566290672251](../数据结构/数据结构图解/1566290672251.png)

![1566290741745](../数据结构/数据结构图解/1566290741745.png)

![1566290835384](../数据结构/数据结构图解/1566290835384.png)



追踪类的卸载：-XX:+TraceClassUnloading

修改main方法：

代码：

```java
public static void main(String[] args) throws IllegalAccessException, InstantiationException, ClassNotFoundException {
    MyTest16 loader1 = new MyTest16("loader1");
    //        loader1.setPath("E:\\IDEA-workspace\\jvm-test-01\\target\\classes\\");
    loader1.setPath("C:\\Users\\weidou.xie\\Desktop\\");
    Class<?> clazz = loader1.loadClass("top.tomxwd.classloader.MyTest1");
    System.out.println("clazz = " + clazz.hashCode());
    Object obj = clazz.newInstance();
    System.out.println("obj = " + obj);
    System.out.println("obj.getClass().getClassLoader() = " + obj.getClass().getClassLoader());

    System.out.println("----------------------------");

    loader1 = null;
    clazz = null;
    obj = null;
    System.gc();

    System.out.println("----------------------------");

    loader1 = new MyTest16("loader1");
    loader1.setPath("C:\\Users\\weidou.xie\\Desktop\\");
    clazz = loader1.loadClass("top.tomxwd.classloader.MyTest1");
    System.out.println("clazz = " + clazz.hashCode());
    obj = clazz.newInstance();
    System.out.println("obj = " + obj);
    System.out.println("obj.getClass().getClassLoader() = " + obj.getClass().getClassLoader());
    System.out.println("----------------------------");
}
```

输出：

```
findClass invoked：top.tomxwd.classloader.MyTest1
Classloadername：loader1
clazz = 356573597
obj = top.tomxwd.classloader.MyTest1@677327b6
obj.getClass().getClassLoader() = top.tomxwd.classloader.MyTest16@4554617c
----------------------------
[Unloading class top.tomxwd.classloader.MyTest1 0x000000013faf1028]
----------------------------
findClass invoked：top.tomxwd.classloader.MyTest1
Classloadername：loader1
clazz = 1836019240
obj = top.tomxwd.classloader.MyTest1@135fbaa4
obj.getClass().getClassLoader() = top.tomxwd.classloader.MyTest16@14ae5a5
----------------------------
```



使用jvisualvm可以看卸载的类有多少。

