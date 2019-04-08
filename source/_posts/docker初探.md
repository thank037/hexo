title: Docker初探
date: 2016-12-31 12:39:04
categories: Docker
tags:
  - Docker
---


[TOC]

### 前言

学习环境是本地虚拟机, linux版本使用CentOS 7. <!-- more -->

由于使用最小安装方式, 在学习Docker之前, 顺带总结了下jdk安装和防火墙相关配置.


### 一: jdk安装

>  在Centos下安装jdk有多种方式, 例如手动下载tar文件解压, yum安装, rpm安装等. 

这里我用yum方式安装:

* ```yum list installed |grep java``` 

   查看系统中是否已经安装jdk, 如果有安装想卸载: ```yum remove xxx```即可.


* ```yum list java-1.8.0*``` 或 ```yum search java|grep jdk``` 

   查看yum库中的jdk安装包.

* ```yum -y install java-1.8.0-openjdk ``` 

   这里我安装jdk1.8

* ```/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64```

   安装完成后的默认目录

* 验证java命令就可以了.

**注意**: 这种yum方式安装的是openjdk, 并不是sun jdk. 如果想要sunjdk就用安装方式.

因为这种方式使用alternatives进行版本控制, 所以在没有设置环境变量的情况下也可以执行java命令.

但是对于tomcat或其他软件要使用jdk的话就要设置环境变量了.

对于环境变量设置: 修改```/etc/profile```文件
```
#set java environment
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

让修改生效```source /etc/profile```



### 二: 防火墙配置

CentOS 7中默认使用firewall作为防火墙, 而6.x中是iptables. 

CentOS 7下防火墙的配置, 也支持iptables防火墙. 前提是需要关闭禁止firewall然后安装iptables-services. 

>  二者不能共用.
>
> CentOS7下Firewall防火墙配置用法详解(推荐): https://yq.aliyun.com/ziliao/94786

我觉得既然CentOS 7 中升级到了firewall防火墙, 就没必要切换iptables. 不如就用firewall.

关于firewall的配置, 首先来看两个目录:

* 系统配置目录: ```/usr/lib/firewalld/services```

  目录中存放定义好的网络服务和端口参数，系统参数，不能修改

* 用户配置目录: ```/etc/firewalld/```

#### 自定义添加端口

   firewall支持两种自定义添加的方式: 命令添加和配置文件添加. 

   其中, 命令添加方式也会在配置文件中体现

* **命令添加方式**: 

  ```shell
  firewall-cmd --permanent --add-port=8050/tcp 

  # 指定zone: firewall中有Zone的概念, 可以将具体的端口指定到具体的zone配置文件中.
  firewall-cmd --zone=public --permanent --add-port=8050/tcp
  ```

  ```shell
  # 参数介绍:
  1. firwall-cmd：是Linux提供的操作firewall的一个工具;
  2. --permanent：表示设置为持久;
  3. --add-port：标识添加的端口;
  4. --zone=public：指定的zone为public;
  ```

  如果命令指定`–zone=dmz`, 会在dmz.xml文件中新增一条. 默认`zone=public`

* **配置文件添加方式**:

   ```/etc/firewalld/zones```目录下添加xml配置, 如public.xml:

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <zone>
     <short>Public</short>
     <description>xxx</description>
     <service name="dhcpv6-client"/>
     <service name="ssh"/>
     <port protocol="tcp" port="8050"/>
     <port protocol="tcp" port="8761"/>
   </zone>
   ```


#### 常用命令

**firewall**

```shell
# 查看firewall服务状态:
systemctl status firewalld

# 查看firewall的运行状态(running or not running):
firewall-cmd --state

# 查看防火墙规则:
firewall-cmd --list-all 

# 查看已开放的端口(--zone可以不写, 默认为public)
firewall-cmd --zone=public --list-ports
```

```shell
# 重启, 开启, 关闭
service firewalld restart 
service firewalld start 
service firewalld stop 
# or
systemctl [stop/stop/restart] firewalld
```

**补充下iptables的基本使用吧**

```shell
# 查看防火墙启动状态
service iptables status

# 关闭防火墙
service  iptables stop

# 启动防火墙
service  iptables start

# 查看规则列表
iptables --list-rules


# 编辑规则
vim /etc/sysconfig/iptables
# 加入一行
-A INPUT -p tcp -m state --state NEW -m tcp --dport 25 -j ACCEPT

# 保存，重启防火墙
service iptables restart
```



### 三: docker

#### 安装方式一

   ```
# docker安装: 
yum install -y docker-io
   ```
这里docker也可以, 网上有说centos7以上用docker, 我这里用docker-io也没问题

```shell
# 启动
systemctl start docker

# 设置开机启动
systemctl enable docker
```

```shell
# 查看docker信息
docker info 

# 查看版本信息
docker version
```



#### 安装方式二

> 上面介绍了一键的安装方式, 这里推荐使用官方的安装方式

官方文档: https://docs.docker.com/install/linux/docker-ce/centos/

##### 卸载

```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

```shell
$ sudo rm -rf /var/lib/docker
```

##### 安装

- 安装相关工具类

  ```shell
  $ sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
  ```

- 配置docker仓库

  ```shell
  $ sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo
  ```

- 安装docker

  ```shell
  $ sudo yum install docker-ce
  ```

在第二步`配置docker仓库`错误:

```
Loaded plugins: fastestmirror
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
Could not fetch/save url https://download.docker.com/linux/centos/docker-ce.repo to file /etc/yum.repos.d/docker-ce.repo
: [Errno 14] curl#35 - "TCP connection reset by pe
```

这是由于国内访问不到docker官方镜像的缘故, 可以通过aliyun的源来完成

```shell
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```



#### 镜像

在线拉取: http://hub.docker.com 中查找需要的镜像

或```docker search xxx```命令查找docker.io上的镜像

- 例如Mysql镜像:  使用```docker pull mysql``` , 会默认从docker.io中拉取.

> Tip: docker.io太慢可以用国内的镜像仓库```hub.daocloud.io```, 或使用daocloud加速器



离线安装: 拷贝tar包至文件中: `docker load <./xxx.tar`

列出当前镜像: `docker images`

打包镜像: `docker save >./mysql123.tar docker`



#### 容器

```docker create```: image(unioned read file system)  -> container(unioned RW file system)
容器创建. 在镜像创建一个容器后加了一层读写层



```docker start```: 在容器(container)之上启动一个可使用的进程空间, 执行后会进入Created状态

```shell
docker create -p 3308:3306 --name mysql2 -e MYSQL_ROOT_PASSWORD=root docker.io/mysql:5.7.14
docker start [container id]
```



```docker run```来启动容器

该命令实际包含两个命令: ```docker create```和```docker start```

```shell
# 例如启动两个mysql
docker run -d -p 3306:3306 --name mysql1 -e MYSQL_ROOT_PASSWORD=root docker.io/mysql:5.7.14
docker run -d -p 3308:3306 --name mysql2 -e MYSQL_ROOT_PASSWORD=root docker.io/mysql:5.7.14
```

参数解释:

> -d: 后台运行
> -p: 对外暴露的端口
> -e: 环境参数
> --name: 指定容器名(不指定的话系统随机分配name名)




```shell
# 查看容器(加-a表示包括运行和非运行所有状态的)
docker ps -a
```

```shell
# 停止
docker stop [id]

# 删除容器(-f:强制)
docker rm -f  [id]  

# 删除镜像
docker rmi [imageID]
```

**注意**: 容器在运行状态时: 可以强制删除容器, 但不能强制删除镜像.

进入容器: ```docker exec -it [container name or conteiner id] /bin/bash```



#### 数据卷

容器中管理数据主要有两种方式：

数据卷（Data Volumes）

数据卷容器（Data Volumes Dontainers）

##### 数据卷管理示例

```shell
docker run -d -it --name=centos1 -v /root/dbdata:/test/dbdata docker.io/centos:7.2.1511
```

```shell
# 参数解释
-v: 挂载一个本地的目录到容器中作为数据卷.
例如: -v /xx:/yy: 将容器中的目录/yy挂载到机器中/xx中
```

> 这里我遇到过一个问题: ```Permission denied```
>
> 这可能会导致容器中映射的目录无权限打开, 或者启动容器后是exited状态. 
>
> 可以通过```docker logs [container id]```查看日志.
>
> 原因是SELinux导致的. 这货的访问控制策略相当复杂, 很多运维人员的建议都是将他关闭.
> 查看SELinux状态: ``` /usr/sbin/sestatus``` 或 ```getenforce```
> 关闭: ```vi /etc/selinux/config``` 文件, 将```SELINUX=enforcing```值改为 disabled, 需要重启.

* 在启动centos1容器后, 在宿主机```/root/dbdata```和容器```/test/dbdata```中的任何操作都会互相映射, 相当于一份数据的备份. 
* 关闭或者删除centos1容器后, 宿主机数据依然在, 也可以启动多个容器, 映射数据卷目录中的数据. 

**示例**: mysql数据备份

```shell
docker run -d -p 3307:3306 \
-v /mysql/data:/var/lib/mysql  \
-v /mysql/config:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root \
--name mysqldb1 \
docker.io/mysql:5.7.14
```

> 这里如果再启一个mysqldb2测试, mysqldb1状态是run, 启动mysqldb2后, 会出现mysqldb2异常关闭或连接不通的现象.
>
> 所以再启动mysqldb2容器时候, mysqldb1不能是run的状态, 因为在-v参数中都有对```/mysql/data```目录的读写操作. 



##### 数据卷容器

数据卷容器: 其实就是一个正常容器, 专门用来提供数据卷供其他容器挂载. 

如果你有持续更新的数据需要在容器之间共享. 那么可以创建数据卷容器.

1. 创建创建一个数据卷容器 data_container 

   ``` docker run -it -d -v /dbdata --name data_container docker.io/centos:7.2.1511```

2. 创建centos1, centos2 两个容器, 并挂载data_container容器中的数据卷.

   ```
   docker run -it -d --volumes-from data_container --name centos1 docker.io/centos:7.2.1511
   docker run -it -d --volumes-from data_container --name centos2 docker.io/centos:7.2.1511
   ```


然后这两个容器中都能够共享到数据卷目录```/dbdata```和其中的数据了.

关于```-v```参数:

* 在创建数据卷容器时, ```-v /dbdata 或 -v /root/dbdata:/dbdata```都可以.

  前者只在数据卷容器中创建dbdata目录, 不和本地宿主机映射.

  后者会在本地宿主机也有映射的目录```/root/dbdata```.

关于````--volumes-from````参数:

* 所挂载数据卷的容器自己并不需要保持在运行状态. 

* 可以使用多个, 从而实现从多个容器中挂载多个数据卷. 

* 可以从其他已经挂载了数据卷的容器来挂载数据卷.




##### 利用数据卷容器进行数据迁移备份

* **备份**

```shell
# 将数据卷容器(data_container)中的数据备份到宿主机的当前目录
docker run --volumes-from data_container -v $(pwd):/backup --name worker docker.io/centos:7.2.1511 tar cvf /backup/backup.tar /dbdata 
```

这条命令包含以下几个步骤:

1. 首先利用centos镜像创建了一个容器worker.

2. 使用```--volumes-from```来让worker容器挂载data_container容器的数据卷(即dbdata数据卷).

3. 使用-v $(pwd):/backup参数来挂载本地的当前目录到worker容器的/backup目录.

4. worker容器启动后, 使用了```tar cvf /backup/backup.tar /dbdata```命令来将```/dbdata```下内容备份为容器

   内的/backup/backup.tar, 即也映射到宿主主机当前目录(pwd)下的backup.tar.

执行成功后在宿主机的当前目录下执行```tar xvf backup.tar```后看到看到```/dbdata```备份内容.

* **恢复**

 1. 创建一个带有数据卷的容器:

     ```docker run -d -it -v /dbdata --name db docker.io/centos:7.2.1511```

 2. 创建另外一个新的容器, 挂载db的容器, 并解压备份文件bakup.tar到**所挂载的容器卷**中.

     ```shell
     docker run --volumes-from db -v $(pwd):/backup docker.io/centos:7.2.1511 tar xvf /backup/backup.tar
     ```

  这样就将宿主机当前目录下的backup.tar恢复到db容器:```/dbdata```路径下了.



#### Dockerfile

如何使用Dockerfile构建一个应用程序的镜像.

将应用程序的jpa.jar和Dockerfile文件拷贝到```/root/trainning```目录下

Dockerfile的内容如下:

```dockerfile
# 指定基础image, 要放在前面, 因为后续的指令有依赖域这个image
FROM java:8u102-jdk

# 指定镜像创建者信息
MAINTAINER thank

# 构建指令, RUN可以运行任何被基础image支持的命令
RUN  mkdir /app

# 构建指令, ADD <src> <dest> 从宿主机的src路径拷贝文件到容器中的dest路径
ADD . /app

# 指定容器需要映射到宿主机器的端口
EXPOSE 8888

# 设置container启动时执行的操作
ENTRYPOINT ["java","-jar","/app/jpa.jar","--spring.profiles.active=prod"]
```



```docker build -t jpa:1.0 .```: 用当前目录```.```中的内容构建一个名叫jpa:1.0的镜像.

执行命令后, 会看到以下输出, 每一层都对应Dockerfile文件中定义的指令.

```
Sending build context to Docker daemon  27.7 MB
Step 1 : FROM java:8u102-jdk
 ---> 69a777edb6dc
Step 2 : RUN mkdir /app
 ---> Running in 6779b11de804
 ---> 44fc7d80f5f6
Removing intermediate container 6779b11de804
Step 3 : ADD . /app
 ---> c419c4d34889
Removing intermediate container 00fa44dcbc4e
Step 4 : EXPOSE 8888
 ---> Running in bb4e70a56e04
 ---> 24e66dd52ee1
Removing intermediate container bb4e70a56e04
Step 5 : ENTRYPOINT java -jar /app/jpa.jar --spring.profiles.active=prod
 ---> Running in b5cbcfce01b1
 ---> 45e74f2c2173
Removing intermediate container b5cbcfce01b1
Successfully built 45e74f2c2173

```

执行成功后, 该应用程序镜像就构建出来了. 

接下来可以启动容器来验证.

```docker run -d --name myapp --link mysql:mysql -p 8901:8888 jpa:1.0  ```

```
参数解释: 
--link mysql:mysql
前面的mysql是mysql容器名. 
后面的mysql是别名, 对应应用程序中的spring.datasource.url=jdbc:mysql://mysql:3306/test中的mysql机器名

-p 8901:8888
前面: 对外暴露的端口号(外界访问的)
后面: 程序容器内暴露的端口号
```



#### Docker-compose

Compose是Docker的服务编排工具，主要用来构建基于Docker的复杂应用，Compose 通过一个配置文件来管理多个Docker容器，非常适合组合使用多个容器进行开发的场景

安装方式： https://github.com/docker/compose/releases

docker-compose up

```shell
curl -L https://github.com/docker/compose/releases/download/1.8.0-rc1/docker-compose-uname -s-uname -m > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```



待续...



---------------------------------

#### 问题记录: 

##### 问题一

`systemctl start docker`时报错

> 有些机器上会出现, 有些不会

通过`systemctl status docker.service -l`查看详细错误
```
Error starting daemon: SELinux is not supported with the overlay2 graph driver on this kernel. 
Either boot into a newer kernel or disable selinux in docker (--selinux-enabled=false)
```
意思是说， 此linux的内核中的SELinux不支持 overlay2 graph driver ，解决方法有两个，要么启动一个新内核，要么就在docker里禁用selinux，`--selinux-enabled=false`

解决方法: 
`vi /etc/sysconfig/docker`

将`--selinux-enabled`改为`--selinux-enabled=false`


##### 问题二
我碰到过在虚拟机中启动一个容器时 报出: WARNING: IPv4 forwarding is disabled. Networking will not work.

虽然容器起来了, 但是导致端口转发了, 外部无法访问: 

解决办法:

```vi /etc/sysctl.conf```

或者

``` vi /usr/lib/sysctl.d/00-system.conf```

添加如下代码：

```net.ipv4.ip_forward=1```

重启network服务

```systemctl restart network```

查看是否修改成功

```sysctl net.ipv4.ip_forward```

如果返回为“net.ipv4.ip_forward = 1”则表示成功了

推荐一个解决Docker学习中的疑难杂症: [Docker问答录](https://blog.lab99.org/post/docker-2016-07-14-faq.html#docker-zi-liao-hao-shao-a-wang-shang-de-ming-ling-zen-me-bu-neng-yong)
