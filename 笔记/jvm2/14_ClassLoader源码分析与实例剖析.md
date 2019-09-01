---
2019-08-19 14:38:21

---

#

## ClassLoader源码分析

作用：一个类加载器是一个对象，这个对象用于负责加载类。

ClassLoader是一个抽象类，如果给定了一个二进制名（以字符串名向CLassLoader提供的类的名字，必须是二进制的名字【全限定名】，如{java.lang.String，java.swing.JSpinner$DefaultEditor，java.security.KeyStore$Builder$FileBuilder$1，java.net.URLClassLoader$3$1}）,ClassLoader尝试去定位或者生成一些类定义相关的数据（动态代理）。典型的策略是将文件的二进制名字转换为文件名，然后从文件系统去读取这个文件名所对应的class文件。

每一个class对象都会包含一个对于定义它的ClassLoader的引用。

对于数组类的Class对象并不是由类加载器创建的，而是由Java虚拟机根据需要在运行时自动创建的。【只有数组是特殊的，其他都需要类加载器创建】；

对于一个数组的类加载器来说，Class.getClassLoader()返回的是数组元素类型的类加载器一致。如果元素是一个primitive【原生】类型，那么这个数组没有类加载器。

应用实现了ClassLoader类的子类，是为了扩展使JVM动态加载类的方式。

类加载器典型情况下是可以被安全管理器所使用去标识一些安全域问题。

ClassLoader类使用了一种委托模型去寻找类和资源，ClassLoader的每一个实例都会有一个与之关联的双亲ClassLoader。当ClassLoader实例被请求去尝试寻找一个类或者资源之前，ClassLoader的实例会将对于类或资源的寻找委托给其双亲加载器。

虚拟机内建的加载器我们称之为启动类加载器(根类加载器【BootStrapClassLoader】)，它没有双亲，但是它可以作为一个类加载器的双亲。

如果一个类加载器支持并发加载，那么我们称之为parallel capable加载器【并行的类加载器】，他们被要求在类进行初始化期间注册自身（调用ClassLoader.registerAsParallelCapable方法）。注意：ClassLoader默认是支持并行加载的，然而如果子类也是支持并行加载的，需要单独给子类也进行注册。

在委托模型并不是严格的层次化环境下（跟JVM内建的委托模型有出入的情况下），类加载器需要做到并行，否则类加载会导致deadlock【死锁】，因为loadlock【加载器锁】在类加载过程中是一直被持有的，那么其他类加载器持有其他锁，那么两个类加载器会造成死锁。

通常情况下，JVM是从本地的文件系统当中加载类（与平台相关的形式）。例如在UNIX系统中，JVM会从CLASSPATH环境变量所定义的目录加载类。

然而，一些类并不是来自于一个文件，他们可能来自于其他来源，比如通过网络，或者他们是由应用本身构建出来的（动态代理）。这种情况下，defineClass方法会将字节数组转换为Class类的实例，这个新定义的类的实例，可以通过Class.newInstance来创建。

由类加载器所创建的对象的方法和构造方法还可能引用其他的类（十分常见），为了确定被引用的类都是什么，JVM会调用类加载器的loadClass方法去加载他所引用的其他的类。

比如说：一个应用可以创建一个网络ClassLoader，然后从一个服务器上去把类文件下载过来：

```java
ClassLoader loader = new NetWorkClassLoader(host,port);
Object main = loader.loaderClass("Main",true).newInstance();
```

网络类加载器的子类必须要定义findClass方法以及loadClassData方法去从网络中加载一个类。一旦将类的字节码下载好后，它要使用defineClass方法去创建一个类的实例。如下：

```java
class NetWorkClassLoader extends ClassLoader{
    String host;
    int port;
    
    public Class findClass(String name){
        byte[] b = loadClassData(name);
        return defineClass(name,b,0,b.length);
    }
    
    private byte[] loadClassData(String){
        // load the class data from the connection
        ...
    }
}
```



## 实例剖析

代码：

```java
public class MyTest15 {

    public static void main(String[] args) {
        String[] str = new String[2];
        System.out.println("str.getClass().getClassLoader() = " + str.getClass().getClassLoader());
        System.out.println("---------------------------------------");
        A[] as = new A[2];
        System.out.println("as.getClass().getClassLoader() = " + as.getClass().getClassLoader());
        System.out.println("---------------------------------------");
        int[] ints = new int[2];
        System.out.println("ints.getClass().getClassLoader() = " + ints.getClass().getClassLoader());
    }

}
class A{

}
```

输出：

```
str.getClass().getClassLoader() = null
---------------------------------------
as.getClass().getClassLoader() = sun.misc.Launcher$AppClassLoader@58644d46
---------------------------------------
ints.getClass().getClassLoader() = null
```

注意第一个null指的是String的类加载器，而String的类加载器是BootStrapClassLoader，在这里getClassLoader的时候是null。

而第二个null指的是原生类型，没有类加载器，返回的是null；

