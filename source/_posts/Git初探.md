---
title: Git初探
date: 2016-03-02 12:39:04
categories: 版本管理
tags:
  - Git
  - 工具
---

[TOC]

### 前言

早就听说了GitHub的强大. 一直没有机会去看, 在公司实习的几个月里也没机会接触SVN和Git <!-- more -->]

抱着对Linus大神的崇敬, 和开源的崇敬之情. 趁着不忙的几天来学习一下Git. 希望以后能够用到.



其实Git还是十分好学. 用不了多久, 你就能体会到它的高效简洁之美!

这里我是在本地虚拟机Centos来学习. . . 只是学习他的简单原理和操作, 并没有真正的尝试项目.

同时借鉴了网上一些有经验的前辈们的理解.. 来自己操作和学习

很多地方简单的例子我都有做图解,,做PPT真的好难阿!!!

并且花了两天时间来学习并整理这个小的文档.

希望能有所收获!

------

### 1. 介绍Git

一款免费, 开源, 高效的分布式版本控制系统

- [https://git-scm.com](https://git-scm.com/) 官网下载 支持命令行和GUI
- [https://gitref.org](https://gitref.org/) 官方的帮助文档.

**SVN和Git的主要区别:**

> SVN是集中式版本控制系统，版本库是集中放在中央服务器的，而干活的时候，用的都是自己的电脑
>
> 所以首先要从中央服务器哪里得到最新的版本，然后干活，干完后，需要把自己做完的活推送到中央服务器。
>
> Git是分布式版本控制系统，那么它就没有中央服务器的，每个人的电脑就是一个完整的版本库，这样，工作的时候就不需要联网了，因为版本都是在自己的电脑上。
>
> 既然每个人的电脑都有一个完整的版本库，那多个人如何协作呢？比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，
>
> 这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。

### 2. 全局配置git

- 安装 : sudo apt-get install git

- 查看版本: git –version

- 全局配置:

  ```
  > git config --global user.name xxx
  > git config --global user.email xxx
  > git config --global color.ui true
  ```

  配置用户名和邮箱是为了提交代码时的团队标识

> git config –list 查看git全局配置
>
> cat ~/.gitconfig 全局配置都保存在用户目录下这个.gitconfig隐藏文件中, 也可以vi编辑

可以通过以下命令获取git帮助

> git help

可以通过以下命令获取特定某个指定的的帮助

> git help 特定指令

可以查看仓库提交的历史记录

> git log

### 3. 创建Repository

初始化一个新的Git仓库

> git init : 初始化git目录环境(创建出新的空的repository) 并生成隐藏文件夹.git

.git(Git仓库文件, 所有相关的数据都保存在这儿)

```
> $ ls -A  查看到隐藏文件.git
> $ cd .git/
```

[![这里写图片描述](http://img.blog.csdn.net/20160325191910795?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325191910795?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

另一种方式:

```
git clone https://github.com/kennethreitz/requests.git
```

克隆网上的git项目,会自动创建repository

### 4. Git的对象类型

Git中有四种基本对象类型，组成了Git更高级的数据结构：

**● Blobs**
每个blob代表一个（版本的）文件，blob只包含文件的数据，而忽略文件的其他元数据，如名字、路径、格式等。

**● trees**

每个tree代表了一个目录的信息，包含了此目录下的blobs，子目录（对应于子trees），文件名、路径等元数据。因此，对于有子目录的目录，git相当于存储了嵌套的trees。

**● commits**

每个commit记录了提交一个更新的所有元数据，如指向的tree，父commit，作者、提交者、提交日期、提交日志等。每次提交都指向一个tree对象，记录了当次提交时的目录信息。一个commit可以有多个（至少一个）父commits。

**● tags**

tag用于给某个上述类型的对象指配一个便于开发者记忆的名字, 通常用于某次commit。

### 5. 提交及添加文件

> git status –查看git repository的状态

[![这里写图片描述](http://img.blog.csdn.net/20160325192627422?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325192627422?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
Untracked files: 没有登记的文件

a).添加

> git add hello.py命令进行添加

[![这里写图片描述](http://img.blog.csdn.net/20160325192642773?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325192642773?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

b).提交

> git commit -m ‘init commit’

(不加-m会默认打开vi, ‘ ’里面是操作的描述,这里的init commit表示首次提交,)

[![这里写图片描述](http://img.blog.csdn.net/20160325192720430?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325192720430?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**可以直接提交到仓库(不暂存)**

> git commit -a -m (或-am) “modify hello.txt”

注意:

1.其实不是在暂存区不存, 而是帮我们跳过(git add暂存区)这一步骤.

2.他不会自动提交 未追踪文件, 也就是新文件没有add过是不行的.

[![这里写图片描述](http://img.blog.csdn.net/20160325192735141?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325192735141?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

关于这三个文件状态及工作区域:

● History 也可以说 Git repository(git仓库):

最终确定的文件能保存到仓库, 成为一个新的版本, 并且对他人可见

● Staging area (暂存区,Cache, Index):

暂存已经修改的文件

● Working directory(工作区):

编辑, 修改文件

### 6. 查看git状态

[![这里写图片描述](http://img.blog.csdn.net/20160325192802407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325192802407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

> git status -s 列出简要状态信息

文件前面有两个flag. 第一个是Staging area, 第二个是Working directory

再次vi hello.py后查看状态:

[![这里写图片描述](http://img.blog.csdn.net/20160325192845501?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325192845501?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

出现两个M, 第一个M表示Staging area里的文件发生变化, 第二个M表示Working directory发生了变化.

两个区完全一致, 标志位就是空白, 有发生变化导致不一致, 标志位就是M

再次进行commit, 三个区完全一致. 两个标志位都为空

[![这里写图片描述](http://img.blog.csdn.net/20160325192859620?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325192859620?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### 7. 查看文件差别

> git diff 是查看第二个flag(也就是Staging area和Working directory) 具体变化信息的命令

[![这里写图片描述](http://img.blog.csdn.net/20160325192913949?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325192913949?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

在modify操作后可以用:

> git diff –staged 可以查看第一个flag(也就是Staging arae和History 之间的变化), 产生相同的效果
>
> git diff HEAD 可以看History 和 Working 之间的变化
>
> git diff –stat 后面加stat可以简化变化信息

### 8. 撤销误操作

[![这里写图片描述](http://img.blog.csdn.net/20160325192928814?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325192928814?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### 9. 移除及重命名文件

**删除文件**

1.系统级别删除

> rm filename

2.从git中删除文件

> git rm filename

3.提交操作

> git commit -m “delete filename”

注意: 只是删除当前版本中的文件, 文件依然被记录在Git仓库历史中

> git rm –cached filename

删除Staking arae中的文件, 文件还存在Working directory中, 可以通过git add 或 git reset 恢复

**重命名文件**

> git mv xxx new xxx
>
> git commit -m “rename xxx”

重命名相当于执行了以下三条命令:

> 1. mv xxx newxxx (先系统级别上改名mv)
> 2. git rm xxx
> 3. git add newxxx

### 10. 暂存工作区

[![这里写图片描述](http://img.blog.csdn.net/20160325192950627?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325192950627?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

假如出现这种情况(MM), 工作区修改完(修改1)后add到暂存区, 还未commit到仓库中, 此时发现一个遗漏或紧急情况, 需要在工作区中在修改1之前做修改(修改2), 但是又不想放弃修改1, 此时可以暂存工作区后进行修改.

**git stash 暂存工作区**
[![这里写图片描述](http://img.blog.csdn.net/20160325193002361?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193002361?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

此时就回到了修改一之前的现场, 此时可以做自己的代码修复vi

> git commit -am ‘quick fix’

修改完后就可以提交这个紧急修复
[![这里写图片描述](http://img.blog.csdn.net/20160325193015981?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193015981?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

> git stash list 查看暂存工作区的列表
> [![这里写图片描述](http://img.blog.csdn.net/20160325193028091?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193028091?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

可以看到一个

> git stash pop 弹出这个现场
> [![这里写图片描述](http://img.blog.csdn.net/20160325193043033?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193043033?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

然后在进行add和commit后, 修改1和修改2的作用就同时生效了.

### 11. 图解commit对象

为了理解HEAD,commit,和commit下对象之间的结构关系, 我们用图来理解:

[![这里写图片描述](http://img.blog.csdn.net/20160325193103518?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193103518?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

下面实际来看一下

git log 后, 出现最近的操作日志如下:

[![这里写图片描述](http://img.blog.csdn.net/20160325193130952?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193130952?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

每个commit后面都有一个哈希码来唯一标识这个commit.

HEAD默认指向第一个commit

> git cat-file -t : show object type
> [![这里写图片描述](http://img.blog.csdn.net/20160325193146546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193146546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
>
> git cat-file -p: print object’s content
> [![这里写图片描述](http://img.blog.csdn.net/20160325193158218?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193158218?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

当然, 用commit 前的哈希值也可以看到这个对象.
[![这里写图片描述](http://img.blog.csdn.net/20160325193218109?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193218109?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

这里就可以看到前面commit图解中的信息

找到这个commit对象了, 我们再来看看这个对象中的tree里面有什么.

[![这里写图片描述](http://img.blog.csdn.net/20160325193235207?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193235207?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

也可以继续追踪下去. 这样你就可以自由的找到commit对象里面的所有内容.

### 12. 理解tree-ish表达式

在.git文件中有一个HEAD

> cat HEAD
> ref: refs/heads/master

我们查看这儿

> cat refs/heads/master
> [![这里写图片描述](http://img.blog.csdn.net/20160325193258094?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193258094?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
>
> git log –oneline
> [![这里写图片描述](http://img.blog.csdn.net/20160325193312051?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193312051?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

可以看到, 这个master分支就指向的是最新的commit

> git rev-parse HEAD
> [![这里写图片描述](http://img.blog.csdn.net/20160325193327785?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193327785?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

可以看到, 这里HEAD和master的最终指向是一致的, 都是commit.

> git show 和 git cat -file -p类似, 能更简便的看到content

如果要定位到commit下的tree. 可以用tree-ish表达式

> HEAD ~3^{tree}
> [![这里写图片描述](http://img.blog.csdn.net/20160325193341407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193341407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

如果要定位到tree下的hello.py, 不用先定位到tree. 也可以用表达式

> HEAD~3:
> [![这里写图片描述](http://img.blog.csdn.net/20160325193357473?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193357473?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

tree-ish表达式的引入, 让我们更方便的定位到任意一个对象和文件

### 13. 创建及删除分支

- 查看分支:

> git branch
>
> – master

- 新建一个分支:

> git branch newBranch
> git branch
> – master
> newBranch

- 切换分支:

> git checkout newBranch
> 创建分支和切换分支可以合并一步进行:
> git checkout -b newBranch

- 删除分支

> git branch -d newBranch

注意: 切换分支实际就是把head指向了不同的branch

[![这里写图片描述](http://img.blog.csdn.net/20160325193416330?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193416330?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

cd heads目录, 就会发现有两个分支,master和newBranch

他们都指向一个commit

[![这里写图片描述](http://img.blog.csdn.net/20160325193431033?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193431033?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### 14. 合并分支(上)

这里我们介绍合并分支的做法, 和两种合并机制

我们在新创建的分支newBranch中进行代码的升级(hello.py)提交后,

再删除newBranch, 会出现一个error.

[![这里写图片描述](http://img.blog.csdn.net/20160325193446690?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193446690?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

这是因为newBranch原本指向的呢个新commit, 删除后, 这个新commit就没有被指向了. 如图:

[![这里写图片描述](http://img.blog.csdn.net/20160325193500190?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193500190?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

所以应该需要在删除newBranch之前让master指向新commit. 也就是进行分支合并, 合并后的效果如图:

[![这里写图片描述](http://img.blog.csdn.net/20160325193514709?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193514709?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

使用git merge newBranch

然后就可以删除newBranch了

这种合并机制叫Fast-forword

### 15. 合并分支(下)

如果出现这种情况: 在两个分支都出现了代码的修改

两个分支master和bugFix

在bugFix分支上做了两次fix (fix1和fix2) 并提交

[![这里写图片描述](http://img.blog.csdn.net/20160325193530519?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193530519?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

然后切换到master分支, 做一次Modify.

[![这里写图片描述](http://img.blog.csdn.net/20160325193540972?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193540972?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

进行git merge bugFix后, 两个分支的操作都被合并了

[![这里写图片描述](http://img.blog.csdn.net/20160325193550787?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193550787?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

但是此时的合并方式就不是Fast-forwrd那么简单了. 而是另一种机制3-way merge

它的合并原理如下:

[![这里写图片描述](http://img.blog.csdn.net/20160325193602613?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)](http://img.blog.csdn.net/20160325193602613?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

1. 先找到两个分支的共同分支对象

2. 将fix code2和B进行比对, 形成对比信息.

3. 然后将对比信息添加应用到Modify code上. 形成新的对象NEW.

   master重新指向NEW. 分支合并完毕!

### 16. Git远程仓库

远程仓库的实现:

1 使用现有的Git网络仓库服务

> Github: [https://github.com](https://github.com/) (创建开源免费哦)
>
> Bitbucket: [https://bitbucket.org](https://bitbucket.org/)

2 搭建自己的Git仓库服务器