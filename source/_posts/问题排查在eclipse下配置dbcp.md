---
title: 问题排查:在eclipse下配置dbcp
date: 2015-10-06 12:39:04
categories: 问题排查
---


### 配置

**使用jndi的方式配置tomcat数据源.**

<!-- more -->

我的步骤如下: 

1. 将Oracle的驱动拷贝到tomcat中的lib目录下

2. 配置`web.xml`文件(这一步尝试了一下不写也暂未出现异常. 原因未知…)

   ```xml
   <resource-ref>  
     <description>dbcp_drp</description>  
     <!--数据源名称, 要和context.xml中的数据源名称一致-->  
     <res-ref-name>jdbc/drp</res-ref-name>    
     <res-type>javax.sql.DataSource</res-type>  
     <res-auth>Container</res-auth>    
   </resource-ref>
   ```

3. 配置`context.xml`文件

   修改了该文件的内容, tomcat就会自动装载该应用.
   为了满足每个应用的单独配置, 所以我是在我的一个项目中的META-INFO中配置的

   配置内容如下:

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>  
   <Context>  
   <Resource  
         name="jdbc/drp"  
         type="javax.sql.DataSource"  
         password="drp"  
         driverClassName="oracle.jdbc.driver.OracleDriver"  
         maxIdle="2"  
         maxWait="5000"  
         username="drp"  
         url="jdbc:oracle:thin:@localhost:1521:orcl"  
         maxActive="4"/>  
   </Context>
   ```

   可以使用Tomcat管理页面配置, 但是个人感觉太麻烦…

4. 在jsp中的Java代码测试

   ```java
   Connection conn = null;  
   Context ctx = new InitialContext();  
   //通过JNDI查找DataSource  
   DataSource ds = (DataSource)ctx.lookup("java:comp/env/jdbc/drp");  
   conn = ds.getConnection();
   ```

按照科学, 这样就得到连接啦…

但是在eclipse下启动Tomcat时. 却出现了几个问题.. 花了几小时去找原因.



### 遇到问题

#### 问题一

```
[SetPropertiesRule]{Server/Service/Engine/Host/Context} Setting property ‘source’ to ‘org.eclipse.jst.jee.server:aa’ did not find a matching property
……………
……………
```

这导致tomcat根本没法启动

大致阅读, 发现有可能是server.xml配置中的source属性的问题

于是我进入`tomcat_home/conf/server.xml`查看, 确实自动加了一段代码标签

然后我尝试删除这段代码, 重新启动tomcat.

发现能启动了, 但却出现另一个严重问题:

#### 问题二

```
Cannot create JDBC driver of class ‘’ for connect URL ‘null’
………
………
```

为什么JDBC的DriverClass和url都为空, 明明在`meta-info`中都配置好了啊.为什么没取到? 

还是jndi数据源读取出现错误?

于是我怀疑他读到的是`tomcat_home/conf/context.xml`这个, 而不是我应用中的`context.xml`

于是将`tomcat_home/conf/`下的`context.xml`删除.发现还是不行

显然, 这种解决方式很愚蠢. 

于是, 我去寻找问题的原因.

最后找到原因如下:

在eclipse下, 启动tomcat时, tomcat的配置文件conf/server.xml中会自动生成一个关于该web工程的配置项信息.类似:

```xml
docBase="" path="" reloadable="" source=""/>
```

而默认情况下, server.xml的Context元素不支持名称为source的属性, 所以产生了警告.



##### 解决办法

点开eclipse下配置好的tomcat, 在打开的页面找到server option选项

选中`Publish modual contexts to separat XML files`选项即可, 就是将context部分放到一个单独的文件中.

此时, 再修改meta-info中的`context.xml`中的配置

`tomcat_home/conf`下的`server.xml`中就不会再出现和什么source属性了.

警告也消失了!