---
title: Oracle错误排除
date: 2013-12-22 12:39:04
categories: 远古笔记
tags:
  - 数据库
---

这两天使用Oracle遇到很多问题, 大都容易解决

<!-- more -->

#### 问题

这个错误是:

```
ERROR: ORA-01034:ORACLE not available ORA-27101:shared memory realm does not exit
```



#### 排查步骤

排查步骤如下: 

1. 先看oracle的监听和oracle的服务是否都启动了。启动oracle监听：
   cmd的命令行窗口下，输入：`lsnrctl start`，回车即启动监听

2. 查看oracle的sid叫什么，比如创建数据库的时候，实例名叫“abc”，那么先手工设置一下oralce的sid

   cmd命令窗口中: `set ORACLE_SID=abc`

3. 再输入`sqlplus /nolog` 回车

4. 再输入`conn / as sysdba` 回车

5. 再输入`startup` ，回车。 

   这步是启动oracle服务。如果startup启动被告知已经启动了，可以先输入`shutdown immediate`；等shutdown结束之后，再输入startup。

6. 过几秒钟等命令运行完成，就能连接了。这个时候，可以输入`select count(*) from user_tables;` 测试一下，看是否有查询结果。



#### 总结

出现`ORA-01034`和`ORA-27101`的原因是多方面的：主要是oracle当前的服务不可用，`shared memory realm does not exist`，是因为oracle没有启动或没有正常启动，共享内存并没有分配给当前实例.所以，通过设置实例名，再用操作系统身份验证的方式，启动数据库。这样数据库就正常启动了，就不会报ORA-01034和ORA-27101两个启动异常了。