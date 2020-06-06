# JAVA中的引用类型



## 强引用

  Java中默认声明的就是强引用 

```java
Object obj = new Object(); //只要obj还指向Object对象，Object对象就不会被回收
obj = null;  //手动置null
```

##  软引用 

软引用是用来描述一些非必需但仍有用的对象。**在内存足够的时候，软引用对象不会被回收，只有在内存不足时，系统则会回收软引用对象，如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常**。这种特性常常被用来实现缓存技术，比如网页缓存，图片缓存等。 

下面以一个例子来进一步说明强引用和软引用的区别：
在运行下面的Java代码之前，需要先配置参数 -Xms2M -Xmx3M，将 JVM 的初始内存设为2M，最大可用内存为 3M。

首先先来测试一下强引用，在限制了 JVM 内存的前提下，下面的代码运行正常

```java
public class TestOOM {
	
	public static void main(String[] args) {
		 testStrongReference();
	}
	private static void testStrongReference() {
		// 当 new byte为 1M 时，程序运行正常
		byte[] buff = new byte[1024 * 1024 * 1];
	}
}
```

 但是如果我们将 

```java
 byte[] buff = new byte[1024 * 1024 * 1]; 
```

 替换为创建一个大小为 2M 的字节数组 

```java
byte[] buff = new byte[1024 * 1024 * 2];
```

 则内存不够使用，程序直接报错，强引用并不会被回收 

 ![img](https://img2018.cnblogs.com/blog/662236/201809/662236-20180922194052676-1646914311.png) 

 接着来看一下软引用会有什么不一样，在下面的示例中连续创建了 10 个大小为 1M 的字节数组，并赋值给了软引用，然后循环遍历将这些对象打印出来。 

```java
public class TestOOM {
	private static List<Object> list = new ArrayList<>();
	public static void main(String[] args) {
	     testSoftReference();
	}
	private static void testSoftReference() {
		for (int i = 0; i < 10; i++) {
			byte[] buff = new byte[1024 * 1024];
			SoftReference<byte[]> sr = new SoftReference<>(buff);
			list.add(sr);
		}
		
		System.gc(); //主动通知垃圾回收
		
		for(int i=0; i < list.size(); i++){
			Object obj = ((SoftReference) list.get(i)).get();
			System.out.println(obj);
		}
	}
}
```

 打印结果：

  ![img](https://img2018.cnblogs.com/blog/662236/201809/662236-20180922194016719-117632363.png) 

我们发现无论循环创建多少个软引用对象，打印结果总是只有最后一个对象被保留，其他的obj全都被置空回收了。
这里就说明了在内存不足的情况下，软引用将会被自动回收。
值得注意的一点 , 即使有 byte[] buff 引用指向对象, 且 buff 是一个strong reference, 但是 SoftReference sr 指向的对象仍然被回收了，这是因为Java的编译器发现了在之后的代码中, buff 已经没有被使用了, 所以自动进行了优化。
如果我们将上面示例稍微修改一下：

```java
private static void testSoftReference() {
		byte[] buff = null;

		for (int i = 0; i < 10; i++) {
			buff = new byte[1024 * 1024];
			SoftReference<byte[]> sr = new SoftReference<>(buff);
			list.add(sr);
		}

        System.gc(); //主动通知垃圾回收
		
		for(int i=0; i < list.size(); i++){
			Object obj = ((SoftReference) list.get(i)).get();
			System.out.println(obj);
		}

		System.out.println("buff: " + buff.toString());
	}
```

  则 buff 会因为强引用的存在，而无法被垃圾回收，从而抛出OOM的错误。 

 ![img](https://img2018.cnblogs.com/blog/662236/201809/662236-20180922194030314-105853688.png) 

如果一个对象惟一剩下的引用是软引用，那么该对象是软可及的（softly reachable）。垃圾收集器并不像其收集弱可及的对象一样尽量地收集软可及的对象，相反，它只在真正 “需要” 内存时才收集软可及的对象。 

## 弱引用

弱引用的引用强度比软引用要更弱一些，**无论内存是否足够，只要 JVM 开始进行垃圾回收，那些被弱引用关联的对象都会被回收**。在 JDK1.2 之后，用 java.lang.ref.WeakReference 来表示弱引用。
我们以与软引用同样的方式来测试一下弱引用： 

```java
private static void testWeakReference() {
		for (int i = 0; i < 10; i++) {
			byte[] buff = new byte[1024 * 1024];
			WeakReference<byte[]> sr = new WeakReference<>(buff);
			list.add(sr);
		}
		
		System.gc(); //主动通知垃圾回收
		
		for(int i=0; i < list.size(); i++){
			Object obj = ((WeakReference) list.get(i)).get();
			System.out.println(obj);
		}
	}
```

 打印结果： 

  ![img](https://img2018.cnblogs.com/blog/662236/201809/662236-20180922194112309-477100844.png) 

 可以发现所有被弱引用关联的对象都被垃圾回收了 

## 虚引用

虚引用是最弱的一种引用关系，如果一个对象仅持有虚引用，那么它就和没有任何引用一样，它随时可能会被回收，在 JDK1.2 之后，用 PhantomReference 类来表示，通过查看这个类的源码，发现它只有一个构造函数和一个 get() 方法，而且它的 get() 方法仅仅是返回一个null，也就是说将永远无法通过虚引用来获取对象，虚引用必须要和 ReferenceQueue 引用队列一起使用。 

```java
public class PhantomReference<T> extends Reference<T> {
    /**
     * Returns this reference object's referent.  Because the referent of a
     * phantom reference is always inaccessible, this method always returns
     * <code>null</code>.
     *
     * @return  <code>null</code>
     */
    public T get() {
        return null;
    }
    public PhantomReference(T referent, ReferenceQueue<? super T> q) {
        super(referent, q);
    }
}
```

# ThreadLocal

线程本地存储

每个线程的内部,有存放线程的本地变量，这个变量是这个线程专享。

使用get/set去获取数据,当set数据的时候，我们会将当前的值在Thread这个类中,去构造一个map

```java
public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

## 内存泄露

ThreadLocal的底层使用的是虚引用,会用Entry去存储数据,当某个线程的key为空时只是移除了entry当中的key,其value的指向消失,但是value其还在内存中存在。就会造成内存泄露

使用remove()这个方法,移除整个Entry

```java
  Entry(ThreadLocal<?> k, Object v) {
      super(k);
      value = v;
  }
```

