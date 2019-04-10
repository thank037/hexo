---
title: Docker镜像仓库
thumbnail: /thumbnails/harbor_logo.png
date: 2019-04-10 15:00:12
categories: Docker
tags:
  - Docker
  - 镜像仓库
---


[TOC]

这里仅以一个场景: 自己制作的镜像如何分享, 主要说明私有镜像仓库[Harbor](https://github.com/goharbor/harbor)的搭建使用

<!-- more -->

## Docker Hub

首先介绍[Docker Hub](http://hub.docker.com)


### 推送&拉取镜像

推送自己的镜像到Docker Hub上, 因为网速... 这里用一个超小的镜像试验

```shell
[root@bogon ~]# docker images | grep busy
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
busybox                      latest              af2f74c517aa        6 days ago          1.2MB
```


**1. 打标签**

```shell
docker tag busybox:latest thank037/busybox:1.0
```

```shell
[root@bogon ~]# docker images | grep busy
busybox                      latest              af2f74c517aa        6 days ago          1.2MB
thank037/busybox             latest              af2f74c517aa        6 days ago          1.2MB
```

> 注意这里的thank037就是DockerHub中的username



**2. 推送**

```shell
docker push thank037/busybox:1.0
```

去自己的镜像仓库看下吧, 有了

> 如果推送过程中提示: `denied: requested access to the resource is denied`
>
> 登录一下即可: `docker login`



拉取镜像很常见

```shell
docker pull thank037/busybox:1.0
```


<br>

## 国内的镜像仓库

国内也有很多, 例如[网易](https://c.163.com/hub#/m/home/), DaoCloud, [阿里云](https://dev.aliyun.com/search.html)等, 上传镜像和拉取镜像的姿势基本和Docker Hub一样

网易的这两天个人认证无法使用, 这里实验阿里云的 

首先需要在`阿里云-容器镜像服务`中创建一个命名空间

![创建命名空间](https://ww1.sinaimg.cn/large/007rAy9hgy1g1xc4ghsm4j30vn0izaar.jpg)

可以设置仓库类型, 公有私有



### 推送和拉取

省略自己制作镜像的过程, 这里以一个我已经拉取好的官方镜像`redis:latest`为例

首先登录
```shell
$ sudo docker login --username=xxx registry.cn-hangzhou.aliyuncs.com
```

```shell
# tag
docker tag redis:latest registry.cn-hangzhou.aliyuncs.com/thank/redis:1.0

# push 
docker push registry.cn-hangzhou.aliyuncs.com/thank/redis:1.0

# pull
docker pull registry.cn-hangzhou.aliyuncs.com/thank/redis:1.0
```

去镜像仓库看看吧, 有了

![镜像仓库](https://ww1.sinaimg.cn/large/007rAy9hly1g1xcp2mip3j30zv08qaaa.jpg)



### 基于代码源构建

上面的方式是自己在docker环境中制作好镜像, 上传到仓库, 这里介绍一种基于代码源的构建方式

以GitHub为例, 代码源为: https://github.com/Thank037/spring-cloud-study2

1. **创建镜像仓库**: 写好仓库名称, 选择好命名空间即可
2. **设置代码源**: 选择github, 第一次使用需要绑定账号
3. **构建设置**
   - 构建设置: 例如可以开启代码变更自动构建镜像
   - 构建规则设置: 设置代码源的分支/tag, Dockerfile目录和名字, 需要构建的镜像版本

4. **开始构建**: 点击规则上的立即构建即可

每次的构建过程会输出到构建日志中, 可以进行查看

![构建设置](https://ww1.sinaimg.cn/large/007rAy9hly1g1xhjiy4gkj31a10jw0tj.jpg)


<br>


## 私有仓库

> 出于安全和速度的考虑, 很多企业会选择搭建私有镜像仓库

### **registry**

#### 安装和使用

是一个官方提供的镜像仓库, 安装极其简单! 

```shell
docker pull registry:2
```

拉取完成后启动registry: `docker run -d -p 5000:5000`

推送和拉取过程跟推送到公有仓库差不多

```shell
# 打标签
docker tag busybox:latest localhost:5000/busybox:1.0

# 推送
docker push localhost:5000/busybox:1.0

# 拉取
docker pull localhost:5000/busybox:1.0
```



**需要注意的是**

如果其它主机在推送到registry所在主机时可能出现安全错误, 也就是镜像仓库和docker客户端不在一台机器上

在Client端`/etc/docker/daemon.json`中加入: 

```json
{ "insecure-registries":["ip:5000"] }
```

然后重启docker即可 `systemctl restart docker `

除了推送和拉取镜像, registry还提供了一些API, 查看镜像信息, 删除等... 例如:

```shell
[root@bogon ~]# curl localhost:5000/v2/_catalog
{"repositories":["busybox"]}
```



可以看到registry很轻便, 具备了基础的镜像管理功能, 但是有以下两个缺点

- 不具备授权认证功能, 需要自己去做一些认证方案
- 虽有API, 但没有一个好看的界面

<br>

### **harbor**

harbor意为港湾, 很贴合它的作用

> github地址: https://github.com/goharbor/harbor
>
> 记得以前是在vmware下, 现在地址转到goharbor下了


官方的定义是企业级私有Registry服务器, 实际内部也是依靠docker registry

那么它与registry相比, 一定提供了很多企业级的特性, 例如:

- 提供良好的Web界面
- 提供良好的安全机制, 包括角色, 权限控制, 日志等
- 支持水平扩展
- 传输优化, 因为镜像的分层机制, 所以每次传输并非全量, 从而提升速度



#### 搭建

**环境准备**

在安装harbor之前, 需要docker环境, 除此还需要docker-compose, 不说了



**安装Harbor**

在[github release](https://github.com/goharbor/harbor/releases)中查看需要安装的版本, 这里我用当前最新的v1.7.5

文档中有两种安装方式: offline和online, 也就是在线和离线

- online: 在线方式会下载一个很小的tar包, 里面只有`docker-compose.yml`和启动脚本, 会在启动安装时在线拉取镜像
- offline: 下载一个大概500多M的tar包, 里面包括了镜像文件, 所以安装时会快

我的浏览器设置了代理, 所以选择offline的离线包, 再上传放到docker环境的主机上

大致看下解压后的目录

```shell
[root@localhost opt]# tree harbor -CL 2
harbor                                 
├── common                             
│   ├── config                         
│   └── templates                      
├── docker-compose.chartmuseum.yml     
├── docker-compose.clair.yml           
├── docker-compose.notary.yml          
├── docker-compose.yml                 
├── harbor.cfg                         
├── harbor.v1.7.5.tar.gz               
├── install.sh                         
├── LICENSE                            
├── open_source_license                
└── prepare                            
```

接下来对`harbor.cfg `几个配置做修改, 我修改了如下: 

```properties
# 注释中也说了, 不要设置成localhost和127.0.0.1阿
hostname = hub.thank.com

# 邮箱相关的设置
email_server = smtp.163.com
email_server_port = 25
email_username = coderthank@163.com
email_password = ******
email_from = thank <coderthank@163.com>

# 修改admin默认的登录密码
harbor_admin_password = 123123
```

更多详细的设置可参考[官方文档](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)和配置文件的注释

然后执行`install.sh`即可

```
[Step 0]: checking installation environment ...

Note: docker version: 1.13.1

Note: docker-compose version: 1.16.1

[Step 1]: loading Harbor images ...
...略

[Step 2]: preparing environment ...
...略

[Step 3]: checking existing instance of Harbor ...


[Step 4]: starting Harbor ...
...略

✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at http://hub.thank.com.
For more details, please visit https://github.com/goharbor/harbor .
```

从Step可以看到这个一键脚本帮我们做了傻瓜式的安装

docker-compose帮我们完成了Harbor所需要的多个容器服务

来看一眼

- 镜像

  ```shell
  [root@localhost harbor]# docker images --format "table {{.Repository}}\t{{.Size}}" | grep harbor
  goharbor/chartmuseum-photon     113 MB
  goharbor/harbor-migrator        679 MB
  goharbor/redis-photon           101 MB
  goharbor/clair-photon           164 MB
  goharbor/notary-server-photon   135 MB
  goharbor/notary-signer-photon   132 MB
  goharbor/harbor-registryctl     102 MB
  goharbor/registry-photon        86.7 MB
  goharbor/nginx-photon           35.9 MB
  goharbor/harbor-log             81.4 MB
  goharbor/harbor-jobservice      84.1 MB
  goharbor/harbor-core            95.6 MB
  goharbor/harbor-portal          40.6 MB
  goharbor/harbor-adminserver     72.3 MB
  goharbor/harbor-db              138 MB
  ```

- 容器

  ```shell
  [root@localhost harbor]# docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Ports}}" | grep harbor
  c517af1cff6a        goharbor/nginx-photon:v1.7.5             0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp
  7bb6eac57291        goharbor/harbor-jobservice:v1.7.5
  723cb4c96a0d        goharbor/harbor-portal:v1.7.5            80/tcp
  1425f64756d1        goharbor/harbor-core:v1.7.5
  f19a00ce693b        goharbor/registry-photon:v2.6.2-v1.7.5   5000/tcp
  4d3f0f8929c7        goharbor/harbor-adminserver:v1.7.5
  23df470d2b0d        goharbor/harbor-db:v1.7.5                5432/tcp
  e7d6d1cc3b1b        goharbor/harbor-registryctl:v1.7.5
  6e2912ec566e        goharbor/redis-photon:v1.7.5             6379/tcp
  ee8685192041        goharbor/harbor-log:v1.7.5               127.0.0.1:1514->10514/tcp
  ```



#### 界面使用

因为刚才在配置文件赔了一个假域名, 这里我修改下hosts
```properties
192.168.118.143 hub.thank.com
```

访问`http://hub.thank.com`, 可以看到Harbor的登录页面

> 直接访问IP也可, 默认80端口

登录后的主页面

![harbor主页](https://ww1.sinaimg.cn/large/007rAy9hly1g1xkfzw2ngj31hc0ooq3v.jpg)

测试: 创建用户`thank`, 创建项目`cloudlink-base`和项目中添加用户`thank`, 角色为项目管理员

其它功能就不做介绍了, 中文简体点一点就好啦



#### 运维管理

从前面的一键启动脚本也能看出, 我们可以用docker-compose统一完成harbor服务的运维管理

- 停止: `docker-comose stop`
- etc...



#### image管理

这里测试一下镜像的拉取和上传

方法跟其它仓库的没什么不一样, 不说了

```shell
# login
docker login hub.thank.com

# image tag 
docker tag redis:latest hub.thank.com/cloudlink-base/thank-redis:latest

# push 
docker push hub.thank.com/cloudlink-base/thank-redis:latest

# pull
docker pull
```



#### https问题

刚才在配置文件`harbor.cfg`中可以看到默认配置了http协议访问

这里在其它主机上传镜像时, docker客户端默认都是https访问Harbor, 所以会出现`connection refused`的错误

有两种解决办法: 

1. 前面提到过, 在`/etc/docker/daemon.json`加入
```json
{
  "insecure-registries" : ["hub.thank.com"]
}
```

2. 修改harbor配置文件为https方式: `ui_url_protocol = https`, 除此之外还需要配置证书

不是很复杂, 可以参照[搭建Harbor企业级docker仓库#五、Harbor配置TLS证书](https://www.cnblogs.com/pangguoping/p/7650014.html)

