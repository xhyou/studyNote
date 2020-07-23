# Dubbo

> dubbo是一个(RPC)远程调用系统,是一种远程过程调用。

***调用规则***

![image-20200601072246252](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200601072246252.png)

***RPC的框架原理图***

![image-20200601073328982](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200601073328982.png)

# 注册中心

[***1.下载Zookeeper***](https://zookeeper.apache.org/releases.html#download)

2.解压后启动zookerper的服务端

![image-20200601120234369](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200601120234369.png)

# 监控中心

[1.图形监控界面下载](https://github.com/apache/dubbo/tree/dubbo-2.6.0)

[2.tomcat下载](https://tomcat.apache.org/)

3.启动zkServer.cmd服务

4.将打包后的dubbo-admin-2.6.0.war放置tomcat的webapps目录下

5.启动tomcat(修改端口号为:9090),访问http://localhost:9090/dubbo-admin-2.6.0/

![image-20200601150315011](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200601150315011.png)

# 通用类的配置（引用它的pom位置实现公用）

```java
//DTO类
@AllArgsConstructor
@NoArgsConstructor
@ToString
@Data
public class UserAddress {
    private Integer id;
    private String userAddress;//用户地址
    private String userId;//用户id
    private String consignee;//收货人
    private String phoneNum;//电话号码
    private String isDefault;//是否为默认地址,Y-是 N-否
}
//接口
public interface UserService {
    List<UserAddress> getUserAddressList(String userId);
}
//接口
public interface OrderService {
    /**
     * 初始化订单
     * @param userId
     */
    void initOrder(String userId);
}
//接口的实现
public class UserServiceImpl implements UserService {
    /**
     * 根据用户的id查找收货地址
     * @param userId
     * @return
     */
    public List<UserAddress> getUserAddressList(String userId) {
        UserAddress address1= new UserAddress(1,"福建厦门湖里区软件园1号楼","1","咆哮的可达鸭","18559326862","Y");
        UserAddress address2= new UserAddress(2,"福建厦门湖里区内厝55号","1","咆哮的可达鸭","18559326862","N");
        return Arrays.asList(address1,address2);
    }
}
```



# 配置Dubbo的提供者

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.study.provider</groupId>
    <artifactId>Provider</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <!--引入dubbo-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.2</version>
        </dependency>
        <!-- 注册中心使用的是zookeeper，引入操作zookeeper的客户端端 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
        <!--引入父依赖 通用类的pom文件-->
        <dependency>
            <groupId>com.study.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>1.0-SNAPSHOT</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</project>
```

```xml
<!--在resource的目录下新建provider.xml配置文件类-->
<?xml version="1.0" encoding="utf-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo
       http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!--1.指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名）-->
    <!--相当于容器的初始化相当于图示中的第一个步骤-->
    <dubbo:application name="Provider"></dubbo:application>

    <!--2.指定注册中心-->
    <!--相当于图示中的第二个步骤-->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" />
    <!--<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181" />-->

    <!--3.配置协议-->
    <dubbo:protocol name="dubbo" port="20880" />

    <!--4.暴露服务 ref指向真正的实现对象-->
    <dubbo:service interface="com.gmall.service.UserService" ref="userServiceImpl"></dubbo:service>

    <!--5.服务的实现-->
    <bean id="userServiceImpl" class="com.gmall.service.impl.UserServiceImpl"></bean>
</beans>
```

```java
//启动类
public class Main {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext ac=new ClassPathXmlApplicationContext("provider.xml");
        ac.start();
        System.in.read();
    }
}
```

## 图形化界面显示

![image-20200601215540268](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200601215540268.png)

# 配置Dubbo的消费者

1.pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.study.consumer</groupId>
    <artifactId>Consumer</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <!--引入dubbo-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.2</version>
        </dependency>
        <!-- 注册中心使用的是zookeeper，引入操作zookeeper的客户端端 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
        <!--引入父依赖-->
        <dependency>
            <groupId>com.study.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>1.0-SNAPSHOT</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</project>
```

2.配置xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <context:component-scan base-package="com.consumer.service.impl"></context:component-scan>

    <!--1.指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名）-->
    <!--相当于容器的初始化相当于图示中的第一个步骤-->
    <dubbo:application name="Consumer"></dubbo:application>

    <!--2.指定注册中心 千万不要忘记2181端口的添加-->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>

    <!--调用远程服务-->
    <dubbo:reference interface="com.gmall.service.UserService" id="userService"></dubbo:reference>

</beans>
```

3.java

```java
@Service
public class OrderServiceImpl implements OrderService {

    @Autowired
    UserService userService;

    public void initOrder(String userId) {
        //查询用户的收货地址
        List<UserAddress> userAddressList = userService.getUserAddressList(userId);
        System.out.println(userAddressList);
    }
}

//启动类
public class Main {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext ac=new ClassPathXmlApplicationContext("consumer.xml");
        OrderService orderService = ac.getBean(OrderService.class);
        orderService.initOrder("1");
        System.in.read();
    }
}

```

## 图形化界面的显示

![image-20200601222357247](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200601222357247.png)

# 监控中心（配置相对简单,不懂自行百度）

1.解压dubbo-monitor-simple-2.0.0，并且修改dubbo.properties配置文件(端口之类的)

2.在comsumer.xml和provider.xml配置

```xml
	<!--配置监控中心-->
    <dubbo:monitor protocol="registry"></dubbo:monitor>
    <!--<dubbo:monitor address="127.0.0.1:7070"></dubbo:monitor>-->
```

![image-20200601231534779](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200601231534779.png)

# 整合SpringBoot

1.引用上面的通用类配置

## 配置Dubbo和提供者

1.引入pom核心文件

```xml
<dependencies>
        <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
            <scope>provided</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.alibaba.boot/dubbo-spring-boot-starter -->
        <!--dubbo整合springboot-->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.2.0</version>
        </dependency>
        <!--引入父依赖-->
        <dependency>
            <groupId>com.study.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>1.0-SNAPSHOT</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</project>
```

2.配置文件,与xml配置文件是一样的

```properties
# 服务的名称
dubbo.application.name=DubboBootProvider
# 注册中心的地址
dubbo.registry.address=127.0.0.1:2181
dubbo.registry.protocol=zookeeper
# 协议
dubbo.protocol.name=dubbo
dubbo.protocol.port=20880
# 连监控中心
dubbo.monitor.protocol=registry
## 暴露的服务在代码中解决 如下
```

3.代码

```java
import com.alibaba.dubbo.config.annotation.Service;
import com.gmall.domain.UserAddress;
import com.gmall.service.UserService;
import org.springframework.stereotype.Component;

import java.util.Arrays;
import java.util.List;

@Component
@Service //它引用的是com.alibaba.dubbo.config.annotation.Service;
public class UserServiceImpl implements UserService {
    /**
     * 根据用户的id查找收货地址
     * @param userId
     * @return
     */
    public List<UserAddress> getUserAddressList(String userId) {
        UserAddress address1= new UserAddress(1,"福建厦门湖里区软件园1号楼","1","咆哮的可达鸭","18559326862","Y");
        UserAddress address2= new UserAddress(2,"福建厦门湖里区内厝55号","1","咆哮的可达鸭","18559326862","N");
        return Arrays.asList(address1,address2);
    }
}

```

4.主启动类

```java
@EnableDubbo //开启dubbo
@SpringBootApplication
public class DubbobootApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubbobootApplication.class, args);
    }
}
```

## 配置Dubbo的消费者

1.核心pom

```xml
<dependencies> 
    <dependency>
        <groupId>com.alibaba.boot</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>0.2.0</version>
    </dependency>
    <!--引入父依赖-->
    <dependency>
        <groupId>com.study.dubbo</groupId>
        <artifactId>dubbo</artifactId>
        <version>1.0-SNAPSHOT</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

2.配置文件

```properties
# 服务的名称
dubbo.application.name=DubboBootConsumer
# 注册中心的地址 
dubbo.registry.address=zookeeper://127.0.0.1:2181
#监控中心的协议
dubbo.monitor.protocol=registry
server.port=8082

# 引用服务在代码中体现
```

3.代码

```java
package com.dubbo.boot.dubboboot.service.impl;

import com.alibaba.dubbo.config.annotation.Reference;
import com.gmall.domain.UserAddress;
import com.gmall.service.OrderService;
import com.gmall.service.UserService;
import org.springframework.stereotype.Service;
import java.util.List;
//业务层
@Service
public class OrderServiceImpl implements OrderService {

    @Reference // com.alibaba.dubbo.config.annotation.Reference; 引用服务
    UserService userService;

    public List<UserAddress> initOrder(String userId) {
        //查询用户的收货地址
        List<UserAddress> userAddressList = userService.getUserAddressList(userId);
        System.out.println(userAddressList);
        return userAddressList;
    }
}
// 控制层
@Controller
public class OrderController {

    @Autowired
    OrderService orderService;

    @RequestMapping("/initOrder")
    @ResponseBody
    public List<UserAddress> initOrders(@RequestParam("userId") String userId){
        return  orderService.initOrder(userId);
    }

}
```

4.主启动类

```java
@SpringBootApplication
@EnableDubbo //开启dubbo
public class DubboBootConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboBootConsumerApplication.class, args);
    }
}

```

# 启动检查

> 背景:如果服务启动了消费者,没有启动提供者。消费者启动时服务找不到提供者的service，启动会报错
>
> 但我们有时候真正需要的是在调用时候找不到才报错,所以我们需要关闭检查

如果是SpringBoot在消费者端关闭检查的代码为

```java
@Reference(check = false) //完整代码在上面  check = false
UserService userService;
```

如果是spring的项目在消费者的xml中配置

```xml
<!--方式一-->
<!--调用远程服务-->
<dubbo:reference interface="com.gmall.service.UserService" id="userService" check="false"></dubbo:reference>

<!--方式二-->
<!--配置消费者的统一规则 当前所有的服务都不检查-->
<dubbo:consumer check="false"></dubbo:consumer>
```

# 超时时间

> 消费中如果长时间没有访问服务提供者没有获取到值,那么就返回

如果是spring的项目在消费者的xml中配置

```xml
优先级:方法级>标签>大于全局
<!--方式一-->
<!--调用远程服务 timeout单位是毫秒 默认超时时间是1000ms-->
<dubbo:reference interface="com.gmall.service.UserService" id="userService" timeout="3000"></dubbo:reference>

<!--指定某个特定方法的超时时间-->
<dubbo:reference interface="com.gmall.service.UserService" id="userService" >
   <dubbo:method name="getUserAddressList" timeout="1000"></dubbo:method>
</dubbo:reference>

<!--方式二-->
<!--配置消费者的统一规则 当前所有的服务都不检查-->
<dubbo:consumer timeout="3000"></dubbo:consumer>
```

```xml
//特别注意
<!--服务的提供者也可配置消费的超时时间,如果是提供者和消费者都配置了超时时间,消费者的优先级高于提供者-->
<dubbo:service interface="com.gmall.service.UserService" ref="userServiceImpl" timeout="1000"></dubbo:service>

<!--统一设置服务提供方的规则-->
<dubbo:provider timeout="1000"></dubbo:provider>
```

小总结

```
优先级:
  如果是相同级别，消费者的优先级大于提供者的优先级
  如果是不同级别,精确优先
```

# 重试次数

> 背景：当我们某个服务连接超时,我们可以触发重试次数。不包含第一次次数

消费者端的xml配置

```xml
<!--调用远程服务 retries 重试次数 retries=3 -->
 <dubbo:reference interface="com.gmall.service.UserService" id="userService" retries="3" >
 </dubbo:reference>
```

> 特别注意:重试次数只针对幂等性操作,非幂等性操作的不能使用
>
> 幂等：不过执行多少次,结果还是相同  比如 查询 删除 修改
>
> 非幂等：比如新增

# 多版本

> 背景:当一个接口实现的时候,可能版本不稳定,我们先用老版本开发,待稳定后我们在过度到新版本

## 如何配置

1.在服务的提供方中,实现不同的实现类

```java
//实现类1
public class UserServiceImpl implements UserService {
    /**
     * 根据用户的id查找收货地址
     * @param userId
     * @return
     */
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("1.0.0版本.");
        UserAddress address1= new UserAddress(1,"福建厦门湖里区软件园1号楼","1","咆哮的可达鸭","18559326862","Y");
        UserAddress address2= new UserAddress(2,"福建厦门湖里区内厝55号","1","咆哮的可达鸭","18559326862","N");
        return Arrays.asList(address1,address2);
    }
}
//实现类2
public class UserServiceImpl1 implements UserService {
    /**
     * 根据用户的id查找收货地址
     * @param userId
     * @return
     */
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("2.0.0版本.");
        UserAddress address1= new UserAddress(1,"福建厦门湖里区软件园1号楼","1","咆哮的可达鸭","18559326862","Y");
        UserAddress address2= new UserAddress(2,"福建厦门湖里区内厝55号","1","咆哮的可达鸭","18559326862","N");
        return Arrays.asList(address1,address2);
    }
}
```

2.在提供者方配置

```xml
   <!--多版本的实现
     version:指定版本
     ref:引用不同的实现接口
   -->	

    <!--4.暴露服务 ref指向真正的实现对象-->
    <dubbo:service interface="com.gmall.service.UserService" ref="userServiceImpl" version="1.0.0"></dubbo:service>
    <!--5.服务的实现-->
    <bean id="userServiceImpl" class="com.gmall.service.impl.UserServiceImpl"></bean>

    <dubbo:service interface="com.gmall.service.UserService" ref="userServiceImpl1" version="2.0.0"></dubbo:service>
    <bean id="userServiceImpl1" class="com.gmall.service.impl.UserServiceImpl1"></bean>
```

3.消费者方

```xml
  <!--调用远程服务 version:指定版本-->
    <dubbo:reference interface="com.gmall.service.UserService" id="userService" version="2.0.0">
    </dubbo:reference>
```

# 本地存根

> 背景:如果本地的消费者想去调用远程的提供者,我们不想给它直接调用,需要在本地做一些校验规则,如果通过了那么在去调用远程的提供者

1.在消费者中xml中添加本地存根的标签路径

```xml
 <!--调用远程服务
   stub="com.gmall.service.impl.UserServiceSub">
  --> 
    <dubbo:reference interface="com.gmall.service.UserService" id="userService"
     version="1.0.0" stub="com.gmall.service.impl.UserServiceSub">
    </dubbo:reference>
```

```java
//配置本地存根的方法
public class UserServiceSub implements UserService {
	
    private UserService userService;
	//构造器必须,它会自动将你配置的userService传入进来
    public UserServiceSub (UserService userService){
        this.userService = userService;
    }

    @Override
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("开始走本地存根的方法。。。。");
        if(!"".equals(userId)){
            return userService.getUserAddressList(userId);
        }
        return null;
    }
}
```

# SpringBoot整合dubbo的三种方式

```java
 * SpringBoot与dubbo整合的三种方式：
 * 1）、导入dubbo-starter，在application.properties配置属性，使用@Service【暴露服务】使用@Reference【引用服务】
 * 2）、保留dubbo xml配置文件;
 * 		导入dubbo-starter，使用@ImportResource导入dubbo的配置文件即可
 * 3）、使用注解API的方式：
 * 		将每一个组件手动创建到容器中,让dubbo来扫描其他的组件
     
//@EnableDubbo //开启基于注解的dubbo功能
//@ImportResource(locations="classpath:provider.xml")
@EnableDubbo(scanBasePackages="com.provider.gmall")
@SpringBootApplication
public class DubbobootApplication {

	public static void main(String[] args) {
		SpringApplication.run(DubbobootApplication.class, args);
	}
}
```

```java
//基于配置的
@Configuration
public class MyDubboConfig {
	
	@Bean
	public ApplicationConfig applicationConfig() {
		ApplicationConfig applicationConfig = new ApplicationConfig();
		applicationConfig.setName("boot-user-service-provider");
		return applicationConfig;
	}
	
	//<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>
	@Bean
	public RegistryConfig registryConfig() {
		RegistryConfig registryConfig = new RegistryConfig();
		registryConfig.setProtocol("zookeeper");
		registryConfig.setAddress("127.0.0.1:2181");
		return registryConfig;
	}
	
	//<dubbo:protocol name="dubbo" port="20882"></dubbo:protocol>
	@Bean
	public ProtocolConfig protocolConfig() {
		ProtocolConfig protocolConfig = new ProtocolConfig();
		protocolConfig.setName("dubbo");
		protocolConfig.setPort(20882);
		return protocolConfig;
	}
	
	/**
	 *<dubbo:service interface="com.atguigu.gmall.service.UserService" 
		ref="userServiceImpl01" timeout="1000" version="1.0.0">
		<dubbo:method name="getUserAddressList" timeout="1000"></dubbo:method>
	</dubbo:service>
	 */
	@Bean
	public ServiceConfig<UserService> userServiceConfig(UserService userService){
		ServiceConfig<UserService> serviceConfig = new ServiceConfig<>();
		serviceConfig.setInterface(UserService.class);
		serviceConfig.setRef(userService);
		serviceConfig.setVersion("1.0.0");
		
		//配置每一个method的信息
		MethodConfig methodConfig = new MethodConfig();
		methodConfig.setName("getUserAddressList");
		methodConfig.setTimeout(1000);
		
		//将method的设置关联到service配置中
		List<MethodConfig> methods = new ArrayList<>();
		methods.add(methodConfig);
		serviceConfig.setMethods(methods);
		
		//ProviderConfig
		//MonitorConfig
		
		return serviceConfig;
	}

}
```

# 高可用

```java
//dubbo直连 如果采取这种方式,即使zookeeper挂掉了 依然还是可以连接上
 @Reference(url = "127.0.0.1:20882")
 UserService userService;
```

# 负载均衡

> 1.dubbo默认采用的是随机的策略

```java
   @Reference(loadbalance = "roundrobin")
   UserService userService;
```

> 在负载均衡中不仅可以设置负载均衡的策略,还可以通过全权重去设置.设置权重主要有两种方式
>
> 第一种在监控中心设置
>
> 第二种在提供者的@Service中设置

![image-20200602124009172](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200602124009172.png)

```java
@Component
@Service(weight = 100) //设置权重
public class UserServiceImpl implements UserService {}
```

# 服务降级

> 当服务器压力剧增的情况下,根据实际的业务情况，对一些服务和页面有策略的处理或者不处理，从而保证核心业务的高效运行

> 两种方法:
>
>    第一种：消费方直接返回为null,不发起远程调用
>
>    第二种: 当我们调取失败的时候,返回为空(如超时)

方法一：

![image-20200602124419992](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200602124419992.png)

方法二:

![image-20200602124654600](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200602124654600.png)

# 集群容错

> 当我们调用提供者的时候,出现错误，给与相对应的处理

## 配置Dubbo的提供者

1.导入hystrix

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>
        spring-cloud-starter-netflix-hystrix
    </artifactId>
</dependency>
```

2.处理代码

```java
 @HystrixCommand //配置容错的注解
    public List<UserAddress> getUserAddressList(String userId) {
        UserAddress address1= new UserAddress(1,"福建厦门湖里区软件园1号楼","1","咆哮的可达鸭","18559326862","Y");
        UserAddress address2= new UserAddress(2,"福建厦门湖里区内厝55号","1","咆哮的可达鸭","18559326862","N");
        if(Math.random()>0.5){
            throw  new  RuntimeException();
        }
        return Arrays.asList(address1,address2);
    }
```

3.在主启动类开启注解

```java
@EnableDubbo
@EnableHystrix
@SpringBootApplication
public class DubbobootApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubbobootApplication.class, args);
    }
}
```

## 配置Dubbo的消费者

1.导入pom文件

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>
        spring-cloud-starter-netflix-hystrix
    </artifactId>
</dependency>
```

2.java代码

```java
@Service
public class OrderServiceImpl implements OrderService {

    @Reference(loadbalance = "roundrobin")
    UserService userService;

    @HystrixCommand(fallbackMethod = "hello")
    public List<UserAddress> initOrder(String userId) {
        //查询用户的收货地址
        List<UserAddress> userAddressList = userService.getUserAddressList(userId);
        System.out.println(userAddressList);
        return userAddressList;
    }

    public List<UserAddress> hello(String userId) {
        //查询用户的收货地址
        return Arrays.asList(new UserAddress(3,"福建厦门湖里区软件园1号楼","1","服务容错","18559326862","Y")) ;
    }
}

```

3.启动类配置

```java
@SpringBootApplication
@EnableHystrix
@EnableDubbo
public class DubboBootConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(DubboBootConsumerApplication.class, args);
    }

}
```

# Dubbo的原理（未完）

> DubboBeanDefinitionParser 类解析器
>
> parse方法解析标签:解析每一个标签

```java
//解析每一个标签
if (ProtocolConfig.class.equals(beanClass)) {

}else if (ServiceBean.class.equals(beanClass)) {

}else if (ProviderConfig.class.equals(beanClass)) {

}else if (ConsumerConfig.class.equals(beanClass)) {

}

//标签的名字是通过给名称空间的解析器获取的
public class DubboNamespaceHandler extends NamespaceHandlerSupport {

    static {
        Version.checkDuplicate(DubboNamespaceHandler.class);
    }

    @Override
    public void init() {
        registerBeanDefinitionParser("application", new DubboBeanDefinitionParser(ApplicationConfig.class, true));
        registerBeanDefinitionParser("module", new DubboBeanDefinitionParser(ModuleConfig.class, true));
        registerBeanDefinitionParser("registry", new DubboBeanDefinitionParser(RegistryConfig.class, true));
        registerBeanDefinitionParser("monitor", new DubboBeanDefinitionParser(MonitorConfig.class, true));
        registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
        registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
        registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
        registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));
        registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, false));
        registerBeanDefinitionParser("annotation", new AnnotationBeanDefinitionParser());
    }
}


//着重注意这个
registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean InitializingBean.class, true));
//在配置加载完之后,会去加载ServiceBean的InitializingBean中的afterPropertiesSet 
//和ApplicationListener 的 onApplicationEvent方法
```

