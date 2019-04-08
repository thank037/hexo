---
title: Shell脚本方便微服务部署
date: 2017-02-15 12:39:04
categories: Linux
tags:
  - Linux
  - shell
---


### 序言

> 之前作为一个开发狗. 并不太多涉及Linux系统管理. 对shell脚本也一无所知.<!-- more -->
>
> 但是对于jar包的部署和打包是要经常在Linux环境下的.
>
> 对于极度懒人的我来说过于繁琐, 每次都要命令来查询, 复制粘贴执行.

能让计算机做的为什么要我们自己做呢哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈

只是为了解决当务之需才去研究shell.  并没有系统完整的学习shell

目前在测试和开发环境已经有十多个脚本来负责服务的批量和单个操作, 用起来得心应手, 能够实现:

- Git远端拉取指定分支的服务
- mvn打包编译
- 单个/批量启停





### 关于shell

shell用C语言编写, 是Linux管理中必不可少的. 也是系统和硬件交互的一种介质.

我们通过指令传给shell, shell再通过系统内核就能实现对计算机硬件的操作.

他有自己的命令和程序设计语言, 供我们编写, 在我看来, 如同PL/SQL编程

我们需要了解他的语法.

shell需要的环境超级简单: 

- 文本编辑器 
- 脚本解释器

所以在Linux环境下可以轻而易举的编写shell



### 部署脚本

接下来按照我要实现的目的, 选择性的记录一些命令

#### 杀死服务进程

要杀死服务的进程, 需要先找到他的网络连接信息. 所以通常做法是先显示:程序

```shell
ps -aux|grep java
```

程序的标识, 状态, ID, 资源占用等信息就会显示出来

再找到进程的ID, 通过kill ID 来杀死这个服务

**lsof命令**

这里要说的是lsof命令: [很好的lsof命令介绍](https://linux.cn/article-4099-1.html)

在这里我只用到 -i, -t, -sTCP

> **-i:[port]**: 显示与指定端口相关的网络信息
> **-t**: 仅获取进程的ID
> **-iTCP**: 仅显示TCP连接(UDP同理)

找出监听状态的端口:

```shell
lsof -i -sTCP:LISTEN
```

使用grep命令 也可以获得:

```shell
lsof -i | grep -i LISTEN
```

所以要杀死某个启动服务的进程如下:

```shell
kill -15 `lsof -i:[端口号] -t -sTCP:LISTEN`
```

再通过一个简单的if判断来确定进程已经杀死

```shell
kill -15 `lsof -i:[端口号] -t -sTCP:LISTEN`
sleep 1
if [ -z $(lsof -i:[端口号] -t -s TCP:LISTEN) ]
then  
  echo " port killed! "
fi
```

在这里, 我写一个通过传参杀死服务进程的脚本 `killed_single_port.sh`

```shell
#!/bin/bash

printf " Please input killed port: "

read port

port_array=(8901 8902 8903 8905 8906 8907 8908 8909 8761 8050)

result=`echo "${port_array[@]}" | grep -wq "$port" &&  echo "y" || echo "n"`

if [ "n" == "${result}" ]
then
   echo "port:\"${port}\" not exits ! "
   exit 5
fi

kill -15 `lsof -i:${port} -t -sTCP:LISTEN`
sleep 1
if [ -z $(lsof -i:${port} -t -s TCP:LISTEN) ]
then
  echo " port \"${port}\" is killed! "
fi

exit 5
```



#### 启动服务

通常, 用nohup命令来不挂断的运行命令来启动一个我们的服务

**nohup命令**

在Linux/Unix启动程序中, 我们一般希望是在后台运行, 在末尾加”&”即可

```shell
nohup java -jar /aa/bb/xxx.jar
```

如果需要程序的启动信息 通过”>”重定向到日志中

0、1和2分别表示标准输入、标准输出和标准错误信息输出，可以用来> 指定需要重定向的标准输入或输出。

- 0: 标准输入信息
- 1: 标准输出信息
- 2: 标准错误输出信息
- /dev/null: 正常输出和错误信息都不显示



例如:

```shell
nohup java -jar /aa/bb/xxx.jar  \
--server.port=8050 \
--eureka.client.serviceUrl.defaultZone=http://127.0.0.1:8761/eureka/ \
>/my_log/zuul.log 2>/my_log/zuul_error.log &
```

如果要批量启动几个程序, 这几个服务之间可能有关系.

如果把这几个程序的启动命令写到一个脚本中顺序执行肯定不行.

所以这里又用到lsof命令来获得一个程序的状态, 进而来判断是否继续执行

以下script实现判断当x服务启动以后, 再顺序执行a,b程序. 最后再判断a,b是否启动成功

```shell
#!/bin/bash
echo "x server booting..."
nohup java -jar /aa/bb/x.jar &>/dev/null &

# 判断9900的x服务启动与否
while [ -z "$(lsof -i:9000 -t -s TCP:LISTEN)" ] 
do
  sleep 3
done
if [ -n "$(lsof -i:9000 -t -s TCP:LISTEN)" ]
then
  echo "x server start success!"
else
  echo "ERROR: x server boot failure !!!! "
  exit 5
fi

# 只有当9900的x服务启动成功后, 继续执行下面

# 启动a服务
nohup java -jar /aa/bb/a_server.jar  \
--server.port=8901 \
>/my_log/a.log 2>/my_log/a_error.log &

# 启动b服务
nohup java -jar /aa/bb/b_server.jar  \
--server.port=8902 \
>/my_log/b.log 2>/my_log/b_error.log &


# 判断a(port:8901),b(port:8902)服务是否启动成功
i=2
a_server=1
b_server=1

while [ 1 -gt 0 ] && [ ${i} -gt 0 ]
do
  if [ -n "$(lsof -i:8901 -t -s TCP:LISTEN)" ] ]
  then
    echo "a server start success !!!"
    i=$[ ${i} - 1 ]
  fi
  if [ -n "$(lsof -i:8902 -t -s TCP:LISTEN)" ] ]
  then
    echo "b server start success !!!"
    i=$[ ${i} - 1 ]
  fi
done
```