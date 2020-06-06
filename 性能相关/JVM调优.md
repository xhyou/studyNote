# 类加载机制

>  虚拟机把描述类的数据从Class文件加载到内存,并对数据尽行校验,转换解析和初始化，最终形成可以被虚拟机直接使用的java类型吗,这就是虚拟机的类加载机制

## 类加载的过程

* 加载 链接(验证 赋予默认值 解析) 初始化

待补充

# Java中的内存划分

## 五个内存空间

* 程序计数器

  > 保证线程切换后能恢复到原来的位置

* 虚拟机栈

  > 栈内存为虚拟机执行java方法。方法被调用创建执行栈-》局部变量表-》局部变量-》引用对象

* 本地方法栈

  > 为虚拟机执行使用到的native方法的服务

* 堆内存

  > 存放所有new出来的东西

* 方法区

  > 存储被虚拟机加载的信息,常量,静态常量,静态方法等。

# 如何定位垃圾?

* 引用计数

  > 给对象添加一个引用计数器,每当有地方使用到它的时候，计数器就加1；当引用失效的时候，计数器就减1；任何时候计数器为0的对象是不可能再被使用的
  >
  > 优点:算法实现简单，效率高
  >
  > 缺点:引用和去引用伴随的加法和减法，影响性能
  >
  > ​           对于循环引用对象无法尽行回收

* 根可达算法

  > 设立若干种根对象,当任何一个根对象到某一个对象不可达时，则认为这个对象是可以被回收的。

# 根

* 可以被当做根的对象有这么四种

  > 栈中的引用对象（指本地变量表）
  >
  > 方法区的静态成员
  >
  > 方法去中的常量引用的对象(指final的常量值)
  >
  > 本地方法栈中的JNI应用的对象（指本地变量表）

# 垃圾回收算法

* 标记清除

  > 标记阶段，先通过根节点，标记所有从根节点开始的可达对象。未被标记的则为垃圾
  >
  > 缺点:位置不连续，产生碎片。效率偏低（需要扫2遍）

* 拷贝算法

  > 将原来的内存空间分为2块,在垃圾回收时,将正在使用的存活对象赋值到未使用的内存块中，然后清除正在使用的内存块中的所有对象
  >
  > 优点:效率高，不产生碎片
  >
  > 缺点:空间的浪费（需要移动指针）

* 标记压缩

  > 将所有存活的对象压缩到内存的一端,之后清理边界外的所有内存空间
  >
  > 优点:不产生碎片
  >
  > 缺点: 效率偏低

# JVM内存的分代模型

* 新生代 + 老年代 + 永久代（1.7）Perm Generation/ 元数据区(1.8) Metaspace

  > 1. 永久代 元数据 - Class
  > 2. 永久代必须指定大小限制 ，元数据可以设置，也可以不设置，无上限（受限于物理内存）
  > 3. 字符串常量 1.7 - 永久代，1.8 - 堆
  > 4. MethodArea逻辑概念 - 永久代、元数据

* 新生代 = Eden + 2个suvivor区 

  > 1. YGC回收之后，大多数的对象会被回收，活着的进入s0
  > 2. 再次YGC，活着的对象eden + s0 -> s1
  > 3. 再次YGC，eden + s1 -> s0
  > 4. 年龄足够 -> 老年代 （15 CMS 6）
  > 5. s区装不下 -> 老年代

* 老年代

  > 1. 顽固分子
  > 2. 老年代满了FGC Full GC

* 对象分配内存图![image-20200418093301642](C:\Users\86185\AppData\Roaming\Typora\typora-user-images\image-20200418093301642.png)



# 常见的垃圾回收器

* Serial 年轻代 串行回收

* SerialOld 

  > 优点:对于单个cpu来说执行效率高
  >
  > 缺点:在执行垃圾回收的时候,必须停止其他线程的所有工作(stop the word)

* ParNew

* CMS

* > 缺点:使用的是（Concurrent Mark Sweep 并发标记清除）当CMS存在碎片化的问题，碎片到达一定程度，CMS的老年代分配对象分配不下的时候，使用SerialOld 进行老年代回收

* ParNew

* ParallelOld

* G1

* ZGC (1ms) PK C++

* Shenandoah

<font color="red">垃圾回收器</font>

![image-20200418093425699](C:\Users\86185\AppData\Roaming\Typora\typora-user-images\image-20200418093425699.png)

# 常见垃圾回收器组合参数设定：(1.8)

> * -XX:+UseSerialGC = Serial New (DefNew) + Serial Old
>
>   * 小型程序。默认情况下不会是这种选项，HotSpot会根据计算及配置和JDK版本自动选择收集器
>* -XX:+UseParNewGC = ParNew + SerialOld
> 
>  * 这个组合已经很少用（在某些版本中已经废弃）
>   * https://stackoverflow.com/questions/34962257/why-remove-support-for-parnewserialold-anddefnewcms-in-the-future
> * -XX:+UseConc<font color=red>(urrent)</font>MarkSweepGC = ParNew + CMS + Serial Old
>* -XX:+UseParallelGC = Parallel Scavenge + Parallel Old (1.8默认) 【PS + SerialOld】
> * -XX:+UseParallelOldGC = Parallel Scavenge + Parallel Old
>* -XX:+UseG1GC = G1
> * Linux中没找到默认GC的查看方法，而windows中会打印UseParallelGC 
>
>   * java +XX:+PrintCommandLineFlags -version
>  * 通过GC的日志来分辨
> * Linux下1.8版本默认的垃圾回收器到底是什么？
>
>   * 1.8.0_181 默认（看不出来）Copy MarkCompact
>  * 1.8.0_222 默认 PS + PO

# JVM调优第一步，了解JVM常用命令行参数

* JVM的命令行参数参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

* HotSpot参数分类

  > 标准： - 开头，所有的HotSpot都支持
  >
  > 非标准：-X 开头，特定版本HotSpot支持特定命令
  >
  > 不稳定：-XX 开头，下个版本可能取消

  java -version

  java -X

  

  试验用程序：

  ```java
  import java.util.List;
  import java.util.LinkedList;
  
  public class HelloGC {
    public static void main(String[] args) {
      System.out.println("HelloGC!");
      List list = new LinkedList();
      for(;;) {
        byte[] b = new byte[1024*1024];
        list.add(b);
      }
    }
  }
  ```

  

  1. 区分概念：内存泄漏memory leak，内存溢出out of memory
  2. java -XX:+PrintCommandLineFlags HelloGC
  3. java -Xmn10M -Xms40M -Xmx60M -XX:+PrintCommandLineFlags -XX:+PrintGC  HelloGC
     PrintGCDetails PrintGCTimeStamps PrintGCCauses
  4. java -XX:+UseConcMarkSweepGC -XX:+PrintCommandLineFlags HelloGC
  5. java -XX:+PrintFlagsInitial 默认参数值
  6. java -XX:+PrintFlagsFinal 最终参数值
  7. java -XX:+PrintFlagsFinal | grep xxx 找到对应的参数
  8. java -XX:+PrintFlagsFinal -version |grep GC

# 日志详情

