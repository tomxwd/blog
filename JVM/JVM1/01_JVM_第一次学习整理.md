---
title: 01_JVM_第一次学习整理
date: 2019-08-05 10:44:18
tags: 
 - JVM
categories:
 - JVM
 - JVM1
---

# 01_JVM_第一次学习整理

## JVM-简析

![01_JVM_JVM组成结构1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_JVM%E7%BB%84%E6%88%90%E7%BB%93%E6%9E%841.png)

![01_JVM_JVM组成结构2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_JVM%E7%BB%84%E6%88%90%E7%BB%93%E6%9E%842.png)

Thread.State;

1. NEW：就绪状态
2. RUNNABLE：运行中
3. BLOCKED：阻塞
4. WAITING：等待
5. TIMED_WAITING：等待一定时间
6. TERMINATED：终结

Java native Interface（JNI）：用底层JVM本地库接口，去调操作系统底层的函数库，说白点就是调用C了；



1. Class Loader类加载器

   负责加载class文件，class文件在文件开头有特定的文件标示，并且ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定。

2. Native Interface

   本地接口的作用是融合不同的编程语言为Java所用，它的初衷是融合C/C++程序，Java诞生的时候是C/C++横行的时候，要想立足，必须要调用C/C++程序，于是就在内存中专门开辟了一块区域处理标记为native方法，它的具体做法是Native Method Stack中登记nativ方法，在Execution Engine执行的时候加载native libraies。

   目前该方法使用的越来越少了，除非是与硬件有关的应用，比如通过Java程序驱动打印机，或者Java系统管理生产设备，在企业级应用中已经比较少见了。

   因为现在的异构领域间通信很发达，比如可以使用Socket通信，也可以使用WebService等等，不多做介绍。

3. Method Area 方法区

   方法区是被所有线程共享的，所有字段和方法字节码，以及一些特殊方法如构造函数，接口代码也在此定义。

   简单说，所有定义的方法的信息都保存在该区域，此区域属于共享区间。

   **静态变量+常量+类信息+运行时常量池存在方法区中+实例变量存在堆内存中**

4. PC Register程序计数器

   每个线程都有一个程序计数器，就是一个指针，指向方法中的方法字节码（下一个将要执行的指令代码），由执行引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不计。

5. Native Method Stack 本地方法栈

   它的具体做法是Native Method Stack中登记native方法，在Execution Engine执行时加载native libraies。

6. Stack栈是什么

   栈也叫栈内存，主管Java程序的运行，是在线程创建时创建，它的生命期是跟随线程的生命期，线程结束栈内存也就释放，**对于栈来说不存在垃圾回收问题**，因为只要线程一结束该栈就Over，生命周期和线程一致，是线程私有的。**基本类型的变量和对象的引用变量都是在函数的栈内存中分配**；

   - 栈存储什么？

     栈帧中主要保存三类数据：

     - 本地变量（Local Variables）：输入参数和输出参数以及方法内的变量；
     - 栈操作（Operand Stack）：记录出栈、入栈的操作；
     - 栈帧数据（Frame Data）：包括类文件、方法等等；

   - 栈运行原理：

     栈中的数据都是以栈帧（Stack Frame）的格式存在，栈帧是一个内存区块，是一个数据集，是一个有关方法（Method）和运行期数据的数据集，当一个方法A被调用的时候就产生一个栈帧F1，并被压入栈中，A方法又调用B方法，于是产生栈帧F2也被压入栈，B方法又调用了C方法，于是产生栈帧F3也被压入栈......执行完毕后，先弹出F3栈帧，再弹出F2栈帧，再弹出F1栈帧......

     遵循**“先进后出”/“后进先出”**的原则；

7. ![01_JVM_JavaStack图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_JavaStack%E5%9B%BE.png)
   - java.lang.StackOverflowError

     ![01_JVM_JavaStack图2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_JavaStack%E5%9B%BE2.png)
     
	- 判断JVM优化是哪里？
	 ![01_JVM_JVM运行时数据](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_JVM%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE.png)
	
	- 三种JVM
	
	 - Sun公司的HotSpot；
	 - BEA公司的JRockit；
	 - IBM公司的J9 VM；
	
8. Heap堆

   ![01_JVM_堆内存示意图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_%E5%A0%86%E5%86%85%E5%AD%98%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

   一个JVM实例只存在一个堆内存，堆内存的大小是可以调节的。类加载器读取了类文件之后，需要把类、方法、常变量放到堆内存中，保存所有的引用类型的真实信息，以方便执行器执行，堆内存分为三部分：

   - Young Generation Space 新生区	 Young

     新生去是类的诞生、成长、消亡的区域，一个类在这里产生，应用，最后被垃圾回收器收集，结束生命。新生区又分为两部分：伊甸区（Eden space）和幸存区（Survivor space），所有的类都是在伊甸区被new出来的。

     幸存区有两个：0区（Survivor 0 space）和1区（Survivor 1 space）。当伊甸区的空间被用完时，程序又需要创建对象，JVM的垃圾回收器将对伊甸区进行垃圾回收（Minor GC），将伊甸区中的不再被其他对象所引用的对象进行销毁，然后将伊甸区中的剩余对象移动到幸存0区。若幸存0区也满了，再对该区进行垃圾回收，然后移动到1区。那如果1区也满了呢？再移动到养老区。若养老区也满了，那么这个时候将产生Major GC（FullGC），进行养老区的内存清理。若养老区执行了Full GC之后发现依然无法进行对象的保存，就会产生OOM异常【OutOfMemoryError】；

     如果出现java.lang.OutOfMemoryError:Java heap space异常，说明Java虚拟机的堆内存不够。原因有二：

     - Java虚拟机的堆内存设置不够，可以通过参数-Xms、-Xmx来调整。
     - 代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）。

   - Tenure Generation Space 养老区   Old

     养老区用于保存从新生区筛选出来的Java对象，一般池对象都在这个区域活跃。

   - Permanent Space 永久区                 Perm

     永久存储区是一个常驻内存区域，用于存放JDK自身所携带的Class，Interface的元数据，也就是说它存储的是运行环境必须的类信息，被装在进此区域的数据是不会被垃圾回收器收掉的，关闭JVM才会释放此区域所占用的内存。

     如果出现java.lang.OutOfMemoryError: PermGen space，说明是Java虚拟机对永久代Perm内存设置不够。一般出现这种情况，都是程序启动需要加载大量的第三方jar包。例如：在一个Tomcat下部署了太多的应用，或者大量动态反射生成的类不断被加载，最终导致Perm区被栈满。

     - JDK1.6及以前：有永久代，常量池1.6在方法区；
     - JDK1.7：            有永久代，但已经逐步“去永久代”，常量池1.7在堆；
     - JDK1.8:及以后：无永久代，常量池1.8在元空间；

9. ![01_JVM_jdk7程序内存划分小总结1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_jdk7%E7%A8%8B%E5%BA%8F%E5%86%85%E5%AD%98%E5%88%92%E5%88%86%E5%B0%8F%E6%80%BB%E7%BB%931.png)

10. ![01_JVM_jdk7程序内存划分小总结2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_jdk7%E7%A8%8B%E5%BA%8F%E5%86%85%E5%AD%98%E5%88%92%E5%88%86%E5%B0%8F%E6%80%BB%E7%BB%932.png)

11. ![01_JVM_jdk7程序内存划分小总结3](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_jdk7%E7%A8%8B%E5%BA%8F%E5%86%85%E5%AD%98%E5%88%92%E5%88%86%E5%B0%8F%E6%80%BB%E7%BB%933.png)

12. ![01_JVM_jdk7程序内存划分小总结4](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_jdk7%E7%A8%8B%E5%BA%8F%E5%86%85%E5%AD%98%E5%88%92%E5%88%86%E5%B0%8F%E6%80%BB%E7%BB%934.png)

13. ![01_JVM_jdk1.7gc图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_jdk1.7gc%E5%9B%BE.png)

14. ![01_JVM_jdk1.8gc图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_jdk1.8gc%E5%9B%BE.png)

15. ![01_JVM_gc-3](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_gc-3.png)

16. ![01_JVM_gc-3-1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_gc-3-1.png)

17. ![01_JVM_gc-3-2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_gc-3-2.png)

18. ![01_JVM_gc-3-3](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_gc-3-3.png)

19. ![01_JVM_gc-3-4](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_gc-3-4.png)

    GC是什么？

    - 频繁收集Young区
    - 较少收集Old区
    - 基本不动Perm区

    JVM在进行GC时，并非每次都对上面三个内存区域一起回收的，大部分时间回收都是指新生代。

    因此GC按照回收的区域又分了两种类型，一种是普通GC（minor GC），一种是全局GC（major GC or Full GC）

    - 普通GC（minor GC）：只针对新生代区域的GC；
    - 全局GC（major GC or Full GC）：针对老年代的GC，偶尔伴随对新生代GC以及对永久代的GC；

20. 普通GC（minor GC）一般发生在新生代，频繁发生，此区域用到的算法是复制算法（Copying）；

    Minor GC会把Eden中所有活的对象都移到Survivor区域中，如果Survivor去中放不下，那么剩下的活的对象就被移动到Old generation中，**也就是说一旦收集后，Eden就变成空了。**

    当对象在Eden（包括一个Survivor区域，这里假设是from区域）出生后，在经历一次Minor GC后，如果对象还存活，并且能够被另一块Survivor区域所容纳（上面已经假设是from区域，这里应为to区域，即to区域有足够的内存空间来存储Eden和from区域中存活的对象），则使用**复制算法**将这些仍然还存活的对象复制到另一块Survivor区域（即to区域）中，然后清理所使用过的Eden以及Survivor区域（即from区域），并且将这些对象的年龄设置为1，以后对象在Survivor区域每熬过一次Minor GC，就将对象的年龄+1，当对象的年龄达到某个值时（默认是15岁，通过**-XX:MaxTenuringThreshold**【设置对象在新生代中存活的次数】来设定参数），这些对象就会成为老年代。

    **谁空谁是to，复制之后要交换**；

21. 年轻代中的GC，主要是复制算法（Copying）

    HotSpot JVM把年轻代分为了三个部分：1个Eden区和两个Survivor区（分别叫from和to）。默认比例是8:1:1，一般情况下新创建的对象都会被分配到Eden区（一些大对象特殊处理），这些对象经过第一次Minor GC之后，如果存活，就会被移到Survivor区。对象在Survivor区中每熬过一次Minor GC，年龄就会增加1岁，当它的年龄增加到一定程度时，就会被移动到年老代中。因为年轻代中的对象基本都是朝生夕死（80%以上），所以在**年轻代的垃圾回收算法使用的是复制算法**，复制算法的基本思想就是讲内存分为两块，每次只用其中一块，当这一块内存用完，就将还活着的对象复制到另一块上面，**复制算法不会产生内存碎片，但是会浪费一成的空间**；

    在GC开始的时候，对象只存在于Eden区和名为“From”的Survivor区，Survivor区“TO”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“TO”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值（年龄阈值，可以通过**-XX:MaxTenuringThreshold**来设置）的对象会被移动到年老代中，没有达到阈值的对象会被复制到“TO”区域。**经过这次的GC后，Eden区和From区已经被清空，这个时候，“From”和“TO”会交换他们的角色，也就是新的“TO”就是上次GC前的“From”，新的“From”就是上次GC前的“TO”。**不管怎样，都会保证名为“TO”的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“TO”区域被填满，“TO”区域被填满后，会将所有的对象都移动到年老代中。

22. ![GC图解](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_GC%E5%9B%BE%E8%A7%A3.png)

    因为Eden区对象一般存活率较低，一般的，使用两块10%的内存作为空闲和活动区间，而另外80%的内存，则是用来给新建对象分配内存的，一旦发生GC，将10%的活动区间与另外80%中存活的对象转移到10%的空闲区间，接下来，将之前90%的内存全部释放，以此类推。

23. 复制算法弥补了标记-清除算法中，内存布局混乱的缺点。不过与此同时，它的缺点也是相当明显的。

    - 它浪费了一半的内存，这太要命了。
    - 如果对象的存活率很高，我们可以极端一点，假设是100%存活，那么我们需要将所有的对象都复制一遍，并将所有引用地址重置一遍。复制这一工作所需要花费的时间，在对象存活率达到一定程度时，将会变得不可忽视。
    - 所以从以上描述不难看出，复制算法想要使用，**最起码对象的存活率要非常低才行**，而且最重要的是，我们必须要克服50%内存的浪费。

24. 老年代一般是由标记-清除或者是标记-清除与标记-整理的混合实现

25. 标记-清除（Mark-Sweep）

    ![01_JVM_标记清除图解](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_%E6%A0%87%E8%AE%B0%E6%B8%85%E9%99%A4%E5%9B%BE%E8%A7%A3.png)

    劣势：

    - 首先，它的缺点就是效率比较低（递归和全堆对象遍历），而且在进行GC的时候，需要停止应用程序，这回导致用户体验非常差劲。
    - 其次，主要的缺点泽森这种方式清理出来的空闲内存是不连续的，这点不难理解，我们死亡的对象都是随机出现在内存的各个角落的，现在把它们清除后，内存的布局自然会乱七八糟。而为了应付这点，**JVM就不得不维持一个内存的空闲列表**，这又是一种开销。而且在分配数组对象的时候，寻找连续的内存空间会不太好找。

26. 标记-整理（Mark-Compact）

    存活的打标记

    ![01_JVM_标记整理图解](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/jvm/jvm1/01_JVM_%E6%A0%87%E8%AE%B0%E6%95%B4%E7%90%86%E5%9B%BE%E8%A7%A3.png)

    劣势：

    - 标记-整理算法唯一的缺点就是效率也不高，不仅要标记所有存活的对象，还要整理所有存活对象的引用地址。从效率上说，标记-整理算法要低于复制算法。

27. 总结：

    - 内存效率：复制算法>标记-清除算法>标记-整理算法（此处的效率只是简单对比时间复杂度，实际情况不一定如此）；

    - 内存整齐度：复制算法=标记-整理算法>标记-清除算法。
- 内存利用率：标记-整理算法=标记-清除算法>复制算法
    
    可以看出，效率上来说，复制算法是当之无愧的老大，但是却浪费了太多内存，而为了尽量兼顾上面所提到的三个指标，标记-整理算法相对来说更平滑一些，但效率上依然不尽如人意，它比复制算法多了一个标记的阶段，又比标记-清除算法多了一个整理内存的过程。

    难道就没有一种最优算法吗？

    答案：无，没有最好的算法，只有最合适的算法==>**分代收集算法**；

28. 问题：
    - JVM内存模型以及分区，需要详细到每个区放什么。
    - 堆里面的分区：Eden，Survivor From TO，老年代，各自的特点。
    - GC的三种收集方法：标记-清除、标记-整理、复制算法的原理与特点。
    - Minor GC和Full GC分别在什么时候发生。







