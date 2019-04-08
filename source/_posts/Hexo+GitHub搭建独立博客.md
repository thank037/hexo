---
title: Hexo+GitHub搭建独立博客
date: 2016-09-14 12:39:04
categories: 独立博客
tags:
  - github
  - hexo
---

[TOC]

### 前言

> 之前, 一直看CSDN上写别人的博客, 博客园, CSDN上都有很丰富的资料. <!-- more -->
>
> 自己在CSDN上记录学习文章过程中, 发现了一些不太喜欢的地方.  
>
> 比如有时候再嵌入代码和图片时候兼容问题(可能是自己的错误) , 会有广告,  缺乏个性. 
>
> 所以就想搭建一个独立的博客.



在互联网上搜集一些资料, 参考了很多别人的个人博客. 最终选择Hexo+GitHub Pages

整体难度不是很大, 但是过程还是比较繁琐的, 需要懂点Git, 很多是需要命令中的配置. 

如果你有强迫症, 想要把博客定制的比较符合心意, 就更需要读更多文档来做个性化配置.

因为网上有很多文档. 包括官方文档. 其实完全够用. 本文简短描述过程. 不做累赘的配置描述.

推荐两篇比较好的博文:

- [如何搭建一个独立博客——简明Github Pages与Hexo教程](http://blog.sina.com.cn/s/blog_617ccc0c0101h84p.html)
- [一步步在GitHub上创建博客主页(7个系列)](http://www.pchou.info/ssgithubPage/2013-01-03-build-github-blog-page-01.html)

### 介绍

#### Hexo

关于Hexo, 文档很详细了: https://hexo.io/zh-cn/docs/



#### GitHub Pages

我有自己的阿里云ECS, 但是不想放在上面, 因为是借同学在校证明买的优惠机, 他毕业机器就没了...

好在看到GitHub Pages

> github Pages 是 Github 的静态页面托管服务。它设计的初衷是为了用户能够直接通过 Github 仓库来托管用户个人、组织或是项目的专属页面。 可以帮助我们通过Git来管理的静态服务器.
> 相比wordpress, 它的搭建部署较为复杂, 需要安装一些软件. 也会由于粗心容易出现一些bug需要解决. 比较适合于爱折腾的coder.

说明:

- 仓库存储的所有文件不能超过 1 GB
- 页面的带宽限制是低于每月 100 GB 或是每月 100,000 次请求
- 每小时最多只能部署 10 个静态网站
- github pages有300M免费空间
- 依托github, 上面有很多大神. 拥抱开源, 分享知识

在搭建之前, 需要懂一点Git相关知识, 关于Git笔记. 可以参考

> 个人博客: [git小玩](http://xiefayang.com/2016/03/25/git%E5%B0%8F%E7%8E%A9/)
>
> CSDN: [git小玩](http://blog.csdn.net/u011650291/article/details/50983167)



### 安装

这俩是必须的

- Git: <http://git-scm.com/>
- Node.js: <http://nodejs.org/>

软件较小, 近乎傻瓜式安装.



### 注册GitHub

搭建博客之前, 需要先到github上做一些事情.

进入 <http://www.github.com/> sign up for GitHub 即可. 牢记自己注册信息, 尤其email. 需要到邮箱自己验证一下.

登录之后 点击头像下的Settings选项.

![这里写图片描述](https://ww1.sinaimg.cn/large/007rAy9hgy1g1n8sbiz29j30tl0gr0ub.jpg)

标注的几个就是需要自己设置的, 比如SSH和仓库.

### 配置SSH keys

前面我们下载好了git, 而这个秘钥, 就是让本地和远程GitHub建立联系.

打开前面下载好的GitBash, 或者右击某个目录下的git bash here.
输入

```shell
$ cd ~/. ssh #检查本机的ssh秘钥
```

提示No Such.. 需要我们自己配置.

```shell
$ ssh-keygen -t rsa -C #“你注册的email”
```

提示: 

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):
```

这里会在你本机生成一个rsa的加密秘钥. 点击回车

接着会提示

```
Enter passphrase (empty for no passphrase)
```

让你输入一个密码, 你可以输入也可以输入空, 直接回车, 这个密码是以后提交项目用的.

提示: Enter same passphrase again: 再次输入

当提示your identification has been saved in …[目录] 和一段随机蚂图形后. 表示已经生成成功.

这时候去你生成的那个目录找`id_rsa.pub`文件, 这个目录跟操作系统有关, 有的在Documents and Settings, win7可能找不到这个文件夹. 或者被隐藏, 需要设置一下再查找文件. 我的是在`C:\Users\lenovo.ssh`下.

找到这个文件, 把里面的内容复制出来. 

到github页面的settings下, 找到SSH keys and GPG keys. 如图所示, 复制到key框中.

![这里写图片描述](https://ww1.sinaimg.cn/large/007rAy9hly1g1n8tmjcf7j30tn0h2aba.jpg)

添加成功后, 在git bash中输入: `$ ssh -T git@github.com`

提示:

> The authenticity of host ‘github.com (207.97.227.239)’ can’t be established.
> RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
> Are you sure you want to continue connecting (yes/no)?

输入yes, 接着提示:

> Hi thank037! You’ve successfully authenticated, but GitHub does not provide shell access.

表示已经连接成功了!

这里继续输入一下以后提交的用户信息(填自己的):

> git config –global user.name “thank037”
> git config –global user.email “coderthank@163.com”



### 建立仓库(repositories)

如图所示:
![这里写图片描述](https://ww1.sinaimg.cn/large/007rAy9hly1g1n8usrsjoj30t90hsabc.jpg)

这里的repositories name 需要是你的 用户名+github.io, 如果要用GitHub Pages建立个人博客, 格式必须这样, 不能乱写. 描述自己随便写. 个人博客的仓库只能建立一个.

然后点击 Crete Repositories , 你的个人博客仓库就建立好了. 以后所有的博客内容, 包括模板, 文章等 都会存放在这里.

### 部署Hexo 搭建工具

Hexo是一个简单, 轻便的静态博客框架. 

它基于node, 使用之前可以检查下node环境: `node -v`

然后输入:

```shell
$ npm install -g hexo # 开始安装….
```

会在你的用户全局目录下安装.

等待安装完毕后, 随便在目录中创建一个叫”Hexo”的文件夹.

将git bash 切换到这个目录下, 输入:

```shell
$ hexo init # 会在Hexo文件夹中初始化网站需要的所有文件.
```

安装好的目录结构如图:
![这里写图片描述](https://ww1.sinaimg.cn/large/007rAy9hgy1g1n8wxkbdzj30hw08874o.jpg)

这时候, 点击themes中可以看到一个landscape, 它是默认的主题. 你也可启动服务看一下.



### 选主题

挑选自己喜欢的主题, 可以参见知乎这里的主题推荐: <http://www.zhihu.com/question/24422335>

我选择的是[**NextT 精于心，简于形**](https://github.com/iissnan/hexo-theme-next). 主题以黑白两种永恒经典的色调搭配, 四周做了大量留白. 

看起来就是 简单! 简单!! 简单!!!

进去之后点击Clone or Download, 把它的URL复制下来.

```shell
$ git clone https://github.com/iissnan/hexo-theme-next.git thems/hexo-theme-next
```

最后一个参数是: 主题/主题建立的名字

克隆完毕后. 打开Hexo目录根下的`_config.yml`文件(可以把它称作**主站的配置文件**)

修改它的themes属性为你的主题名字:

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: hexo-theme-next
```

在git clone之后, 可以用git pull命令更新一下.

完毕后, 进入`Hexo/themes/主题` 目录下, 它的结构是这样的:
![这里写图片描述](https://ww1.sinaimg.cn/large/007rAy9hly1g1n8z5mlp4j30jo0det9q.jpg)
这里也有一个叫`_config.yml`的文件, 可以把它称作**主题配置文件**.



### 本地预览

在git bash中输入:

```shell
hexo g # 生成
hexo s # 启动服务

# 启动后
INFO Start processing
INFO Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

浏览器中输入<http://localhost:4000/> 就可以看到你的博客了.

到此, 这个博客已经搭建完毕. 之后就需要花些时间来研究这个主题.

 以及发挥你的想象力来自己定制这个主题.

主题的设置很多很多, 懒得写了, 我也只是配置了一部分. 

所以最好的就是参考NextT的官方文档, 这个文档简单明了.

<http://theme-next.iissnan.com/getting-started.html>

可以对主题和功能强大的第三方插件进行配置.



### 发表文章

主题配置完啦, 你也满意了. 接下来的就是日常操作, 例如写文章, 生成, 部署等等…

也懒得写了, 常用命令总结如下:

- `hexo new page “pageName”` : 新建页面
- `hexo new “blogName”` : 简写 hexo n #新建文章
- `hexo p` : 简写 hexo publish
- `hexo server` : 简写 hexo s #启动服务
- `hexo generate` : 简写 hexo g #生成静态网页
- `hexo deploy` : 简写 hexo d #部署

hexo命令详细的可以参考: <https://segmentfault.com/a/1190000002632530>

页面在新建时默认是.md格式, 也就是markdown格式的文档. 也是我非常喜欢的书写方式. 

在这里, 你只需要关注内容, 而不用过分关心格式. 只需要花上几分钟, 就可熟练掌握它基本的书写语法.

如果你觉得它的局限太多, 很难出来你想要的格式, 它也是支持html的, 你可以在里面加入html标签来做出你想要的格式.

关于markdown可以参考: [认识 Markdown](http://sspai.com/25137)



#### 问题记录

在`hexo deploy`时, 可能出现一个错误: `ERROR Deployer not found: git`
做这两步即可消除:

1. 输入: `npm install hexo-deployer-git –save`
2. 在`_config.yml`文件中将`deploy: type: github `修改为`deploy:type: git`



### 个性化域名

当在本地预览也调试好之后, 通过hexo deploy部署到了远程仓库, 就可以通过例如: `thank037.github.io`来访问你的博客. 如果你想要一个域名, 直接访问这个域名来访问你的个人博客, 就需要购买和设置一个域名.

国内和国外网站都可以购买域名. 个有好坏.

我是从[godaddy](https://sg.godaddy.com/) 上购买的. 虽然是国外的, 但是已经支持支付宝.

大致步骤就是:

- 搜索你想要的域名
- 选择一些购买项
- 填写个人信息
- 验证邮箱

购买完毕后登录godaddy, 进入my product页面. 可以看到你购买的域名信息.
![这里写图片描述](https://ww1.sinaimg.cn/large/007rAy9hgy1g1n943c824j30z60h2jsh.jpg)

点击DNS管理

对A记录, CNAME做基本的设置就可以了:
![这里写图片描述](https://ww1.sinaimg.cn/large/007rAy9hly1g1n963562oj30tm0gr3yy.jpg)

对于A记录, github pages目前是这两个(如果变了, 就要做更换):

> 192.30.252.153
> 192.30.252.154

下一步: 在你github上的博客仓库里面, 新建一个File, 取名CNAME, 里面只编辑一行(就是你购买的域名):

> www.xxx.com

然后访问这个域名, 就能看到了. 

至此, 个人独立博客搭建完毕.

因为是在github上, 访问的速度当然不如CSDN上的.

不过现在已经支持DNS配置访问选择距离你最近的服务器. 速度已经有所提升.
