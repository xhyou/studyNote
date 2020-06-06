# MQ的概念

## 为什么要使用MQ

> 在日常中我们多台服务器同时处理一个请求时,服务器处理不过来 其他进来的请求处于一个等待的时候,MQ的引入有此而来

## 使用MQ的好处

> 1.系统解耦
>
>    采用中间件之后,如果在系统A上去调用了B C D 三个服务,B下面的服务A就不用去管了；
>
> 2.异步调用
>
>    当一个请求需要经过三个系统的时候,A和B系统的处理时间为3S,C系统的处理时间为30S，整个系统就会因为C系统而导致的非常的慢。
>
> 比如我们在美团或者饿了吗下单，那么在下单的时候，后台需要通过后台处理：订单支
>
> 付->账户扣款->创建订单->通知商家准备菜品->安排骑手
>
> 这个时候我们对于扣款和创建订单来讲对于时效性要求强，处理效率也比较高。对于通
>
> 知商家准备菜品和安排骑手来讲时效性的要求不是很高，处理时间也可能会稍长。这个
>
> 时候就可以使用 mq 进行异步处理，先处理完前面的业务，然后反馈给用户，之后再通
>
> 过 mq 处理通知商家准备菜品和安排骑手
>
> 3.流量削峰
>
>  当每秒有上千个请求的时候,我们可以将服务放到MQ中去等待,一个一个的慢慢的处理和消费

# MQ的安装

[1.下载官网](http://activemq.apache.org/)

2.下载linux的安装包,放置linux的/opt目录下

3.解压找到/bin目录启动

![image-20200522150346699](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200522150346699.png)

```bash
-  ./activemq start   启动
-  ./activemq stop    停止
-  ./activemq restart 重新启动
-  ./activemq status  查看启动的状态
```

4.查看是否启动

```bash
1.[root@bogon bin]# netstat -anp|grep 61616
tcp6       0      0 :::61616                :::*                    LISTEN      1820/java 
2.[root@bogon bin]# ps -ef|grep activemq|grep -v grep
root       1820      1  0 14:32 pts/0    00:00:09 /u
```

# Window下显示MQ页面

```bash
1.关闭linux的防火墙(centos7)
  systemctl stop firewalld.service
  1_1.查看状态是否关闭
      firewall-cmd --state 
      systemctl status firewalld.service
2.关闭window防火墙
3.浏览器访问linux下的地址+8161端口
```

![image-20200522154031207](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200522154031207.png)

# 队列

## 小总结

> 1.每个消息只有一个消费者
>
> 2.消费的生产者和消费者之间没有时间上的相关性。无论消费者在生产者发送消息的时候是否处于运行状态,消费者都可以提取得到信息。
>
> 3.消息被消费者消费之后,队列中不会在存储，所以消费者不会消费到已经消费得信息

## 消息生产者编码

- 引入pom文件

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-activemq</artifactId>
  </dependency>
  ```

- 编码开发

  ```java
  @SpringBootTest
  class ActivemqApplicationTests {
  
      public static final String QUEEN_ADDRESS="tcp://192.168.234.128:61616";
      public static final String QUEEN_NAME="myQueen";
      //创建工厂,引入地址(linux下的地址和端口)
      ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(QUEEN_ADDRESS);
  
      @Test
      void contextLoads() throws JMSException {
          //创建工厂连接
          Connection connection = factory.createConnection();
          //开启
          connection.start();
          //创建连接会话 第一个参数为事务,第二个参数为签收规则
          Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
          //创建消息队列
          Queue queue = session.createQueue(QUEEN_NAME);
          //使用队列创建生产者
          MessageProducer producer = session.createProducer(queue);
          TextMessage textMesasage = session.createTextMessage("这是我的第一条MQ队列");
          //通过produce发送消息给MQ
          producer.send(textMesasage);
          //关闭资源
          producer.close();
          session.close();
          connection.close();
          System.out.println("====消息发送完毕====");
      }
  
  }
  ```

- 生产这发送完毕在客户端查看

![生成者的队列](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200523112057210.png)

## 消费者消费编码

```java
 @Test
    void consumer() throws JMSException{
        Connection connection = factory.createConnection();
        connection.start();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue(QUEEN_NAME);
        //创建了一个消费者
        MessageConsumer consumer = session.createConsumer(queue);
        //同步阻塞方式(receive)
        //订阅者或者接收者调用MessageConsumer的receive()方法来接受消息,receive方法能够在接收消息之前(或者)
        //将一直阻塞
        while(true){
            TextMessage receive = (TextMessage)consumer.receive();
            // TextMessage receive = (TextMessage)consumer.receive(4000L); 4S还没有接收到消息断开连接
            if(null!=receive){
                System.out.println("接受到信息:"+receive.getText());
            }else{
                break;
            }
        }
        //方法二:通过监听的方式
        /* consumer.setMessageListener(new MessageListener() {
            @Override
            public void onMessage(Message message) {
                if(message!=null && message instanceof TextMessage){
                    try {
                        System.out.println("消费了:"+((TextMessage) message).getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
            }
        });*/
        //如果不加,会导致还来不及消费程序就已经被关闭了
	   //System.in.read();
        consumer.close();
        session.close();
        connection.close();
    }
```

- 客户端展示

![消费者队列](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200523113634267.png)

> 注意:
>
> 1.如果先创建消费者(2个),在创建生产者,那么者两个消费者一人消费一半生产的内容
>
> 2.如果先创建生产者,在创建A消费者,在创建B消费者.那么A消费者消费全部内容,B消费者无消费内容 

# 主题

## 小总结

> 1.生产者将消息发布到topic中,每个消息有多个消费者,属于1：N得关系
>
> 2.生产者和消费者之间有时间上得相关性,订阅某一个主题的消费者只能消费它订阅之后发布的消息
>
> 3.生产者生产的时候,topic不保存消息它是无状态的不落地,假如没有订阅此消息那么它就是一条废消息,所以一般是==先启动消费者==在启动生产者

## 通用代码

- 通用代码类

```java
public class CommonCode {

    public static final String MQ_URL ="tcp://192.168.234.128:61616";
    static ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(MQ_URL);
    static Connection connection =null;
    static Session session= null;

    public static Session getSession() throws JMSException {
        connection = factory.createConnection();
        connection.start();
        session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        return session;
    }

    public static void trunOffCon() throws JMSException {
        if(session!=null){
            session.close();
        }
        if(connection!=null) {
            connection.close();
        }
    }
}
```

## 消费者编程

```java
public class ConsumerTopic {
    static String TOPIC_NAME ="topic001";

    public static void main(String[] args) throws JMSException, IOException {
        Session session = CommonCode.getSession();
        Topic topic = session.createTopic(TOPIC_NAME);
        MessageConsumer consumer = session.createConsumer(topic);
        consumer.setMessageListener(message -> {
            if(message!=null && message instanceof  TextMessage){
                TextMessage receive = (TextMessage) message;
                System.out.println(receive);
            }
        });
        System.in.read();
        //关闭所有连接
        consumer.close();
        CommonCode.trunOffCon();
    }
}
```

![image-20200523174743710](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200523174743710.png)

## 生产者编程

```java
public class ProductTopic{

    static String TOPIC_NAME = "topic001";

    public static void main(String[] args) throws JMSException {
        Session session = CommonCode.getSession();
        Topic topic = session.createTopic(TOPIC_NAME);
        MessageProducer producer = session.createProducer(topic);
        TextMessage textMessage = session.createTextMessage("我是生产者,我开始生产数据");
        producer.send(textMessage);
        producer.close();
        CommonCode.trunOffCon();
    }
}
```

![image-20200523174934226](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200523174934226.png)

# 消息的可靠性

- 持久化
- 非持久化

```java
MessageProducer producer = session.createProducer(topic);
TextMessage textMessage = session.createTextMessage("我是生产者,我开始生产数据");
producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT); //设置消息不是持久化,那么服务器宕机后,消息获取不到
```

- 服务器正常生产者开启

![image-20200524123403252](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200524123403252.png)

- 服务器关闭之后在开启

  ![image-20200524123554159](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200524123554159.png)

- 服务器重新开启(此时我们生产者设置的是持久化)

  ![image-20200524123719955](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200524123719955.png)

- 消费者消费

  ![image-20200524123847254](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200524123847254.png)

# 消息持久化

客户端代码

```java
public class ConsumerTopic {
    public static final String MQ_URL ="tcp://192.168.234.128:61616";
    public static final ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(MQ_URL);
    public static final String TOPIC_NAME = "topic002";

    public static void main(String[] args) throws JMSException {
        Connection connection = factory.createConnection();
        //设置客户端的ID
        connection.setClientID("z4");
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        TopicSubscriber subscriber = session.createDurableSubscriber(topic, "xhy");
        //启动连接移到这里来了
        connection.start();
        Message message = subscriber.receive();
        while(message!=null){
            TextMessage textMessage =(TextMessage)message;
            System.out.println("**收到持久化的消息**:"+textMessage.getText());
            message = subscriber.receive(1000L);
        }
        session.close();
        subscriber.close();
        connection.close();
    }
}
```

生产者代码

```java
public class ProductTopic {

    public static final String MQ_URL ="tcp://192.168.234.128:61616";
    public static final ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(MQ_URL);
    public static final String TOPIC_NAME = "topic002";

    public static void main(String[] args) throws JMSException {
        Connection connection = factory.createConnection();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        MessageProducer producer = session.createProducer(topic);
        //开启持久化 默认就是持久化
        producer.setDeliveryMode(DeliveryMode.PERSISTENT);
        //启动连接移到这里来了
        connection.start();
        for(int i=0;i<3;i++) {
            TextMessage textMessage = session.createTextMessage("message-persist");
            producer.send(textMessage);
        }
        //关闭
        producer.close();
        session.close();
        connection.close();
    }
}
```

- 浏览器端展示(消费者开启,生产者还没有开始时候)

  ![image-20200524134125195](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200524134125195.png)

  ![image-20200524134254632](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200524134254632.png)

- 当开启生产者之后

  ![image-20200524134414879](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200524134414879.png)

![image-20200524134449148](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200524134449148.png)

# 事务

```java
 //创建连接会话
//connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
//第一个参数是设置是否开启事务,如果是true那么就是开启事务后续需要自己自动提交 session.commit (使用场景如在try()catch()中)
//如果设置的是false,那么就是每次会自动提交
//事务一般使用在生产者中
Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
```

# 签收

```java
//一般使用在生产者中
 Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
//核心代码
 TextMessage message = (TextMessage) consumer.receive();
 if (null != message) {
    System.out.println("接受到信息:" + message.getText());
    //手动签收
    message.acknowledge();
 }
```

> ==着重注意==以下几种情况:
>
> 1.当事务设置为true时,写不写手动签收都无所谓
>
>    connection.createSession(true, Session.CLIENT_ACKNOWLEDGE);
>
> 2.当事务设置为false时,你即使写了message.acknowledge();也会导致重复签收

![image-20200524153405403](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200524153405403.png)

# 不同配置启动MQ

> 按照不同的conf配置文件来启动activemq

```bash
[root@192 bin]
# ./activemq start xbean:file:/activeMQ/apache-activemq-5.15.12/conf/activemq.xml
```

# Broker

> 简介:Broker是ActiveMQ的一个简易实现，我们只需要在代码中启动Broker（用跑代码的方式启动ActiveMQ），从而实现嵌入式的ACtiveMQ

## 代码

```java
public class BrokerConfig {
    public static void main(String[] args) throws Exception {
        BrokerService brokerService = new BrokerService();
        brokerService.setUseJmx(true);
        brokerService.addConnector("tcp://localhost:61616");
        brokerService.start();
    }
}
```

# Spring整合MQ

1.pom文件

```xml
<dependencies>
        <!--activemq所需要的jar包-->
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-all</artifactId>
            <version>5.12.0</version>
        </dependency>
        <!--  activemq 和 spring 整合的基础包 -->
        <dependency>
            <groupId>org.apache.xbean</groupId>
            <artifactId>xbean-spring</artifactId>
        </dependency>
        <!--  嵌入式activemq的broker所需要的依赖包   -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.10.1</version>
        </dependency>
        <!-- activemq连接池 -->
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-pool</artifactId>
            <version>5.12.0</version>
        </dependency>
        <!-- spring支持jms的包 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jms</artifactId>
            <version>5.2.1.RELEASE</version>
        </dependency>
        <!--spring相关依赖包-->
        <dependency>
            <groupId>org.apache.xbean</groupId>
            <artifactId>xbean-spring</artifactId>
            <version>4.15</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>4.3.23.RELEASE</version>
        </dependency>
        <!-- Spring核心依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>4.3.23.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.23.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>4.3.23.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>4.3.23.RELEASE</version>
        </dependency>
    </dependencies>
```

2.配置xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启包的自动扫描-->
    <context:component-scan base-package="com.example.mq"/>

    <!--配置生产者-->
    <bean id="connectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
        <property name="connectionFactory">
            <!--正真可以生产Connection的ConnectionFactory,由对应的JMS服务商提供-->
            <bean class="org.apache.activemq.spring.ActiveMQConnectionFactory">
                <property name="brokerURL" value="tcp://192.168.234.128:61616"/>
            </bean>
        </property>
        <property name="maxConnections" value="100"/>
    </bean>

    <!--队列目的地,点对点的Queue-->
    <bean id="destinationQueue" class="org.apache.activemq.command.ActiveMQQueue">
        <!--通过构造注入Queue名-->
        <constructor-arg index="0" value="spring-active-queue"/>
    </bean>

    <!--队列目的地,布订阅的主题Topic-->
    <bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg index="0" value="spring-active-topic"/>
    </bean>

    <!--Spring提供的JMS工具类,可以进行消息发送,接收等 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <!--传入连接工厂-->
        <property name="connectionFactory" ref="connectionFactory"/>
        <!--传入目的地-->
        <property name="defaultDestination" ref="destinationTopic"/>
        <!--消息自动转换器-->
        <property name="messageConverter">
            <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
        </property>
    </bean>

    <!-- 消息监听容器，属性中引用的对象要和生产者的一致 -->
    <bean  id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory" />
        <property name="destination" ref="destinationTopic" />
        <property name="messageListener" ref="myMessageListener" />
    </bean>
</beans>
```

3.程序的使用

```java
//生产者
@Service
public class ProductMQ {

    @Autowired
    private JmsTemplate jmsTemplate;

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        ProductMQ productMQ = (ProductMQ) context.getBean("productMQ");
        productMQ.jmsTemplate.send(new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                TextMessage textMessage = session.createTextMessage("我是生产者。。。。");
                return textMessage;
            }
        });
        System.out.println("发送完成......");
    }
}
//消费者
@Service
public class ConsumerMQ {

    @Autowired
    private JmsTemplate jmsTemplate;

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        ConsumerMQ consumerMQ = (ConsumerMQ) context.getBean("consumerMQ");
//        while(!"".equals( (String) consumerMQ.jmsTemplate.receiveAndConvert())) {
        	//这种情况下只是消费一条
            String message = (String) consumerMQ.jmsTemplate.receiveAndConvert();
            System.out.println(message);
//        }
    }
}

//不使用消费者 可以开启监听
@Component
public class MyMessageListener implements MessageListener {

    public void onMessage(Message message) {
        if(null!=message && message instanceof TextMessage){
            TextMessage textMessage =(TextMessage)message;
            try {
                System.out.println("======"+textMessage.getText());
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}

```

# SpringBoot整合MQ

1.核心pom

```xml
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
```

2.配置文件

```properties
spring.activemq.broker-url=tcp://192.168.234.128:61616
spring.activemq.user=admin
spring.activemq.password=admin
spring.jms.pub-sub-domain=false
# spring.jms.pub-sub-domain=true 代表设置的主题模式
#自定义队列名称
myQueen=boot-active-mq
myTopic=boot-topic
```

3.编码

```java
//主启动类
@SpringBootApplication
@EnableJms
@EnableScheduling
public class ActivemqApplication {

    public static void main(String[] args) {
        SpringApplication.run(ActivemqApplication.class, args);
    }
}

//消息队列
@Configuration
public class ConfigBean {

    //设置队列
    @Value("${myQueen}")
    private String muQueen;
    
    //设置主题
    @Value("${myTopic}")
    private String myTopic;

    //设置当前模式为队列
    @Bean
    public Queue queue(){
        return new ActiveMQQueue(muQueen);
    }

    //设计当前模式为主题
    @Bean
    public Topic topic(){
        return new ActiveMQTopic(myTopic);
    }
}

//生产者
@Component
public class ProducerMQ {

    @Resource
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Autowired
    private Queue queue;
    @Autowired
    private Topic topic;

    public void produceMsg(){
        jmsMessagingTemplate.convertAndSend(queue,"hello producer");
    }

    @Scheduled(fixedDelay = 3000)
    public void produceMsgSend(){
        jmsMessagingTemplate.convertAndSend(queue,"hello producer");
    }

    @Scheduled(fixedDelay = 3000)
    public void produceMsgTopicSend(){
        System.out.println("执行了。。。。。");
        jmsMessagingTemplate.convertAndSend(topic,"topic");
    }

}

//消费者
@Component
public class Consumer {

    @JmsListener(destination = "${myQueen}")
    public void receiveMsg(TextMessage textMessage) throws JMSException {
        System.out.println("消费者收到的队列为"+textMessage.getText());
    }

    @JmsListener(destination = "${myTopic}")
    public void receiveTopic1(TextMessage textMessage) throws JMSException {
        System.out.println("消费者收到的主题为11:"+textMessage.getText());
    }

    @JmsListener(destination = "${myTopic}")
    public void receiveTopic2(TextMessage textMessage) throws JMSException {
        System.out.println("消费者收到的主题为22:"+textMessage.getText());
    }
}

//测试类
@SpringBootTest(classes = ActivemqApplication.class)
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
class ActivemqApplicationTests {

    @Resource
    private ProducerMQ producerMQ;

	//测试队列的,如果要测试定时任务启动的,需要启动主启动类
    @Test
    void produceMsg() {
        producerMQ.produceMsg();
    }
}
```

# 传输协议

> ActiveMQ支持的client-broker通讯协议有：TVP、NIO、UDP、SSL、Http(s)、VM。其中配置Transport Connector的文件在ActiveMQ安装目录的conf/activemq.xml中的<transportConnectors>标签之内。
>
> activemq传输协议的官方文档：http://activemq.apache.org/configuring-version-5-transports.html

## 手动配置nio协议

> 找到activemq.xml文件 在文件的<transportConnectors>标签中添加协议

```xml
<transportConnectors>
  <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
  <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
  <transportConnector name="nio" uri="nio://0.0.0.0:61618 maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
</transportConnectors>
```

## NIO协议增强

> auto	: 针对所有的协议，他会识别我们是什么协议。

```xml
<transportConnector name="auto+nio" uri="auto+nio://0.0.0.0:61608?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600&amp;org.apache.activemq.transport.nio.SelectorManager.corePoolSize=20&amp;org.apache.activemq.transport.nio.Se1ectorManager.maximumPoo1Size=50"/>
```

# 消息的存储和持久化

>  持久化机制：kahaDB、levelDB、JDBC、AMQ

## 使用的机制

### kahaDB

> 自MQ5.4之后默认是用的持久化机制是KahaDB,我们可以通过配置文件读取

![image-20200527141927662](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200527141927662.png)

![image-20200527142105716](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200527142105716.png)

#### 文件类型说明：

>  db-1.log:储存消息预定义大小的数据记录文件,存储的文件名为db-<Number>.log。当数据文件已满时,一个新的文件会随之创建,number也会随之递增,它随着消息数量的增多,如每32M一个文件,文件名按照数字进行编号,如db-1.log、db-2.log。当不在引用到数据文件的任何消息的时候,文件会被删除或者归档

> db.data 该文件包含了持久化的Btree索引,索引了消息数据记录中的消息,它是消息的索引文件，本质上是B-tree.使用B-tree作为索引指向db<Number>.log里面的存储信息

> db.redo用来进行消息的恢复,如果KahaDB消息存储强制退出后启动,可以恢复Btree的索引

> lock文件锁,表示当前获得kahadb读写的broker

> db.free 当前db.data文件里哪些页面时空闲的,文件具体内容时所有的空闲的Id

### LevelDB

> 如何使用levelDB

```xml
配置文件activemq.xml中，如下
   <persistenceAdapter>
         <levelDB directory="${activemq.data}/kahadb"/>
   </persistenceAdapter>
```

### JdbcDB

1.配置

```bash
#1. 将mysql的驱动包放置到指定的目录下
[root@localhost opt]# mv mysql-connector-java-5.1.26.jar /activeMQ/apache-activemq-5.15.12/lib/
#2. 修改activemq.xml的配置文件
<!--  
<persistenceAdapter>
            <kahaDB directory="${activemq.data}/kahadb"/>
      </persistenceAdapter>
-->
<persistenceAdapter>  
      <jdbcPersistenceAdapter dataSource="#mysql-ds" createTableOnStartup="true"/> 
</persistenceAdapter>
# 参数说明：dataSource指引用持久化数据库bean的名称 
# createTableOnStartup是在启动的时候创建数据库表,默认值是true,这样每次启动都会去创建数据库表了 一般是第一次启动设置为true后之后改为false
#3.在activemq.xml的</broker>标签和<import>标签之间插入数据库连接池配置
   <bean id="mysql-ds" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://192.168.43.127:3306/activemq?relaxAutoCommit=true&amp;serverTimezone=UTC"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
        <property name="poolPreparedStatements" value="true"/>
    </bean> 
```

![image-20200528172946077](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528172946077.png)

#### 排坑指南

> 问题描述:启动MQ程序报Could not get JDBC connection: Cannot create PoolableConnectionFactory （前提:已经引入了mysql的驱动包）
>
> 解决:因为引入的驱动包的版本为5.1的,而我window下的mysql版本是8.0+的,导致不兼容,重新引入8.0+的驱动包

![image-20200528173421872](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528173421872.png)

> 问题描述1：Cannot create PoolableConnectionFactory (The server time zone value '?й???????' is unrecognized or represents more than one time zone
>
> 问题描述2: 对实体 "serverTimezone" 的引用必须以 ';' 分隔符结尾
>
> 解决1:时区问题,在url后拼接serverTimezone=UTC
>
> 解决2: xml 中 &符号是作为实体字符形式存在的，实体字符,将&换为\&amp;

#### 对列模式（数据库中的消息表）

> 1.队列模式:开启生产者,但是==不开启消费==监听,此时我们可以看到activemq_msgs会有信息

![image-20200528175117510](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528175117510.png)

#### 注意

> 如果没有开启持久化,即使没有消费监听也不会保存到表中
>
> ```java
> //关闭持久化
> jmsTemplate.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
> //this is only used when "isExplicitQosEnabled" equals "true".
> //仅仅在isExplicitQosEnabled=true的时候使用
> jmsTemplate.setExplicitQosEnabled(true);
> ```

#### topic模式（数据库中的消息表）

```java
//消费者端
public class ConsumerTopic {
    public static final String MQ_URL ="tcp://192.168.234.128:61616";
    public static final ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(MQ_URL);
    public static final String TOPIC_NAME = "topic002";

    public static void main(String[] args) throws JMSException {
        Connection connection = factory.createConnection();
        //设置客户端的ID
        connection.setClientID("z5");
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        TopicSubscriber subscriber = session.createDurableSubscriber(topic, "xhy");
        //TODO
        connection.start();
        Message message = subscriber.receive();
        while(message!=null){
            TextMessage textMessage =(TextMessage)message;
            System.out.println("**收到持久化的消息**:"+textMessage.getText());
            message = subscriber.receive(1000L);
        }
        session.close();
        subscriber.close();
        connection.close();
    }
}

//生产者端
public class ProductTopic {

    public static final String MQ_URL ="tcp://192.168.234.128:61616";
    public static final ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(MQ_URL);
    public static final String TOPIC_NAME = "topic002";

    public static void main(String[] args) throws JMSException {
        Connection connection = factory.createConnection();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic(TOPIC_NAME);
        MessageProducer producer = session.createProducer(topic);
        //不开启不会保存到activemq_msgs中
        producer.setDeliveryMode(DeliveryMode.PERSISTENT);
        //启动连接移到这里来了
        connection.start();
        for(int i=0;i<3;i++) {
            TextMessage textMessage = session.createTextMessage("message-persist");
            producer.send(textMessage);
        }
        //关闭
        producer.close();
        session.close();
        connection.close();
    }
}
```

![image-20200528201250030](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528201250030.png)

![image-20200528201324078](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528201324078.png)

### JdbcDB的高速缓存

![image-20200528201633367](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528201633367.png)

> 注意:这样配置就会生产之后,会将数据分到一个缓存的管道中,如果长时间没有消费才会到数据库中。

# ActiveMQ的多集群搭建

待完成

# 高级特性

## 异步投递

> 概念:ActiveMQ支持同步、异步两种方式将消息发送到broker,模式的选择对发送延时有巨大的影响。使用异步消息可以提高性能
>
> MQ默认使用异步发送的模式,除非明确指明==使用同步发送==或者==未使用事务的前提下发送持久化==的消息,这两种都是同步的
>
> ==缺点==:可能会有少量的数据丢失

![image-20200528221347383](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528221347383.png)

代码实现:

```java
public class CommonCode {

    //public static final String MQ_URL ="tcp://192.168.234.128:61616?jms.useAsyncSend=true"; 方式一
    public static final String MQ_URL ="tcp://192.168.234.128:61616";
    static ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(MQ_URL);
    static Connection connection =null;
    static Session session= null;

    public static Session getSession() throws JMSException {
        //        factory.setUseAsyncSend(true); 方式二
        factory.setUseAsyncSend(true);
        connection = factory.createConnection();
        connection.start();
        session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        return session;
    }

    public static void trunOffCon() throws JMSException {
        if(session!=null){
            session.close();
        }
        if(connection!=null) {
            connection.close();
        }
    }
}

public class ProductTopic{

    static String TOPIC_NAME = "topic001";
    public static final String QUEEN_NAME = "queen_001";

    public static void main(String[] args) throws JMSException {
        Session session = CommonCode.getSession();
        Queue queue = session.createQueue(QUEEN_NAME);
        //创建生成这使用ActiveMQMessageProducer接口,为了使用AsyncCallback接口
        ActiveMQMessageProducer producer = (ActiveMQMessageProducer) session.createProducer(queue);
        TextMessage textMessage = session.createTextMessage("我是生产者,我开始生产数据");
        //增加一个消息头标识
        textMessage.setJMSMessageID(UUID.randomUUID().toString()+"x");
        String jmsMessageID = textMessage.getJMSMessageID();
        //使用异步方法,判断是否成功返回
        producer.send(textMessage, new AsyncCallback() {
            @Override
            public void onSuccess() {
                System.out.println("send ok "+jmsMessageID);
            }

            @Override
            public void onException(JMSException e) {
                System.out.println("send fail "+jmsMessageID);
            }
        });
        producer.close();
        CommonCode.trunOffCon();
    }
}
```

![image-20200528224201335](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528224201335.png)

![image-20200528224208549](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528224208549.png)

## 延时投递和定时投递

```xml
//在启动标签中开启延时投递 schedulerSupport="true"
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" dataDirectory="${activemq.data}" schedulerSupport="true">
</broker>
```

代码

```java
public class ProductTopic{

    static String TOPIC_NAME = "topic001";
    public static final String QUEEN_NAME = "queen_001";

    public static void main(String[] args) throws JMSException {
        Session session = CommonCode.getSession();
        Queue queue = session.createQueue(QUEEN_NAME);
        //创建生成这使用ActiveMQMessageProducer接口,为了使用AsyncCallback接口
        ActiveMQMessageProducer producer = (ActiveMQMessageProducer) session.createProducer(queue);
        TextMessage textMessage = session.createTextMessage("我是生产者,我开始生产数据");
        //增加一个消息头标识
        long delay=1*1000;//延时时间
        long period=5*1000;//间隔时间
        int repeat =3;//重复次数
        textMessage.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY,delay); 
        textMessage.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD,period);
        textMessage.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT,repeat);
        //使用异步方法,判断是否成功返回
        producer.send(textMessage);
        producer.close();
        CommonCode.trunOffCon();
    }
}
```

## 消息消费的重试机制

> 是什么？
>
> 消费者收到消息，之后出现异常了，没有告诉broker确认收到该消息，broker会尝试再将该消息发送给消费者。尝试n次，如果消费者还是没有确认收到该消息，那么该消息将被放到死信队列重，之后broker不会再将该消息发送给消费者。

> 具体哪些情况会引发消息的重发呢？
>
> 1.Client用了transactions且在session中调用了rollback
>
> 2.Client用了transactions且在调用commit之前关闭或者没有commit
>
> 3.Client再CLIENT_ACKNOWLEDGE的传递模式下，session中调用了recover
>
> 重发的次数:
>
>  默认是间隔1s,次数6次

> 有毒消息Poison ACK 
>
> 一个消息被redelivedred超过默认的最大重发次数（默认6次）时，消费的回个MQ发一个“poison ack”表示这个消息有毒，告诉broker不要再发了。这个时候broker会把这个消息放到DLQ（死信队列）。

![img](file:///C:\Users\xuehy\AppData\Local\Temp\ksohtml14980\wps1.jpg)

验证死信队列

```java
public class CommonCode {

    public static final String MQ_URL ="tcp://192.168.234.128:61616";
    static ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(MQ_URL);
    static Connection connection =null;
    static Session session= null;

    public static Session getSession() throws JMSException {
        /**
           修改重试次数为3
            RedeliveryPolicy redeliveryPolicy = new RedeliveryPolicy();
            redeliveryPolicy.setMaximumRedeliveries(3);
            factory.setRedeliveryPolicy(redeliveryPolicy);
        **/
        connection = factory.createConnection();
        connection.start();
        //设置为手动提交事务
        session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
        return session;
    }

    public static void trunOffCon() throws JMSException {
        if(session!=null){
            session.close();
        }
        if(connection!=null) {
            connection.close();
        }
    }
}

//生产者
public class ProductTopic{

    static String TOPIC_NAME = "topic001";
    public static final String QUEEN_NAME = "queen_001";

    public static void main(String[] args) throws JMSException {
        Session session = CommonCode.getSession();
        Queue queue = session.createQueue(QUEEN_NAME);
        //创建生成这使用ActiveMQMessageProducer接口,为了使用AsyncCallback接口
        ActiveMQMessageProducer producer = (ActiveMQMessageProducer) session.createProducer(queue);
        TextMessage textMessage = session.createTextMessage("我是生产者,我开始生产数据");
        producer.send(textMessage);
        session.commit();
        producer.close();
        CommonCode.trunOffCon();
    }
}

//消费者  
public class ConsumerTopic {
    static String TOPIC_NAME ="topic001";
    public static final String QUEEN_NAME = "queen_001";

    public static void main(String[] args) throws JMSException, IOException {
        Session session = CommonCode.getSession();
        Queue queue = session.createQueue(QUEEN_NAME);
        MessageConsumer consumer = session.createConsumer(queue);
        consumer.setMessageListener(message -> {
            if(message!=null && message instanceof  TextMessage){
                TextMessage receive = (TextMessage) message;
                try {
                    System.out.println(receive.getText());
                    //session.commit(); 不提交 读取6次后 信息进入死信队列
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });
        System.in.read();
        //关闭所有连接
        consumer.close();
        CommonCode.trunOffCon();
    }
}
```

![image-20200529000849237](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200529000849237.png)

## 死信队列的配置

**1.** ***\*sharedDeadLetterStrategy\****

不管是queue还是topic，失败的消息都放到这个队列中。下面修改activemq.xml的配置，可以达到修改队列的名字

![img](file:///C:\Users\xuehy\AppData\Local\Temp\ksohtml14980\wps2.jpg)

**1.** ***\*individualDeadLetterStrategy\****

可以为queue和topic单独指定两个死信队列。还可以为某个话题，单独指定一个死信队列。

![img](file:///C:\Users\xuehy\AppData\Local\Temp\ksohtml14980\wps3.jpg)

![img](file:///C:\Users\xuehy\AppData\Local\Temp\ksohtml14980\wps4.jpg)

**1.** ***\*自动删除过期消息\****

过期消息是值生产者指定的过期时间，超过这个时间的消息

![img](file:///C:\Users\xuehy\AppData\Local\Temp\ksohtml14980\wps5.jpg)

**1.** ***\*存放非持久消息到死信队列中\****

![img](file:///C:\Users\xuehy\AppData\Local\Temp\ksohtml14980\wps6.jpg)

## 消息不被重复消费，幂等性

> 如何保证消息不被重复消费呢？幕等性问题你谈谈
>
> 造成的原因:
>
>   网络延时传输,会造成进行MQ的重试中,可能造成重复消费。
>
>   **解决** :
>
>    1.如果消息是做数据库的插入操作,给这个消息一个唯一主键,那么就算重复消费的情况下，会导致主键冲突，避免数据库的脏数据
>
> ​    2.可以使用redis,记录key-value,消费前去redis中查询是否有消费记录,有的话就说明重复消费