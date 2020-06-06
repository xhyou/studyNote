# Eureka

## Server端

> Eureka使用服务标识符(服务名称)就可以访问服务。功能类似dubbo的zookeeper(服务注册中心)

> 构建Server端Eureka的一些关键代码 

1.pom文件的添加

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

2.yml关键配置

```yml
server:
  port: 7001
eureka:
  instance:
    hostname: eureka7001.com #使用了域名映射,为了集群的时候方便 #localhost 在本机    #eureka服务端的实例名称
  client:
    register-with-eureka: false #false表示服务端注册中心注册自己。只有客户端才注册
    fetch-registry: false       #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #设置Eureka Server交互的地址查询服务与注册服务 简称:注册地址
      #defaultZone:http://${eureka.instance.hostname}:${server.port}/eureka/ 单机的时候使用
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/                   #集群配置
```

3.在启动配置文件中识别该服务是eureka的服务端

```java
@SpringBootApplication
@EnableEurekaServer  //关键代码
public class Server7001 {
    public static void main(String[] args) {
        SpringApplication.run(Server7001.class, args);
    }
}
```

4.测试此服务是否成功启动

> 浏览器访问 localhost:7001

## provider端

1.pom文件的添加

```xml
<dependency>   
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
 <!-- actuator监控信息完善 用于下面info的提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

2.yml的关键配置

```yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      #defaultZone:http://localhsot:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka/, #集群配置
                   http://eureka7002.com:7002/eureka/,
                   http://eureka7003.com:7003/eureka/
       instance:
      		instance-id: microservicecloud-dept  #自定义入驻到微服务中的服务别名
      		prefer-ip-address: true 		    #访问信息有IP的提示   		
       info: #点超链接的时候不报errorPage的提示
          	app.name: microservicecloud
          	company.name: infomation 
          	build.artifactId: $project.artifactId$  #需要配置build的pom
          	build.version: $project.version$
```

3.在启动配置文件中识别该服务是eureka的客户端

```java
@SpringBootApplication
@EnableEurekaClient //关键代码
public class Provide8001 {
    public static void main(String[] args) {
        SpringApplication.run(Provide8001.class, args);
    }
}
```

## 服务发现

> 让其他服务发现我的一些信息

主启动类添加

```java
@EnableDiscoveyClient//核心代码
@SpringBootApplication
@EnableEurekaClient 
public class Provide8001 {
    public static void main(String[] args) {
        SpringApplication.run(Provide8001.class, args);
    }
}
```

```java
@Autowired
private DiscoveryClient client; //注入发现客户端

@RequestMapping(value = "/dept/discovery", method = RequestMethod.GET)
public Object disconvery() {
    List<String> list = client.getServices(); //eureka中的服务的列表清单
    System.out.println("*********" + list);

    List<ServiceInstance> instanceList = client.getInstances("microservicecloud-dept");
    for (ServiceInstance element : instanceList) {
        System.out.println(element.getServiceId() + "\t" + element.getHost() + "\t" + 		                 element.getPort() + "\t" + element.getUri());//打印IP 主机 端口 url等
    }
    return this.client;
}
```

## Eureka和zookeeper的区别

> Eureka遵守AP     (A:可用性 P:分区容错性)
>
> zookeeper遵循CP(C:强一致性 P:分区容错性)

# Ribbon负载均衡

> NetFlix Ribbon实现的客户端的负载均衡
>
> 不用再关系服务和端口 直接访问eureka中的服务名称

1.添加Pom

```xml
<!-- Ribbon相关 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

2.配置算法

```java
    @Bean
    @LoadBalanced //负载均衡注解
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    } 
   
   //自定义负载均衡
   /* @Bean
    public IRule myRule() {
        //使用随机算法
        return new RandomRule();
    }
   */
```

3.主启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## Ribbon的策略

> IRule接口
>
> 轮询：RoundRobinRule
>
> 随机：RandomRule
>
> 重试：RetryRule  默认使用轮询，假设有其中的一个服务器挂了，那么重试几次之后跳过这个服务器
>
> ......

```java
//自定义负载均衡
@Bean
public IRule myRule() {
    //使用随机算法
    return new RandomRule();
}
```

> 自定义轮询算法

```java
主启动类中添加
@RibbonClient(name="MICROSERVICECLOUD-DEPT",configuration=MySelfRule.class)
```

```java
放在主启动扫描不到的包
@Configuration
public class ConfigBean {
     @Bean
    public IRule myRule() {
      	//可以自定义轮询 继承AbstractLoadBanlanceRule为例子
        return new RoundRobinRule();
    }
}
```

# Feign

> 作用于客户端的负载均衡 接口+注解的声明式webService端的调用

1.核心pom

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

2.定义接口

```java
@FeignClient(value = "microservicecloud-dept") #定义Feign的接口 value代表注册中心中的服务名称
public interface DeptClientService {

    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    public List<Dept> list();

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    public boolean add(Dept dept);
}
```

3.访问类中使用

```java
@RestController
public class DeptController_Consumer {
    @Autowired
    private DeptClientService service;
    
    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id) {
        return this.service.get(id);
    }
    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list() {
        return this.service.list();
    }
    @RequestMapping(value = "/consumer/dept/add")
    public Object add(Dept dept) {
        return this.service.add(dept);
    }
}
```

4.主启动类配置

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients //核心代码 
public class DeptConsumer80_Feign_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_Feign_App.class, args);
    }
}

```



# Hystrix

> 熔断器:主要负责做服务熔断和服务降级(雪崩现象) 与AOP的通知有些相似
>
> 假设多个服务相互调用 A调用B B调用C 假设A长时间挂起我们应该对其隔离和采取相对于的措施
>
> 向调用方返回一个预期可处理的备选响应

1.核心pom

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
</dependency>
```

2.yml关键配置中

```yml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
      instance-id: microservicecloud-dept-hystrix #自定义的hystrix服务名称信息
      prefer-ip-address: true 
```

3.核心代码示例

```java
@RequestMapping("/dept/get/{id}")
@HystrixCommand(fallbackMethod = "processHystrix_Get")
public Dept findByDept(@PathVariable("id") Long id) {
    Dept dept = deptDao.findById(id);
    if (null == dept) {
        throw new RuntimeException("该ID：" + id + "没有没有对应的信息");
    }
    return dept;
}

//服务熔断
public Dept processHystrix_Get(@PathVariable("id") Long id) {
    return new Dept().setDeptNo(id).setDeptName("该ID：" + id + "没有没有对应的信息,null--                                 @HystrixCommand").setDbSource("no this database in MySQL");
}
```

4.主启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker //熔断
public class DeptProvider8001_Hystrix_App {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider8001_Hystrix_App.class, args);
    }
}
```

# 服务降级

> 资源不够用了，将某些服务先关闭，待资源充足重新开启

实现方法

在接口中定义

```java
@FeignClient(value = "microservicecloud-dept", fallbackFactory =                                                           DeptClientServiceFallbackFactory.class)
public interface DeptClientService {

    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    public List<Dept> list();

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)	
    public boolean add(Dept dept);
}
```

实现fallbackFactory方法

```java
@Component
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {//当服务关闭时或者异常时,会调用服务降级方法 
            @Override
            public Dept get(long id) {
                return new Dept().setDeptNo(id).setDeptName("该" + id + "没有部门").setDbSource("no                                                               dbSource");
            }
 
            @Override
            public List<Dept> list() {
                return null;
            }

            @Override
            public boolean add(Dept dept) {
                return false;
            }
        };
    }
}
```

在pom中

```yml
feign:
  hystrix:
    enabled: true #开启服务降级
```

# Zuul(路由网关)

1.pom文件配置

```xml
 <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
 </dependency>   
```

2.yml关键配置中

```yml
server:
  port: 9527

spring:
  application:
    name: microservicecloud-zuul-gateway

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true
    
zuul:
   prefix: /atiguik # 添加统一的前缀
   ignored-services: microservicecloud-dept  # 忽略这个服务名,意思用这个名称就访问不了 如果使用*屏蔽所有
   routes:
     mydept.service: microservicecloud-dept  # 服务名称
     mydept.path : /mydept/**  				# 服务名称替换地址

info:
  app.name: atguigu-microcloud
  company.name: www.atguigu.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```

3.主启动类

```java
@SpringBootApplication
@EnableZuulProxy
public class ZuulApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class, args);
    }
}
```

