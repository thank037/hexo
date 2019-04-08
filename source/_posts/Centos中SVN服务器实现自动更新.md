---
title: Centos中SVN服务器实现自动更新
date: 2017-03-09 12:39:04
categories: Linux
tags:
  - Linux
  - 运维
  - shell
---



### 前言

> 在上一篇[《Centos搭建SVN服务器》](http://xiefayang.com/2017/02/28/centos%E4%B8%AD%E6%90%AD%E5%BB%BAsvn/)中提到用钩子实现自动更新和同步.

<!-- more -->

> 难受的是我们的SVN服务器上是不允许我直接登录放hooks的, 但还是不想每次别人喊我手动去执行更新
>
> 于是尝试了以下两种方案



利于钩子实现有一个前提是: SVN服务器版本库和要更新的WEB目录在同一台机器(SERVER-A)上

实际是利用`/hooks/post-commit`这个文件的 提交后自动触发功能.

在post-commit中编写`svn up`脚本来实现SERVER-B提交后, 触发SERVER-A的post-commit文件.

自动执行脚本, 更新SERVER-A中的WEB目录.

如果SVN服务器版本库在SERVER-A上, 要更新的WEB目录在SERVER-B上呢?



### 方案一: 循环定时更新

先来说一种较"傻"方法, 事实上我刚开始也是这么做的

在WEB目录所在机器(SERVER-B)上, 写一个循环定时更新的脚本.

例如每过10秒update:

```shell
#!/bin/bash
while [ 1 -gt 0 ]
do
  sleep 10
  svn update  /home/www/project/ &>/dev/null
done
```

加上日志的方式:

```shell
#!/bin/bash
pre_update="$(svn log /home/www/project/ | head -2 | gawk NR==2{'print $1'})"
while [ 1 -gt 0 ]
do
  svn update /home/www/project/ &>/dev/null
  now_update="$(svn log /home/www/project/ | head -2 | gawk NR==2{'print $1'})"
  if [[ ${now_update} > ${pre_update} ]]
  then
    echo " update_time : $(date) " >>/tmp/svn_update.log
    pre_update=${now_update}
  fi
  sleep 5
done
```



### 方案二: 利用钩子触发更新

还是利用post-commit的提交后自动触发功能.

但是这里update的是另外一台机器了, 所以就需要远程传输文件.



#### 命令

介绍两个命令:

- `scp`: 用于在Linux下进行远程拷贝文件的命令

  和它类似的命令有cp，不过cp只是在本机进行拷贝不能跨服务器，而且scp传输是加密的.

- `rsync`: 是一个远程数据同步工具，可通过LAN/WAN快速同步多台主机间的文件

  rsync使用所谓的“rsync算法”来使本地和远程两个主机之间的文件达到同步.
  这个算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快.

> 注意: 虽然 rsync比scp会快一点，但当小文件众多的情况下，rsync会导致硬盘I/O非常高，而scp基本不影响系统正常使用。

具体的语法和参数参照:

- rsync命令: [rsync命令](http://http//man.linuxde.net/rsync)
- scp命令: [scp命令](http://http//man.linuxde.net/scp)



#### 配置ssh

例如: 上传本地文件到远程机器指定目录: `scp [本地路径] root@10.10.10.10:[远程路径]`

之后会让你输入远程登录的密码.

因为最终是以脚本方式来拷贝到远程机器.

为了方便, 我配置了SSH免密登录

配置很简单:

1. 命令在A上生成秘钥, 在`.ssh`中就能看到`id_rsa`和`id_rsa.pub`两个文件
2. 把`id_rsa.pub`随便拷贝到B上, 并将其中内容放到`.ssh`下的`authorized_keys`就可以了
   注意权限和用户就好了.
   可以参考: [SSH免密码远程登录Linux](http://www.cnblogs.com/bootoo/p/5068514.html)

配置完成后可以`ssh root@[B的IP] `试试, 应该不输入密码就登录了.



#### 写脚本

下面, 就开始`vi xx/project/hooks/post-commit`:

```shell
#!/bin/sh

export LANG=en_US.utf8
SVN=/usr/bin/svn
WEB=/home/www/project/
RSYNC=/usr/bin/rsync
LOG=/tmp/uplog_xfy.log
WEBIP="192.168.100.212"

echo " start update ! " >> $LOG

#更新本机WEB目录
$SVN update $WEB --username 'xiefy' --password 'xiefy'

#远程传输
if [ $? == 0 ]
then
   echo `date` >> $LOG 
   #chown -R xiefy:cloudlink /home/xiefy/project
   
   #1.scp方式
   scp -r $WEB root@$WEBIP:/home/xiefy/ >> $LOG
   
   #2.rsync同步方式
   #rsync -vaztpH --timeout=90 $WEB root@$WEBIP:/home/xiefy/ >>LOG   
fi
```

好了, 作为一名开发狗.

Linux下的SVN服务器就研究到这…

**The End**