启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

是SpringBoot的启动场景

比如spring-boot-starter-web他就会帮我们导入web环境下的所有依赖

# 主程序

SpringBootApplication标注这个类是一个SpringBoot的启动类

```java
@SpringBootApplication
public class SpringBootDemoApplication {
	public static void main(String[] args) {
        //将这个类启动
		SpringApplication.run(SpringBootDemoApplication.class, args);
	}
}
```

```
扫描类
@ComponentScan

@SpringBootConfiguration springBoot的配置
  @Configuration 是一个spring的配置注解
    @Component是一个组件

@EnableAutoConfiguration 自动配置
  @AutoConfigurationPackage 自动配置包
    @Import(AutoConfigurationPackages.Registrar.class) 自动配置包注册
      metadata 导入元数据
  @Import(AutoConfigurationImportSelector.class) 导入选择器
```

从Application或者yml中获取信息

```yml
person:
  name: xue
  age: 27
  height: 172
```

```java
@Data
@ToString
@Configuration
@ConfigurationProperties(value = "person")
public class Person {
    private String name;
    private String age;
    private String height;
}
```

JSR-303

```java
@Data
@ToString
@Validated //添加jsr-303校验
@Configuration
public class Person {
	@Email(message="邮箱格式不正确")
    private String name;
    private String age;
    private String height;
}
```

# 静态资源的获取

> 源码位置 WebMvcAutoConfiguration -》addResourceHandlers

* SpringBoot使用以下的方式处理静态资源
* webjars localhost:8080/webjars/
* public ,static ,/**,resources

优先级:resources>static(默认)>public

![image-20200428222818890](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200428222818890.png)

# 定制首页

相关源码

> WebMvcAutoConfiguration -》WelcomePageHandlerMapping

![image-20200428224721370](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200428224721370.png)

存放位置

![image-20200428224808897](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200428224808897.png)

# Thymeleaf

1.引入pom包

2.在页面中引入thymeleft的语言包

```html
<html xmlns:th="http://www.thymeleaf.org">
```

3.使用

```java
 @RequestMapping(value="demo")
    public String demo(Model model){
        model.addAttribute("msg","hello,word"); 
        return "modules/demo"; //当浏览器输入/index时，会返回 /templates/home.html页面
    }
```

```html
<div data-th-text="${msg}"></div>
//或者
<div th:text="${msg}"></div>
```

## 语法

[使用手册]: https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#a-multi-language-welcome	"thymeleft官网"

简单表达式：

- 变量表达式： `${...}`
- 选择变量表达式： `*{...}`
- 消息表达： `#{...}`
- 链接URL表达式： `@{...}`
- 片段表达式： `~{...}`

# jdbc

## 问题:安装mysql8版本密码重置问题

### 方法一

> 1.确认sql目录下有没有**data（Data）**文件夹，如果有就**删掉**。(我的是解压版本的)

![image-20200528164614084](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528164614084.png)

> 2.然后在cmd输入**mysqld --initialize**，等待一段时间（这段时间就是在创建data（Data）文件夹），然后就再次输入**net start mysql**便可，mysql服务器会启动，这个之后 重新登录时会产生一个随机密码,先**关闭cmd**。

> 3.随后以**管理员身份**运行 cmd,我们先找到随机密码，可以**搜索.err**文件（找文件目录**:\**mysql\bin\data\ .err** 文件 ，或寻找你的安装按目录），貌似里面就这一个.err的文件,用记事本打开，直接查找password，后面就是随机密码。

![image-20200528164713070](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200528164713070.png)

> 二、修改密码
> 	   1.以管理员身份打开cmd
>
> 2.进入数据库：mysql -u root -p然后会提示输入密码,输入上面找到的密码。
>
> 3.数据库修改密码：ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';  

[博客链接](https://blog.csdn.net/liu_xin_xin/article/details/96473805)



### 方法二

> 1、打开cmd，输入：net stop mysql，停止Mysql的服务；
>
> 2、跳过密码验证登录Mysql服务
>
> 输入命令：mysqld --console --skip-grant-tables --shared-memory 
>
> 3、打开一个新的cmd，输入登录命令：mysql -u root -p，在要求输入密码的地方直接回车；
>
> 4、设置密码为空
>
> 输入命令：
>
> use mysql
>
> update user set authentication_string='' where user='root';
>
> 5、输入quit，退出Mysql；
>
> 6、关闭之前打开的第一个cmd；
>
> 7、在第二个打开的cmd中，启动Mysql服务
>
> 输入命令：net start mysql 
>
> 8、步骤4已经把密码置空，所以直接输入命令：mysql -u root -p登录；
>
> 9、利用命令修改密码
>
> ```
> 输入命令：ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';  
>  ```
> 
>10、验证更改后的密码
> 
>输入quit，退出当前Mysql服务，输入登录命令：mysql -u root -p
> 
>输入密码，成功登录

## 使用Navicat Premium 12链接出现[Authentication plugin 'caching_sha2_password' cannot be loaded]问题

```
1. 管理员权限运行命令提示符，登陆MySQL（记得添加环境变量）

   mysql -u root -p

   password:        
```

![image-20200501165851746](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200501165851746.png)

```
2. 修改账户密码加密规则并更新用户密码

 ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;#修改加密规则 

 ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';#更新一下用户的密码 
```

![image-20200501165944044](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200501165944044.png)

```
3. 刷新权限并重置密码

   FLUSH PRIVILEGES;   #刷新权限 

 单独重置密码命令：alter user 'root'@'localhost' identified by '111111';
```

# SpringBoot配置druid的sql可视化界面

界面展示

![image-20200502084918389](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200502084918389.png)

步骤

> 1.数据库的连接方式改为druid
>
> 2.配置文件中添加druid的连接信息
>
>    spring.datasource.filters = stat,wall,slf4j

1.pom文件

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.21</version>
</dependency>
```

配置文件

```properties
# 数据库访问配置
# 主数据源，默认的
spring.datasource.type= com.alibaba.druid.pool.DruidDataSource 
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/demo?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
spring.datasource.username=root
spring.datasource.password=root

# 下面为连接池的补充设置，应用到上面所有数据源中
# 初始化大小，最小，最大
spring.datasource.initialSize=5
spring.datasource.minIdle=5
spring.datasource.maxActive=20
## 配置获取连接等待超时的时间
spring.datasource.maxWait=60000
## 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.datasource.timeBetweenEvictionRunsMillis=60000
## 配置一个连接在池中最小生存的时间，单位是毫秒
spring.datasource.minEvictableIdleTimeMillis=300000
spring.datasource.validationQuery=SELECT 1 FROM DUAL
spring.datasource.testWhileIdle=true
spring.datasource.testOnBorrow=false
spring.datasource.testOnReturn=false
## 打开PSCache，并且指定每个连接上PSCache的大小
spring.datasource.poolPreparedStatements=true
spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
## 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
## 注意如果是需要配置log4j 需要引入pom文件 否则启动会报错误
spring.datasource.filters=stat,wall  
## 通过connectProperties属性来打开mergeSql功能；慢SQL记录
spring.datasource.connectionProperties=stat.mergeSql=true;stat.slowSqlMillis=5000
```

核心代码

```java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }

    //后台监控
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean<StatViewServlet> bean = new ServletRegistrationBean<>(new StatViewServlet(), "/druid/*");

        HashMap<String,String> initParams = new HashMap<>();
        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");
//        initParams.put("deny","localhost");
        bean.setInitParameters(initParams);
        return bean;
    }

    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        HashMap<String,String> initParams = new HashMap<>();
        //这些东西不进行过滤
        initParams.put("exclusions","*.js,*.css,/druid/*");
        bean.setInitParameters(initParams);
        return bean;
    }
}
```

# SpringSecurity

> springBoot封装的一种安全框架,实现需要继承WebSecurityConfigurerAdapter去实现响应功能,
>
> 简单的写了一个小demo

```java
@Configuration
public class MySecurityConfig extends WebSecurityConfigurerAdapter {

    //请求授权的规则
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("/emps").hasRole("admin").and().formLogin();
        //修改默认登录页面
        http.formLogin().loginPage("/index");
        //防止跨站攻击
        http.csrf().disable();
        //记住我。
        http.rememberMe();
        //注销成功
        http.logout().logoutSuccessUrl("/");
    }

    //认证登录
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("admin").password(new BCryptPasswordEncoder().encode("123456"))
                .roles("admin");
    }
}
```

# Swagger

> 描述:Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务.

作用

> 接口文档的在线自动生成

界面展示

![image-20200504102616576](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200504102616576.png)

导入核心pom文件

```xml
 <!--导入swagger的相关组件-->
        <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
```

配置核心文件

```java
@Configuration
@EnableSwagger2 //开启Swagger2的注解
public class MySwaggerConfig {

    @Bean
    public Docket docket(){
        Docket docket = new Docket(DocumentationType.SWAGGER_2)
                        .apiInfo(apiInfo())
                        .groupName("xhyou")
                        .select()
                        .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
                        .paths(PathSelectors.any())
                        .build();
        return  docket;

    }

    public ApiInfo apiInfo(){
        //配置个人标题的信息
        return new ApiInfoBuilder()
                  .contact(new Contact("xhyou","http://www.google.com","604624146@qq.com"))
                  .licenseUrl("http://www.google.com")
                  .title("xhyou的ApiInfo信息")
                  .description("用restful风格写接口")
                  .termsOfServiceUrl("http://www.baidu.com")
                  .version("v1.0")
                  .build();
    }

}

```

# Aynsc异步

主配置类中开启

```java
@EnableAsync //开启异步任务
```

demo演示

```java
@Service
public class AsyncService {

    @Async
    public void asyncHello() throws InterruptedException {
        System.out.println("先睡3S");
        Thread.sleep(3000);
    }
}

```

```java
@RestController
public class AsyncController {

    @Autowired
    AsyncService asyncService;

    @GetMapping("/asyncHello")
    public String asyncHello() throws InterruptedException {
        asyncService.asyncHello();
        return "OK"; //页面不需要等待3s直接打印OK
    }

}
```

# Scheduling

异步任务

开启异步任务

```java
@EnableScheduling //开启定时任务
```

```java
@Controller
public class ScheduleController {
    @Scheduled(cron ="0/2 * * * * ?" ) //每2S执行一次
    public void getString(){
        System.out.println("hello");
    }
}
```

# 自定义异常

页面:自定义异常页面 404 500 error

自定义拦截类并且处理拦截请求到指定的页面

```java
//拦截所有标注有controllerd的页面
@ControllerAdvice
public class HandlerExceptionController {
    //获取日志
    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @ExceptionHandler(Exception.class)
    public ModelAndView exceptionHandler(HttpServletRequest request,Exception e) throws Exception{
        //{}相当与占位符 会将后面的信息填充到里面去
        logger.error("Request URL:{},Exception :{}",request.getRequestURL(),e);
        //当有些异常标注了状态码的话,就直接返回
        if(AnnotationUtils.findAnnotation(e.getClass(), ResponseStatus.class)!=null){
            //将异常抛出,让SpringBoot处理
            throw e;
        }
        ModelAndView mv = new ModelAndView();
        mv.addObject("url",request.getRequestURL());
        mv.addObject("exception",e);
        mv.setViewName("error/error");
        return mv;
    }
}
```

自定义error页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>错误</h1>
    <div>
        <!--获取的异常信息在源代码中有，界面中不显示-->
        <div th:utext="'&lt;!--'" th:remove="tag"></div>
        <div th:utext="'Failed Request URL : ' + ${url}" th:remove="tag"></div>
        <div th:utext="'Exception message : ' + ${exception.message}" th:remove="tag"></div>
        <ul th:remove="tag">
            <li th:each="st : ${exception.stackTrace}" th:remove="tag"><span th:utext="${st}" th:remove="tag"></span></li>
        </ul>
        <div th:utext="'--&gt;'" th:remove="tag"></div>
    </div>
</body>
</html>
```

自定义异常类,当页面找不到时候跳转404页面 (书写的位置为templates/error)

```java
@Controller
public class IndexController {
    @GetMapping("/index")
    public String index(){
//        int i = 9/0;
        String blog =null;
        if(blog==null){
            throw new NotFoundException("blog not found");
        }
        return "index";
    }
}
```

```java
//定义状态码
@ResponseStatus(HttpStatus.NOT_FOUND)
public class NotFoundException extends RuntimeException {

      public NotFoundException(){
          
      }

      public NotFoundException(String message){
          super(message);
      }

      public NotFoundException(String message,Throwable cause){
         super(message,cause);
      }
}

```

# 日志处理

```java
@Aspect
@Component
public class LogAspect {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Pointcut("execution(* com.blog.blogProduct.controller.*.*(..))")
    public void log(){
    }
	
    //获取日志的请求地址 ip方法名和请求的参数
    @Before("log()")
    public void doBefore(JoinPoint joinPoint){
        //记录请求的日志
        ServletRequestAttributes attributes = (ServletRequestAttributes) 																   RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        String url = request.getRequestURL().toString();
        String ip = request.getRemoteAddr();
        String classMethod = joinPoint.getSignature().getDeclaringTypeName() + "." +      				   joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        RequestLog requestLog = new RequestLog(url,ip,classMethod,args);
        logger.info("Request:{}",requestLog);
    }

    @After("log()")
    public void doAfter(){
    }

    @AfterReturning(returning = "result",pointcut = "log()")
    public void doAfterReturning(Object result){
        logger.info("Result：{}",result);
    }

    @AllArgsConstructor
    @ToString
    private class RequestLog{
        private String url;
        private String ip;
        private String classMethod;
        private Object[] ags;
    }
}
```

结果显示

```java
2020-05-08 23:21:34.916  INFO 11804 --- [nio-8080-exec-5] com.blog.blogProduct.aspect.LogAspect    : Request:LogAspect.RequestLog(url=http://127.0.0.1:8080/favicon.ico, ip=127.0.0.1, classMethod=com.blog.blogProduct.controller.IndexController.index, ags=[])
```

