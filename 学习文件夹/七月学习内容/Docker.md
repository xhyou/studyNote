# 背景

**docker是什么？**

> docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化，容器是完全使用沙箱机制，相互之间不会有任何接口。简言之，就是可以在Linux上镜像使用的这么一个容器。

**docker可以在什么情况下使用**

> 1.web应用自动化打包发布，像tomcat应用的发布
>
> 2.自动化测试和持续集成、发布
>
> 3.在服务型环境中部署和调整数据库或其他的后台应用
>
> 4.搭建paas环境



# Docker的组成

**镜像**:就是一个只读的模板。镜像可以用来创建 Docker 容器，一个镜像可以创建很多容器。

```java
 例子:	Person p1 = new Person();
         Person p2 = new Person();
其中Person就相当于一个镜像,p1 p2就相当于容器
```

**容器**:独立运行的一个或一组应用。容器是用镜像创建的运行实例

**仓库**:是集中存放镜像文件的场所

# Docker的安装

**官方文档**：https://docs.docker.com/engine/install/centos/

```shell
#1.卸载旧版本
$ yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
#2.安装储存库
$ yum install -y yum-utils
#3.配置阿里云的镜像地址
$ yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#4.更新软件包索引(可选)
$ yum makecache fast
#5.安装最新版本的Docker Engine和容器
$ yum install docker-ce docker-ce-cli containerd.io
#6.启动docker
 systemctl start docker
#7.验证是否启动成功
 docker version
#8.测试hello word程序
 docker run hello-world
 #具体流程为:
 #在docker本机寻找是否有该镜像,如果有就运行容器实例,如果没有就上阿里云的镜像上查找
 #在阿里云镜像上查找,如果有的话就下载该镜像到本地,以该镜像为模板生产容器运行,如果没有就返回错误找不到该镜像 
#9.查看当前的镜像
 docker images
#10.卸载docker
 $ sudo yum remove docker-ce docker-ce-cli containerd.io
 $ sudo rm -rf /var/lib/docker
```

## 配置阿里云镜像加速器

**地址**:https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

![image-20200705161402896](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200705161402896.png)

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://sv01gfb1.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

# 镜像命令

**常用命令**

```shell
  docker version
  docker info 对个人信息描述
  docker --help
  docker images 列出本地的镜像
  docker images -a 列出本地所有的镜像（含中间镜像层）
  docker images -q 当前镜像的image_id
  docker images -qa
  docker images --digests 显示镜像的摘要信息
  docker images --no -trunc 显示完整的镜像信息
  docker search -s 30 tomcat 显示点赞数超过30
  docker search --no -trunc 显示完整的信息
  docker search --automated 只列出automated build类型的镜像
```

```shell
# 查询镜像 
docker images --no -trunc   	 	# 显示全镜像号
# 搜索镜像
docker search -s 30 tomcat  	 	# 收藏数大于30
docker search --no-trunc mysql   	# 显示完整信息
# 拉取镜像
docker pull mysql               	# 拉取mysql的最新镜像
docker pull mysql 5.7			 	# 拉取指定版本的镜像
# 删除镜像
 docker rmi  镜像名称 			      #删除单个镜像  默认删除最新（如果后面不跟版本号的话）
 docker rmi -f 镜像名称 		      #强制删除
 docker rmi -f 名称1 名称2		      #删除多个镜像
 docker rmi -f $(docker images -qa)  #删除全部镜像 
 # 启动镜像
 docker run -it --name '别名' 镜像名/镜像id
 # 启动守护镜像并且编写与前台交互的脚本
 docker run -d 镜像名/镜像id /bin/bash -c "while true;do echo hello docker;sleep 2;done"
```



# 容器命令

> 容器的启动依赖于镜像

```shell
# 查询容器
#OPTIONS说明（常用）:
#-a :列出当前所有正在运行的容器+历史上运行过的。
#-l :显示最近创建的容器。
#-n：显示最近n个创建的容器。
#-q :静默模式，只显示容器编号。
#--no-trunc :不截断输出
docker ps -qa 
docker ps -qa -n 2 # 展示最近创建的2个

#退出容器
exit #直接退出容器
ctrl+p+q #以后台的方式运行容器

#启动容器
docker start 容器id/名称
#重启容器
docker restart 容器id/名称
#强制停止容器 
docker kill 容器id/名称

#删除容器
docker rm 容器id/名称
#一起性强制删除多个容器
docker rm -f $(docker ps -qa)

#显示日志
#完整的日志信息
#f： 跟随最新的日志打印
#t： 是加入时间戳
#tail：tail 数字 显示最后多少条
docker logs 容器id/名称
docker logs -tf tail 3 容器id/名称

#查看容器的运行进程
docker top 容器id/名称

#查看容器的具体信息
docker inspect 容器id

#进入到后台管理(ctrl+p+q)的容器
docker exec -it 容器id /bin/bash
#或者
docker attach 容器id
#从容器内拷贝文件到主机上
docker cp 容器id:地址 主机存放的路径
docker cp 

# 例子 从docker上下载elasticsearch
docker search elasticsearch
docker pull   elasticsearch
# -e "discovery.type=single-node" 集群名字 -e ES_JAVA_OPTS="-Xms64m -Xmx512m" 设置虚拟机内存
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch
#本地测试
curl localhost:9200
#查看docker容器内的cpu状态
dockers stats
```

# 可视化界面

* portainer

```shell
#下载并且启动portainer
docker run -d -p 8888:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

访问后的地址:

![image-20200708210736430](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200708210736430.png)

# 将容器制作为镜像

```shell
docker commit -a='作者' -m='信息' 容器id 保存的新路径:版本号
docker commit -a='xuehy' -m='my first docker contains' 46cd06af01c9 opt/tomcat:1.2
```

# 容器数据卷

```shell
# 挂载容器卷
docker run -it -v /myDataHost:/myDataContains centos(镜像号)
#设置只读权限 容器中只能读取,不能写入
docker run -it -v /myDataHost:/myDataContains:ro centos(镜像号)
#具名权限
docker run -it -v mycentOs:/var/lib/mysql centos(镜像号)
#匿名权限
docker run -it -v /var/lib/mysql centos(镜像号)
```

![image-20200709234956983](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200709234956983.png)

# 挂载mysql

```shell
# 拉取镜像
# 执行镜像
[root@192 ~]# docker run -d -p 3333:3306 -v /home/mysql/con:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --name mysql be0dbf01a0f3
```

# 数据卷

> 介绍:通俗来讲就是一个容器(A)copy另外一个容器(B)的数据,如果C容器需要数据,可以从A容器上获取,实现数据的传递
>
> 实现的是挂载的容器卷之间的共享

```shell
docker run -it --name dc01 xuehy/centos
#在指定文件夹中添加touch dc01_add.txt
#ctrl+p+q 退出当前容器
docker run -it --name dc02 --volumes-from dc01 xuehy/demo
```



# DockerFile

> 在linux下的一个可执行文件,可以实现数据卷的挂载并创建出镜像

```shell
在linux下创建可执行脚本DockerFile
DockerFile的简单脚本内容
# volume test
    FROM centos
    VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
    CMD echo "finished,--------success1"
    CMD /bin/bash
执行脚本命令
docker build -f /root/myDocker/DockerFile -t xuehy/centos . 
/root/myDocker/DockerFile:dockerFile所在的目录
xuehy/centos:生成dockerFile的名称
```

**dockerFile解析**

```
1.每条保留字指令都必须是大写字母且后面要跟至少一个参数 
2.按照顺序从上到下执行 
3.#表示注释 
4.每条指令都会创建一个新的镜像层,并对镜像进行提交
```

**保留关键字**

```shell
FROM:基础镜像,当前镜像是基于哪个镜像的
MAINTAINER:镜像维护者的姓名和邮箱地址（作者和作者的邮箱）
RUN:容器构建时需要执行的额外命令（linux命令）
EXPOSE:当前容器对外暴露的端口
WORKDIR:指定在创建容器后,终端默认登录进来的工作目录
ENV:用来构建镜像过程中设置环境变量
ADD:将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和tar压缩包
COPY:类似ADD但不具备解压
VOLUME:容器数据卷
CMD:指定文件启动时要运行的命令,可以有多个cmd命令但只会有最后一个生效
ENTRYPOINT:指定文件启动时要运行的命令,多个命令追加执行
ONBUILD:当构建一个被继承的DockerFile的运行命令时,父镜像在被子继承后父镜像的onbuild被触发。
```

**案例**

> 自定义的myCentos使用我们自己的镜像具备如下
>
> ​     1.登录后默认路径
>
> ​     2.vim编辑器
>
> ​     3.查看网络配置的支持

```dockerfile
FROM centos 
MAINTAINER xuehy<604624146@qq.com>
ENV mypath /tmp
WORKDIR $mypath
RUN yum -y install vim 
RUN yum -y install net-tools
EXPOSE 80
CMD /bin/bash
```

```shell
docker build -f /root/myDocker/dockerFile -t mycentos:1.3 .
```

>    执行myCentos,查看自定义镜像是否生效,以下为截图(落脚点,vim编辑器,ifconfig）

![img](C:\Users\xuehy\AppData\Local\YNote\data\qq87321C4B5B0B36C3718EDF8C50316AF1\5e6a11bbfb4d427293cb6a68c53e3541\clipboard.png)

```shell
#查看镜像的变更历史
docker history 镜像id
```

```shell
#CMD和ENTRYPONIT的命令案例
Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被docker run 之后的参数替换
docker run 之后的参数会被当做参数传递给 ENTRYPOINT，之后形成新的命令组合
制作CMD版可以查询IP信息的容器
 FROM
 centos
 RUN
 yum install -y curl
 CMD [ "curl", "-s", "http://ip.cn" ]
 
制作ENTROYPOINT版查询IP信息的容器
FROM centos
RUN yum install -y curl
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]

curl命令可以用来执行下载、发送各种HTTP请求，指定HTTP头部等操作
```

> 案例1:Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换

使用CMD的不可以追加信息

![img](C:\Users\xuehy\AppData\Local\YNote\data\qq87321C4B5B0B36C3718EDF8C50316AF1\ce53bd8725f4461395d0a63382b24c4c\clipboard.png)

执行entrypoint可以追加信息

![img](C:\Users\xuehy\AppData\Local\YNote\data\qq87321C4B5B0B36C3718EDF8C50316AF1\531c10e12c4946b29897d67a57987e33\clipboard.png)

```shell
# ONBUILD案例讲解:
#构建两个dockerfile文案
  -父文件
  FROM centos
  RUN yum install -y curl
  ENTRYPOINT ['curl',"-s","http://ip.cn"]
  ONBUILD RUN echo 'father onbuild--886'
```

```shell
# ADD和COPY的介绍:
# 以手动创建一个tomcat为例子
综合所有的DOCKERFILE命令来说
在指定的目录以/root/myDocker/为统一路径,在下面创建一个mydockerfile文件
FROM centos
MAINTAINER xhyou<604624146@qq.com>
#把宿主机当前上下文的c.txt拷贝到容器/usr/local/路径下
#COPY只有复制,没有解压
COPY c.txt /usr/local/cincontainer.txt 
#把java与tomcat添加到容器中
#ADD是复制加解压
ADD jdk-7u79-linux-x64.gz /usr/local/
ADD apache-tomcat-7.0.70.tar.gz /usr/local/
#安装vim编辑器
RUN yum -y install vim
#设置工作访问时候的WORKDIR路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH
#配置java与tomcat环境变量
ENV JAVA_HOME /usr/local/jdk1.7.0_79
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.70
ENV CATALINA_BASE /usr/local/apache-tomcat-7.0.70
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#容器运行时监听的端口
EXPOSE 8080
#启动时运行tomcat
# ENTRYPOINT ["/usr/local/apache-tomcat-7.0.70/bin/startup.sh" ]
# CMD ["/usr/local/apache-tomcat-7.0.70/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-7.0.70/bin/startup.sh && tail -F /usr/local/apache-tomcat-7.0.70/bin/logs/catalina.out

# 生成镜像
docker bulid -f mydockerfile -t mytomcat .

# 启动并且挂载容器卷
docker run -d -p 9080:8080 --name myt9 
# 创建数据卷
-v /root/myDocker/mydockerfile/tomcat7/test:/usr/local/apache-tomcat-7.0.70/webapps/test
-v /root/myDocker/mydockerfile/tomcat7/tomcat7logs/:/usr/local/apache-tomcat-7.0.70/logs 
--privileged=true xhyou（镜像名称）
```

在数据卷下创建web.xml和a.jsp用作测试用例

```html

<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" version="2.4">  
</web-app>

JSP文件
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
  </head>
  <body>
    -----------welcome------------
    <%="i am in docker tomcat self "%>
    <br>
    <br>
    <% System.out.println("=============docker tomcat self");%>
  </body>
</html>
```

![img](C:\Users\xuehy\AppData\Local\YNote\data\qq87321C4B5B0B36C3718EDF8C50316AF1\b1c827cffd4945199e294a9d8a788233\clipboard.png)

```shell
# 修改镜像的版本号
dockers tag 镜像号 修改的镜像名称和版本号
```

# 发布镜像

> 案例以阿里云发布

> ```shell
> # 登录
> $ sudo docker login --username=用户名 registry.cn-hangzhou.aliyuncs.com
> # 推送
> $ sudo docker login --username=用户名 registry.cn-hangzhou.aliyuncs.com
> $ sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/xhyou/xhyou:[镜像版本号]
> $ sudo docker push registry.cn-hangzhou.aliyuncs.com/xhyou/xhyou:[镜像版本号]
> ```



# docker网络

![image-20200711181632921](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200711181632921.png)

**link的使用**

> 可以直接使用容器别名实现两个容器之间的互通

```shell
# tomcat01 第一个容器的别名
[root@192 xuehy]# docker run -d --name tomcat02 --link tomcat01  6055d4d564e1
```

![image-20200711191355107](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200711191355107.png)

**自定义网络**

```
查看docker的网络
docker network 
```

![image-20200712103333735](C:\Users\xuehy\AppData\Roaming\Typora\typora-user-images\image-20200712103333735.png)

> bridge :桥接
>
> none：不配置网络
>
> host：和linux共享网络
>
> container：容器网络联通！（局限大）

```shell
[root@192 ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet 
110b0b8f96eacf73f3b24a2fc1537f91e8d56c0fb175aa54bcedf492f294965b
[root@192 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
5227c5d16e2b        bridge              bridge              local
a91e9fd546ef        host                host                local
110b0b8f96ea        mynet               bridge              local
b98b4d13d6c8        none                null                local
[root@192 ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "110b0b8f96eacf73f3b24a2fc1537f91e8d56c0fb175aa54bcedf492f294965b",
        "Created": "2020-07-12T11:57:14.535230293+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
[root@192 ~]# docker exec -it  8f665e01b859 ping tomcat-net-02 
64 bytes from 8f665e01b859 (192.168.0.3): icmp_seq=1 ttl=64 time=0.024 ms
64 bytes from 8f665e01b859 (192.168.0.3): icmp_seq=2 ttl=64 time=0.037 ms
```

**网络联通**

> 两个docker之间互通就不行 案例中 tomcat03使用的是docker0网卡 ,tomcat-net-01使用的是自定义mynet网卡

```shell
[root@192 ~]# docker exec -it tomcat03 ping tomcat-net-01
ping: tomcat-net-01: Name or service not known
# 解决方案
[root@192 ~]# docker network connect mynet tomcat03
[root@192 ~]# docker inspect mynet
    "Containers": {
            "047ec36c7f5ec799f533ccc6daab945bc147534a81e212d24617bdd393638302": {
                "Name": "tomcat03", # mynet已经把tomcat加入
                "EndpointID": "7423924c336d819a659cbcf37eca7908af4ed25a3ad78b7c0bdc95e21f7a0119",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",
                "IPv6Address": ""
            },
            "369dd4a2094dd58235056aae10c39033ccc29974779dd1aa11bfd510a5a7bc9c": {
                "Name": "tomcat-net-01",
                "EndpointID": "aafd0c538345aca811acd593411c97fd5e9b1b2791d6778969756371bd799fd1",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "8f665e01b859db6aaa013b39105def59a9140a6b226cfff371a37984fc5ae892": {
                "Name": "tomcat-net-02",
                "EndpointID": "71841fe2504a867244c9969e0ed8bf4509156ec9e2b8f048d1ff4f381d24a35d",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            }
        },
PING tomcat-net-01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.087 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.048 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=3 ttl=64 time=0.047 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=4 ttl=64 time=0.049 ms
```

