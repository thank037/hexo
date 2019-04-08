---
title: 明日黄花Struts2
date: 2015-10-19 12:39:04
categories: Java
---

[TOC]

虽然Struts2已成为明日黄花 但还是决定来学习一下, 体会设计思想, 实用为辅.

<!-- more -->



#### Struts2 登录

> 需要注意: Struts2需要运行在JRE1.5及以上

1 创建Java Web项目

2 引入Struts2的依赖包. 将依赖包放在WEB-INFO的lib目录下

- commons-logging-1.0.4.jar
- freemarker-2.3.15.jar
- ognl-2.7.3.jar
- struts2-core-2.1.8.1.jar
- xwork-core-2.1.6.jar
- commons-fileupload-1.2.1.jar

3 在web.xml配置文件中, 配置StructPrepareAndExecuteFilter或FilterDispatcher

```xml
<filter>  
    <filter-name>struts2</filter-name>  
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>  
</filter>  
  
<filter-mapping>  
    <filter-name>struts2</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>
```

4 提供Struts2配置文件struts.xml, 放在src下.

5 建立JSP(login.jsp, login_success.jsp, login_error.jsp)

6 创建Struts2的Action, Struts2的Action可以不用继承Struts2框架中的任意一个类.
也不用实现Struts2框架中的任何接口, 所以Struts2的Action可以是一个POJO(纯粹的Java对象)
所以Struts2的Action测试更容易.
Struts2缺省方法名称: public String execute() throw Exception;

7 在Action中提供getter和setter方法, 便于收集数据(这样收集数据的模式一般可以称为属性驱动模式)

8 将JSP和Action配置到struts.xml配置文件中

9 了解structs-default.xml配置文件, default.properties, Struts2的默认后缀是: .action

- struts2-core-2.1.8.1.jar/structs-default.xml
- struts2-core-2.1.8.1.jar/org.apache.struts2/default.properties



#### Struts2的常用配置

1 标签的name属性, 如果不配置, 缺省为success.
2 Struts2提供了一个Action接口, 在Action接口中定义了一些常量和execute方法, 我们可以使用该接口, 使得开发更加规范.
3 常用配置参数: (在default.properties配置文件中能找到)

> struts.configuration.xml.reload=true
> –当struts.xml配置文件发生更改, 会立刻加载, 一般在开发环境, 在生产环境别配 * struts.devMode=true
> – 会提供更加友好的错误提示

以上配置方式有两种:

- 在struts.properties中配置
- 在struts.xml配置文件中(建议)



#### Struts2对团队开发的支持(多配置文件)

1 可以为某个模块建立单独的配置文件, 该配置文件的格式需要和struts.xml文件的格式一样
2 在struts.xml文件中用标签引入



#### Struts2对ModelDriven模式的支持

Struts2可以采用类似于Struts1中的ActionForm方式收集数据, 这样的方式叫做ModelDriven模式

**如何实现模型驱动模式?**

- 创建User
- Action需要实现ModelDriven接口
- 实现getModel()方法, 返回Bean对象



#### Struts2中直接对Action中的对象进行赋值(不new)

在html中可以通过如下方式命名输入域:

```html
<form action="login.action">  
用户：<input type="text" name="user.username"><br>  
密码：<input type="password" name="user.password"><br>  
<input type="submit" value="登录">  
</form>
```



#### Struts2的Action访问Servlet API

**方式一:**
1 可以通过ActionContext访问Servlet API，此种方式没有侵入性

2 在Struts2中默认为转发，也就是标签中的type=”dispatcher”，type的属性可以修改为重定向. Struts的重定向有两种：

- type=”redirect”,可以重定向到任何一个web资源，如：jsp或Action 如果要重定向到Action，需要写上后缀：xxxx.action
- type=”redirectAction”,可以重定向到Action，不需要写后缀，此种方式更通用些,不会因为后缀的改变影响配置

3 关于Struts2的type类型，也就是Result类型，他们都实现了共同的接口Result，都实现了execute方法

他们体现了策略模式，具体Result类型参见：struts-default.xml文件：

```xml
<result-types>  
   <result-type name="chain" class="com.opensymphony.xwork2.ActionChainResult"/>  
   <result-type name="dispatcher" class="org.apache.struts2.dispatcher.ServletDispatcherResult" default="true"/>  
   <result-type name="freemarker" class="org.apache.struts2.views.freemarker.FreemarkerResult"/>  
   <result-type name="httpheader" class="org.apache.struts2.dispatcher.HttpHeaderResult"/>  
   <result-type name="redirect" class="org.apache.struts2.dispatcher.ServletRedirectResult"/>  
   <result-type name="redirectAction" class="org.apache.struts2.dispatcher.ServletActionRedirectResult"/>  
   <result-type name="stream" class="org.apache.struts2.dispatcher.StreamResult"/>  
   <result-type name="velocity" class="org.apache.struts2.dispatcher.VelocityResult"/>  
   <result-type name="xslt" class="org.apache.struts2.views.xslt.XSLTResult"/>  
   <result-type name="plainText" class="org.apache.struts2.dispatcher.PlainTextResult" />  
</result-types>
```

我们完全可以自己根据需求扩展Result类型

4 全局Result和局部Result

**方式二:**

可以通过实现装配接口, 完成对Servlet-API的访问

```
* ServletRequestAware取得HttpServletRequest对象  
* ServletResponseAware取得HttpServletResponse对象  
* ServletContextAware取得ServletContext对象(工具类)
```

**方式三:**
可以通过ServletActionContext提供的静态方法取得Servlet API

> - getPageContext();
> - getRequest();
> - getResponse();
> - getServletContext();



#### Struts2命名空间设置:

- 采用命名空间, 可以区分不同包下的相同Action名称.
- 如果package的namespace属性没有设定, 使用默认的命名空间为””.
- Struts2中Action的完整路径为: namespace + Action的名称.
- 寻找原则: 首先在指定的命名空间下寻找Action,　如果找到了就使用此Action, 如果没有找到, 就在上层目录中查找, 一直到根(缺省命名空间), 找到则使用此Action, 没找到就抛出Action没找到异常.



#### Struts2的字符集设置:

```xml
* struts.properties配置文件中: struts.i18n.encoding  
* struts.xml配置文件:   
<constant name="struts.i18n.encoding" value="GB18030"/>  
* 在StrutsPrepareAndExecuteFilter中配置:  
<filter>  
        <filter-name>struts2</filter-name>  
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>  
        <init-param>  
            <param-name>struts.i18n.encoding</param-name>  
            <param-value>GB18030</param-value>  
        </init-param>  
</filter>
```

注: 这个配置只针对表单提交为post方式, 如果提交方式为get, 则需要在tomcat_home/conf/server.xml文件中
在8080port呢个位置后面加个URIEncoding=”GB18030”.



#### Struts2的Action包含多个方法时如何调用 (方法的动态调用)

具体的调用方式:
1 方法的动态调用
2 在中配置method属性
3 使用通配符

方式一: 方法的动态调用

```
[action名称] + ! + 方法名称 + 后缀  
如: <a href="user!add.action">添加用户</a><br>  
<a href="user!del.action">删除用户</a><br>  
<a href="user!update.action">修改用户</a><br>  
<a href="user!list.action">查询用户</a><br>
```

Action不用实现Action接口, 只需继承ActionSupport(它实现了Actio接口)

动态调用的参数配置, 默认为true, 可以调用, 为false表示禁用:

```xml
<constant name="struts.enable.DynamicMethodInvocation" value="false" />
```

Action中的所有方法最好和execute方法一致(参数, 返回值, 异常 最好都一致)

方式二: 在标签中配置method属性

添加method属性, jsp页面要写name名, 而不是method名

```xml
<pre name="code" class="html"><!--        
    <package name="user-package" extends="struts-default">  
        <action name="add" class="com.xie.struts2.UserAction" method="add">  
            <result>/success.jsp</result>  
        </action>  
        <action name="delete" class="com.xie.struts2.UserAction" method="delete">  
            <result>/success.jsp</result>  
        </action>  
        <action name="update" class="com.xie.struts2.UserAction" method="update">  
            <result>/success.jsp</result>  
        </action>  
        <action name="list" class="com.xie.struts2.UserAction" method="list">  
            <result>/success.jsp</result>  
        </action>  
    </package>  
-->  
       
     <package name="user-package" extends="struts-default">  
        <action name="addUser" class="com.xie.struts2.UserAction" method="add">  
            <result>/success.jsp</result>  
        </action>  
        <action name="deleteUser" class="com.xie.struts2.UserAction" method="delete">  
            <result>/success.jsp</result>  
        </action>  
        <action name="updateUser" class="com.xie.struts2.UserAction" method="update">  
            <result>/success.jsp</result>  
        </action>  
        <action name="listUser" class="com.xie.struts2.UserAction" method="list">  
            <result>/success.jsp</result>  
        </action>  
    </package>
```

方式三: 使用通配符

具体配置:

```xml
<action name="*User" class="com.xie.struts2.UserAction" method="{1}">  
<result>/{1}Success.jsp</result>  
</action>
```

在Struts2的标签中的class, name, method和result标签上都可以使用通配符
通配符的作用就是为了减少配置量, 通配符需要建立在约定的基础上.



#### Struts2的上传

1 Struts2默认采用了apache commons-fileupload.
2 Struts2支持三种类型的上传组件
3 需要引入commons-fileupload相关依赖包

- commons-io-1.3.2.jar
- commons-fileupload-1.2.1.jar

4 表单中需要采用post提交方式, 编码方式需要是: multipart/form-data
5 Struts2的Action

- 取得文件名称: 输入域的名称 + 固定字符串”FileName”
- 取得文件数据: File输入域的名称
- 取得内容类型: 输入域的名称 + 固定字符串”ContentType”

6 得到输入流, 通过输出流写出文件



#### Struts2的类型转换

**如何实现Struts2的类型转换器?**
1 继承StrutsTypeConverter
2 覆盖convertFromString和convertToString方法

**注册类型转换器:**
这里我们手动编写一个转换日期的转换器.

**局部类型转换器:**
只对当前Action起作用, 需要提供如下配置文件:

> [MyActionName]-conversion.properties
> –[MyActionName]指需要使用转换器的Action名称(不是转换器的名称)

–conversion.properties”固定字符串, 不能修改.

例如: 我们addUserAction类型转换器的配置文件名称应该为: AddUserAction-conversion.properties.
该配置文件一定要和该Action放在同一个目录中, 该配置文件的格式为:Action中的属性名称=转换器的完整路径
如: birthday=com.xie.struts2.UtilDateConverter

**全局类型转换器:**
可以对所有的Action起作用(同Struts1的类型转换器), 需要提供如下配置文件:
xwork-conversion.properties, 该配置文件需要放在src目录下
该配置文件的格式: 需要转换的类型的完整路径=转换器的完整路径.
如: java.util.Date=com.xie.struts2.UtilDateConverter

如果全局类型转换器和局部类型转换器同时存在, 则局部优先.

Struts2的标签库就一个, 类似于JSTL, 默认以s开头.

```
引入: <%@ taglib uri="/struts-tags" prefix="s"%>  
读取: <s:property value="birthday" />
```

采用标签读取属性, 才能调用转换器的convertToString方法, 采用JSTL就不会.



#### Struts2对国际化的支持

1 Locale是由国家和语言代码构成的
2 国际化资源文件是由: baseName + locale.properties
3 国际化的内容:

- 在页面中的硬编码
- 动态文本(提示或错误)

4 配置baseName:

```xml
<constant name="struts.custom.i18n.resources" value="Message"/>
```

5 提供国际化资源文件:

- Message.properties
- Message_zh_CN.properties
- Message_en_US.properties

6 在开发阶段, 可以进行如下配置:

当国际化资源文件发生修改, 则立刻加载, 在生产环境下一般不要配置

```xml
<constant name="struts.i18n.reload" value="true"/>
```



#### Struts2对异常的支持(声明式异常, 自动的异常处理)

局部异常 配置:

```xml
<exception-mapping result="error" exception="com.xie.struts2.ApplicationException" />  
<result name="error">/login_error.jsp</result>
```

全局异常 配置:

```xml
<global-results>  
<result name="global-error">/global_error.jsp</result>  
</global-results>  
<global-exception-mappings>  
<exception-mapping result="global-error" exception="com.xie.struts2.ApplicationException"></exception-mapping>  
</global-exception-mappings>
```

在页面可以使用EL取得异常信息

```
${exception.message} --自定义错误信息
${exceptionStack}    --异常信息堆栈
```



#### Struts2的拦截器

1 Struts2的拦截器只能拦截Action，拦截器是AOP的一种思路，可以使我们的系统架构
更松散（耦合度低），可以插拔，容易互换，代码不改变的情况下很容易满足客户需求其实体现了OCP

2 如何实现拦截器？（整个拦截器体现了责任链模式，Filter也体现了责任链模式）

- 继承AbstractInterceptor（体现了缺省适配器模式）
- 实现Interceptor

3 如果自定了拦截器，缺省拦截器会失效，必须显示引用Struts2默认的拦截器

4 拦截器栈，多个拦截器的和

5 定义缺省拦截器,所有的Action都会使用

6 拦截器的执行原理，在ActionInvocation中有一个成员变量Iterator，这个Iterator中保存了所有拦截器，
每次都会取得Iterator进行next,如果找到了拦截器就会执行，否则就执行Action，都执行完了
拦截器出栈（其实出栈就是拦截器的intercept方法弹出栈）