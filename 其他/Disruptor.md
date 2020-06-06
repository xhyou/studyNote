# Disruptor

## 介绍

主页：http://lmax-exchange.github.io/disruptor/

源码：https://github.com/LMAX-Exchange/disruptor

GettingStarted: https://github.com/LMAX-Exchange/disruptor/wiki/Getting-Started

api: http://lmax-exchange.github.io/disruptor/docs/index.html

maven: https://mvnrepository.com/artifact/com.lmax/disruptor

## Disruptor的特点

对比ConcurrentLinkedQueue : 链表实现

JDK中没有ConcurrentArrayQueue

Disruptor是数组实现的

无锁，高并发，使用环形Buffer，直接覆盖（不用清除）旧的数据，降低GC频率

实现了基于事件的生产者消费者模式（观察者模式）

## RingBuffer

环形队列

RingBuffer的序号，指向下一个可用的元素

采用数组实现，没有首尾指针

对比ConcurrentLinkedQueue，用数组实现的速度更快

> 假如长度为8，当添加到第12个元素的时候在哪个序号上呢？用12%8决定
>
> 当Buffer被填满的时候到底是覆盖还是等待，由Producer决定
>
> 长度设为2的n次幂，利于二进制计算，例如：12%8 = 12 & (8 - 1)  pos = num & (size -1)

## Disruptor开发步骤

1. 定义Event - 队列中需要处理的元素

2. 定义Event工厂，用于填充队列

   > 这里牵扯到效率问题：disruptor初始化的时候，会调用Event工厂，对ringBuffer进行内存的提前分配
   >
   > GC产频率会降低

3. 定义EventHandler（消费者），处理容器中的元素

## 事件发布模板

## ProducerType生产者线程模式

> ProducerType有两种模式 Producer.MULTI和Producer.SINGLE
>
> 默认是MULTI，表示在多线程模式下产生sequence
>
> 如果确认是单线程生产者，那么可以指定SINGLE，效率会提升
>
> 如果是多个生产者（多线程），但模式指定为SINGLE，会出什么问题呢？

## 等待策略

1，(常用）BlockingWaitStrategy：通过线程阻塞的方式，等待生产者唤醒，被唤醒后，再循环检查依赖的sequence是否已经消费。

2，BusySpinWaitStrategy：线程一直自旋等待，可能比较耗cpu

3，LiteBlockingWaitStrategy：线程阻塞等待生产者唤醒，与BlockingWaitStrategy相比，区别在signalNeeded.getAndSet,如果两个线程同时访问一个访问waitfor,一个访问signalAll时，可以减少lock加锁次数.

4，LiteTimeoutBlockingWaitStrategy：与LiteBlockingWaitStrategy相比，设置了阻塞时间，超过时间后抛异常。

5，PhasedBackoffWaitStrategy：根据时间参数和传入的等待策略来决定使用哪种等待策略

6，TimeoutBlockingWaitStrategy：相对于BlockingWaitStrategy来说，设置了等待时间，超过后抛异常

7，（常用）YieldingWaitStrategy：尝试100次，然后Thread.yield()让出cpu

8. （常用）SleepingWaitStrategy : sleep

## 消费者异常处理

默认：disruptor.setDefaultExceptionHandler()

覆盖：disruptor.handleExceptionFor().with()

> disruptor的简单案例

1.定义一个Event事件

```java
public class StringEvent implements Runnable{

    private String name;

    public void set(String name)
    {
        this.name = name ;
    }

    @Override
    public String toString() {
        return "Event{" +
                "name='" + name + '\'' +
                '}';
    }

    @Override
    public void run() {
        System.out.println("我试试实现一个多线程...");
    }
}
```

2.定义一个EventFactory

```java
public class StringEventFactory implements EventFactory<StringEvent> {
    @Override
    public StringEvent newInstance() {
        return new StringEvent();
    }
}
```

3.定义处理类

```java
public class StringEventHandler implements EventHandler<StringEvent> {

    public static long count = 0 ;

    @Override
    public void onEvent(StringEvent stringEvent, long l, boolean b) throws Exception {
        count++;
        System.out.println(Thread.currentThread().getName()+","+stringEvent+",序号"+l);
    }
}
```

> 最简单的入门

```java
public class Main01 {
    public static void main(String[] args) {
        //创建工厂
        StringEventFactory stringEventFactory = new StringEventFactory();
        //定义缓存去大小
        int buffer = 8;
        //构造容器
        Disruptor<StringEvent> disruptor = new Disruptor(stringEventFactory, buffer, Executors.defaultThreadFactory());
        //处理规则
        disruptor.handleEventsWith(new StringEventHandler());
        //启动容器
        disruptor.start();
        //使用ringbuffer
        RingBuffer<StringEvent> ringBuffer = disruptor.getRingBuffer();
        //获取序号
        long sequence = ringBuffer.next();

        StringEvent stringEvent = ringBuffer.get(sequence);
        stringEvent.set("是小薛啊。。。");

        ringBuffer.publish(sequence);
    }
}
```

> 使用EventTranslator创建

```java
public class Main02 {
    public static void main(String[] args) {
        StringEventFactory factory=new StringEventFactory();

        int buffer = 1024;

        Disruptor<StringEvent> disruptor = new Disruptor<StringEvent>(factory,buffer, Executors.defaultThreadFactory());

        disruptor.handleEventsWith(new StringEventHandler());

        disruptor.start();

        RingBuffer<StringEvent> ringBuffer = disruptor.getRingBuffer();

        EventTranslator<StringEvent> eventTranslator = new EventTranslator<StringEvent>() {

            @Override
            public void translateTo(StringEvent event, long l) {
                System.out.println(l);
                event.set("小薛进步的第一天");
            }
        };

        ringBuffer.publishEvent(eventTranslator);

        EventTranslatorOneArg<StringEvent,Long> oneArg = new EventTranslatorOneArg<StringEvent,Long>() {
            @Override
            public void translateTo(StringEvent event, long sequence, Long o2) {
                event.set("傲雪寒梅"+o2);
            }
        };

        ringBuffer.publishEvent(oneArg,111l);

        EventTranslatorTwoArg<StringEvent,Long,String> twoArg = new EventTranslatorTwoArg<StringEvent,Long,String>() {
            @Override
            public void translateTo(StringEvent event, long sequence, Long o2,String o3) {
                event.set("傲雪寒梅"+o2+o3);
            }
        };
        ringBuffer.publishEvent(twoArg,26l,"菜鸟");

        EventTranslatorThreeArg<StringEvent,Long,String,String> threeArg = new EventTranslatorThreeArg<StringEvent,Long,String,String>() {
            @Override
            public void translateTo(StringEvent event, long l, Long aLong, String s, String s2) {
                event.set("傲雪寒梅"+aLong+s+s2);
            }
        };
        ringBuffer.publishEvent(threeArg,26l,"菜鸟","blibli");

        EventTranslatorVararg<StringEvent> vararg = new EventTranslatorVararg<StringEvent>() {
            @Override
            public void translateTo(StringEvent event, long l, Object... objects) {
                String str ="";
                for(Object obj:objects){
                    String s = (String)obj;
                    str+=s;
                }
                event.set(str);
            }
        };
        ringBuffer.publishEvent(vararg,"你","快","加","油","吧");
        disruptor.shutdown();
    }
}
```

> 使用lambda表达式创建

```java
public class Main03 {
    public static void main(String[] args) {
        int buffer = 1024;
        Disruptor<StringEvent> disruptor = new Disruptor(StringEvent::new, buffer, Executors.defaultThreadFactory());
        disruptor.handleEventsWith((event,sequence, endOfBatch)->{
            System.out.println(Thread.currentThread().getName()+","+event+",序号"+sequence);
        });
        disruptor.start();

        RingBuffer<StringEvent> ringBuffer = disruptor.getRingBuffer();
        ringBuffer.publishEvent((event,sequence,str1)->{
            event.set(str1);
        },"here");

        ringBuffer.publishEvent((event,sequence,str1)->{
            event.set(str1);
        },"here");

        ringBuffer.publishEvent((event,sequence,str1,str2)->{
            event.set(str1+str2);
        },"here","i");

        disruptor.shutdown();
    }
}
```

> 案例四:异常处理

```java
public class Main04 {
    static int num=0;
    public static void main(String[] args) throws InterruptedException {
        int buffer = 1024;
        Disruptor<StringEvent> disruptor = new Disruptor(StringEvent::new, buffer, 	Executors.defaultThreadFactory(), ProducerType.MULTI, new BlockingWaitStrategy());
        /*StringEventHandler s1 = new StringEventHandler();
        StringEventHandler s2 = new StringEventHandler();
        disruptor.handleEventsWith(s1,s2);*/
        EventHandler eventHandler =(event,sequence,flag)->{
            System.out.println("hello word");
            throw new Exception("一场梦");
        };
        disruptor.handleEventsWith(eventHandler);
        /*disruptor.handleEventsWith((event,sequence,endOfBatch)->{
            num++;
		System.out.println(Thread.currentThread().getName()+"
		,sequence:"+sequence+",endOfBatch:"+endOfBatch);
            throw new Exception("异常了");
        });*/
        disruptor.handleExceptionsFor(eventHandler).with(new ExceptionHandler<StringEvent>() {
            @Override
            public void handleEventException(Throwable throwable, long l, StringEvent o) {
                throwable.printStackTrace();
            }

            @Override
            public void handleOnStartException(Throwable throwable) {
                System.out.println("handleOnStartException");
            }

            @Override
            public void handleOnShutdownException(Throwable throwable) {
                System.out.println("handleOnShutdownException");
            }
        });
        disruptor.start();
        RingBuffer<StringEvent> ringBuffer = disruptor.getRingBuffer();
        final int threadCount = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(threadCount);
        ExecutorService service = Executors.newCachedThreadPool();
        for(int i=0;i<threadCount;i++){
            final long threadNum = i;
            service.submit(()->{
                System.out.println("生产了:"+threadCount);
                try {
                    int await = cyclicBarrier.await();
                    System.out.println(await);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
                for(int j=0;j<100;j++){
                    ringBuffer.publishEvent((event,sequence)->{
                        event.set(threadNum+"");
                        System.out.println("生产了 " + threadNum);
                    });
                }
            });
        }
        TimeUnit.SECONDS.sleep(3);
        System.out.println(StringEventHandler.count);

    }
}
```

