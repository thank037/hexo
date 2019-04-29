---
title: Linux远程无密传输实现脚本
date: 2017-05-15 12:39:04
categories: Linux
tags:
  - Linux
  - shell
---

### 序言

> 作为一名后台开发, 开发过程中, 除了要部署后台的微服务, 还要对前端的WEB目录进行部署.<!-- more -->
> 虽然每次只需要2分钟左右, 但却是重复性劳动. 我认为很xx!


听说过俄罗斯一程序员, 认为90秒以上的事情, 就必须要做成脚本, 比如煮咖啡, 骗老婆说要加班等等…

所以我决定花几个小时时间来把这种事情写成脚本.

目前公司的前端WEB目录部署其实就是文件的上传下载, 不需要启动什么的.

既然前端人员不想用Linux也不想用FTP.

那… 在linux下编写成脚本后, 让前端人员直接在windows下通过一个按钮点击, 就完成WEB目录的一键部署.

我就完全可以脱身这件事.



### 场景描述

首先预上线环境部署WEB目录的流程:
- 开发环境: 213
- 测试环境: 212
- 预上线环境: 110

前两个环境是我来手动维护, 因为写了很多脚本(自动Git拉取, Maven打包, 一键杀死/启动微服务等…)

所以维护起来很轻松.

110环境是通过Jenkins来构建的, 每次构建完成都会生成全新的虚拟机镜像

服务器, 数据库, Redis都会是新的(IP不变)

所以WEB目录的部署, 是在212上拉取测试通过的目录到预上线的WEB目录.

但是经常会有bug修正, 每次修正完又要重新拉一遍.

所以阿. 看起来这个脚本应该很简单. 就是copy主机A的WEB目录到主机B



### 建立通信

但是涉及到两台机器通信, 就会有密码. 总不能执行脚本之后再来输入密码.

有以下两种方式: 

- ssh免密:建立互信关系
- 通过expect自动输入



#### 方式一: ssh

关于什么是ssh, 公钥, 密钥 原理什么的可以Google 简单介绍下操作方法:

假设现在有

- A(192.168.100.212)
- B(192.168.100.213)

两台机Linux(CentOS7), 要实现从A用ssh远程登录到B:

**1. 首先在A上生成公钥**

```shell
ssh-keygen -t rsa
```

这里一直点回车即可.
执行完成后, 在`~/.ssh/`下就会生成`id_rsa`和`id_rsa.pub`两个文件

**2. 把A机器刚才生成的id_rsa.pub文件复制到B上.**

```shell
scp ~/.ssh/id_rsa.pub root@192.168.100.213:/root/
```

然后在B的root目录下就能看到id_rsa.pub了(这里放root下只是临时放一下, 你放哪都行)

**3. 在B机器中, 把id_rsa.pub中的内容加到authorized_keys文件**

```shell
cat id_rsa.pub >> ~/.ssh/authorized_keys
```

如果~/下没有.ssh或者.ssh下没有authorized_keys, 就自己手动创建

**4. 在B上重启sshd服务**

```shell
service ssh restart
```

完了, 现在在A上, 试试登陆B: `ssh root@192.168.100.213`. 应该就不输入密码了



**注意事项**：

1. ssh-keygen -t rsa 生成公钥时一直回车不要输入密码，就是空密码
2. 要用哪个用户远程登录就把`id_rsa.pub`复制到用户对应路径下，如：root用户就复制到`/root/`下; xxx用户就复制到`/home/xxx/`下，不要混了
3. B机上的文件权限： `.ssh`文件夹(700)；`authorized_keys`(600)



#### 方式二: expect

在我写这个脚本时候, 并未使用上面那种方法.

原因是上面提到过, 每次预上线环境构建完都会是一台全新虚拟机(只有IP不变)

所以不可能每次构建完都去配一次ssh信任关系.

所以介绍一种更优雅的方式, 在脚本里自动输入密码(当屏幕交互需要输入时自动输入)

这里用到的就是expect.

要使用首先要安装:

```shell
sudo yum install expect
```

建立scp.exp

```shell
#!/usr/bin/expect  
set timeout 20  

if { [llength $argv] < 2} {  
 puts "Usage:"  
 puts "$argv0 local_file remote_path"  
 exit 1  
}  

set local_file [lindex $argv 0]  
set remote_path [lindex $argv 1]  
set passwd your_passwd  

set passwderror 0  

spawn scp $local_file $remote_path  

expect {  
 "*assword:*" {  
	 if { $passwderror == 1 } {  
	 puts "passwd is error"  
	 exit 2  
	 }  
	 set timeout 1000  
	 set passwderror 1  
	 send "$passwd\r"  
	 exp_continue  
 }  
 "*es/no)?*" {  
	 send "yes\r"  
	 exp_continue  
 }  
 timeout {  
	 puts "connect is timeout"  
	 exit 3  
 }  
}
```

这里我找到一个比较通用的(除了输入自动输入密码之外, 还支持yes/no的自动选择)

可以看到第一行并不是`#!/bin/bash`. 所以这段脚本不是由bash解释执行, 而是expect.

代码的语法和shell也不一样.

```shell
set local_file [lindex $argv 0]   # 本地要拷贝的文件
set remote_path [lindex $argv 1]  # 远程拷贝文件的目标地址
set passwd your_passwd   #your_passwd就是你要连接远程机器的密码
spawn scp $local_file $remote_path  # 这个就是实际的拷贝, 如果要复制目录下的文件, 可以加-r以递归方式复制
```

例如我要把本机(213)的某个文件复制到主机110上

```shell
./scp.exp /hahah/hello.py root@192.168.120.110:/xx/yy/
```



#### scp目录的坑

对于上述第二种方式: scp的目录问题, 有个坑记录一下

例如:

- `/scp.exp /hahah/hello.py root@192.168.120.110:/xx/yy/`: 是把hello.py文件拷贝到yy目录下, 没问题.

- `./scp.exp /hahah/dir1 root@192.168.120.110:/xx/yy/`: 是把dir1目录拷贝到yy目录下, 也没问题

- `./scp.exp /hahah/dir1/ *root@192.168.120.110:/xx/yy/`: 按理说应该是把dir1目录下的文件(不包括dir1目录本身)拷贝到yy目录下.

  这个直接在scp命令下是没问题的. 但是因为这里执行的是scp.exp脚本. 

  他会直接把*当成特殊字符, 所以需要”\”来转义, 例如: 

  ```shell
  ./scp.exp /hahah/dir1/* root@192.168.120.110:/xx/yy/
  ```



#### 方便调用`scp.exp`

两种通信方式就介绍完了.

因为我要通过一个请求去调用scp脚本. 为了方便, 又写了一个脚本去调用scp.exp

`update_pre_online.sh`内容如下:

```shell
#!/bin/bash

echo "Content-Type: text/html"
echo ""

# Source path
SRC_PATH=/home/www/update_pre_online/xxws-web-120/
# Target path and Ip address
#TAG_PATH=/home/xiefy/dir/
#TAG_ADDRESS=192.168.100.214
TAG_PATH=/sysroot/tmp/xxws/
TAG_ADDRESS=192.168.120.110

# Pull 212WEB to 213WEB
scp -r root@192.168.100.212:/home/www/xxws-web/* ${SRC_PATH}

chmod -R 755 ${SRC_PATH}
echo "============================"
echo "递归修改目录访问权限(755)..."
echo "============================"

sleep 1

# execute expect, pull to 110WEB
/home/www/update_pre_online/scp.exp ${SRC_PATH}\* root@${TAG_ADDRESS}:${TAG_PATH}

cat ~/update_pre_online.log
echo "--------------------------------------"
echo " 更新 212WEB目录 到 110WEB目录 成功!  "
#echo " 禁止更新! 新的预上线后台微服务未更新 "
echo "--------------------------------------"
sleep 2
exit 0
```

### 总结

完成后, 加一个小页面按钮, 效果如下:
![这里写图片描述](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/shell%E9%83%A8%E7%BD%B2%E6%95%88%E6%9E%9C%E5%9B%BE.png)

虽然之前部署这个熟练的话每次只需要三分钟.

但我编写脚本加测试就用了好几个小时.

成果就是现在每次只需要点击按钮后等待5秒即可, 最重要的是!!!　不需要我来操作了.

**The End**