---
title: Centos搭建SVN服务器
date: 2017-02-28 12:39:04
categories: Linux & Shell
tags:
  - Linux
  - 运维
  - shell
---



### 序言
工作所需, 写接口很忙, 不想每次别人让我手动去`svn up`更新代码!

<!-- more -->

**预期目的:**

1. 搭建完成SVN服务器, 创建版本库
2. 配置用户和组, 使得可以完成基本的checkout和commit操作
3. 实现仓库代码更新后自动同步到WEB目录下, 不用手动(SVN钩子)



### 安装环境并创建SVN版本库

#### 安装

安装Subversion:

```shell
yum install -y subversion
```

查看是否安装成功:

```shell
svnserve --version
```

显示如下信息: svnserve, version 1.7.14 (r1542130)….

#### 创建版本库

下面创建版本库 

- 建立目录: `mkdir /cloudlink/webapp ``
-  ``svnadmin create /cloudlink/webapp/project `

可以看到在project下生成如下版本文件:

```
conf db format hooks locks README.txt
```

进入conf目录可以看到: 

- authz: 权限配置文件
- passwd: 用户名口令配置文件
- svnserve.conf: 服务配置文件



### 配置用户和组

#### 配置authz

```
[aliases]
# joe = /C=XZ/ST=Dessert/L=Snake City/O=Snake Oil, Ltd./OU=Research Institute/CN=Joe Average

[groups]
# harry_and_sally = harry,sally
# harry_sally_and_joe = harry,sally,&joe
cloudlink = xiefayang ### 将用户加入用户组

# [/foo/bar]
# harry = rw
# &joe = r
# * =

###用户组cloudlink对版本库project具有读和写的权限
# [repository:/baz/fuz]    
# @harry_and_sally = rw
# * = r
[project:/]
@cloudlink = rw
```

#### 配置passwd

```
[users]
# harry = harryssecret
# sally = sallyssecret
xiefayang = xiefayang
```



#### 配置`svnserve.conf`

找到如下配置项, 去掉”#”, 做相应配置

```
# 匿名用户访问权限: 无
anon-access = none

# 普通用户访问权限: 读,写
auth-access = write

# 密码文件
password-db = passwd

#权限配置文件
authz-db = authz

# 版本库所在地
realm = /cloudlink/webapp/project
```



### 测试

启动SVN服务

```shell
svn -d -r /cloudlink/webapp
```

如果不能启动成功就杀掉再重启

```shell
ps aux | grep svn
```

测试checkout, 切换到`/home/www`目录下

```shell
svn co svn://localhost/project
```

查看是否检出成功

测试commit

```shell
vi test.py
svn add test.py
svn commit test.py -m "test commit file"
```

如果显示提交成功，则服务器搭建成功．

**其他测试**
在另一台机器(windows)上安装SVN, 这里我安装了TortoiseSVN

点击SVN Checkout: 填入相应仓库地址, 用户名和密码, 点击OK即可检出

![svn_checkout](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/svn_checkout.png)

注意: 如果检出失败了, 就需要检查一下`/etc/sysconfig`下的iptables. 加入:

```shell
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3690 -j ACCEPT
```



### 钩子实现自动更新和同步

这一步主要实现当更改完代码提交到SVN服务器后, WEB目录下的仓库自动同步.

不需要手动update了

在刚才版本库目录的project下新建`post-commit`文件
(post-commit意味提交后调用此文件更新, post-commit.tmpl只是个模板, 不用管)

- `vi post-commit`

  ```shell
  #!/bin/sh
  export LANG=en_US.utf8
  SVN_PATH=/usr/bin/svn   
  WEB_PATH=/home/www  #web目录
  $SVN_PATH update $WEB_PATH --username 'xiefayang' --password '111111' --no-auth-cache
  ```

​        看看/home/www 的用户组和所有者(假如都是:thank)

- 修改post-commit用户为www的目录用户: `chown thank:thank post-commit`
- 赋予执行权限: `chmod 755 post-commit`

至此, 一定要区分[SVN服务器]和[WEB目录]这两个地址

如果SVN服务器和WEB目录都在同一台机器.

那么在其他地点修改代码提交后, WEB目录就会自动更新同步.

but: 如果SVN服务器在远程另外一台机器. 待研究~