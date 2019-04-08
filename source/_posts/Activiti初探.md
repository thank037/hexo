---
title: Activiti初探
date: 2017-11-15 12:39:04
categories: activiti
tags:
  - 工作流
  - activiti
---



### 前言

> Activiti是一个基于Java开源的工作流引擎, 实现了BPMN2.0规范.  <!-- more -->
>
> 提供了任流程的定义, 部署和流程调度.



这里很重要一个概念是工作流, 听起来很虚. 

如果接触过git workflow, 会了解到它应用的本质是使用Git来进行有效的代码流程管理和搞笑的开发协同约定. 

关于Activiti的由来和简介, 不在赘述.

关于理论, 可以参考:

> * 工作流(workflow): http://blog.csdn.net/u014682573/article/details/29922093
> * BPMN2.0规范: http://www.mossle.com/docs/jbpm4devguide/html/bpmn2.html



### 准备

**下载Activiti**

官网http://activiti.org/download.html

下载解压后的目录结构: 

- database: 提供一些数据库脚本.

- libs: Activiti所需要的jar包和源码包.

- wars: 官方提供的一些demo, 可以部署到tomcat下看看.



**安装Activiti Eclipse设计器**

在`Eclipse -> Help -> Install New Software -> Add`

> Name: Activiti BPMN 2.0 designer
>
> Location: http://activiti.org/designer/update/

当然也支持离线安装方式. 

此外还需要jdk6以上的环境, activiti支持多种主流数据库环境.



### 了解ProcessEngine

ProcessEngine: 流程引擎对象. 所有的操作都离不开它.

要了解ProcessEngine, 最好的方式是在你的项目先建立activiti project, 引入相关jar包. 

然后在其中编写junit, 来了解它的工作方式. 



#### 初始化数据库

整个流程引擎需要数据库表的支持, 不同版本可能数量不同, 我的是activiti-6.0.0版本, 需要28张表. 

初始化数据库的方式也有多种.

1. 执行脚本方式: 可在```activiti-6.0.0\database```目录下手动执行初始化脚本. 其中包括了建立数据库表和约束

2. 代码方式: 使用ProcessEngine来创建, 代码如下: 

   ``` java
   @Test
   public void createTable1(){
   	
   	ProcessEngineConfiguration processEngineConfiguration = ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
   	processEngineConfiguration.setJdbcDriver("com.mysql.jdbc.Driver");
   	processEngineConfiguration.setJdbcUrl("jdbc:mysql://192.168.118.128:3306/activiti_test?useUnicode=true&characterEncoding=utf8");
   	processEngineConfiguration.setJdbcUsername("root");
   	processEngineConfiguration.setJdbcPassword("root");
   	processEngineConfiguration.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
   	
   	ProcessEngine processEngine = processEngineConfiguration.buildProcessEngine();
   	System.out.println("processEngine1: " + processEngine);
   }
   ```

   工作流引擎配置也可以通过配置文件```activiti.cfg.xml```读取:

   ```java
   @Test
   public void createTable2(){
   	System.out.println("init table");
   	ProcessEngineConfiguration processEngineConfiguration = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("activiti.cfg.xml");
   	ProcessEngine processEngine = processEngineConfiguration.buildProcessEngine();
   	System.out.println("processEngine2: " + processEngine);
   }
   ```

   或者, 更简单, 使用默认方式获取ProcessEngine的方式:

   ```java
   /**
    * getDefaultProcessEngine()会调用init方法, 
    * 自动读取classpath下的activiti.cfg.xml文件来初始化数据库
    */
   ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
   ```

   配置文件```activiti.cfg.xml```:

   ```xml
   <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
   	<property name="jdbcDriver" value="com.mysql.jdbc.Driver" />
   	<property name="jdbcUrl" value="jdbc:mysql://192.168.118.128:3306/activiti_test?useUnicode=true&amp;characterEncoding=utf8" />
   	<property name="jdbcUsername" value="root" />
   	<property name="jdbcPassword" value="root" />
   	<property name="databaseSchemaUpdate" value="true" />
   </bean>  
   ```

完成之后, 查看数据库会看到ACT_开头的表



#### 关于表

表名的开头标识了该表业务含义: 

* RE: repository, 该类表包含流程定义和流程静态资源.
* RU: runtime, 该类表包含流程实例, 任务, 变量, 异步任务等运行中的数据.
* ID: identity,  该类表包含身份信息, 比如用户, 组.
* HI: history, 这些表包含历史数据, 比如历史流程实例, 变量, 任务等.
* GE: 表示通用数据, 用于不同场景下, 如存放资源文件.


```sql
# 部署对象, 流程定义相关
select * from act_re_deployment  #部署对象表

select * from act_re_procdef  #流程定义表

select * from act_ge_bytearray  #资源文件表

select * from act_ge_property  #主键生成策略表

###########################################

# 流程实例, 执行对象, 任务
select * from act_ru_execution  #正在执行的执行对象表

select * from act_hi_procinst  #流程实例的历史表

select * from act_ru_task  #正在执行的任务表(只有节点是UserTask的时候, 该表中存在数据)

select * from act_hi_taskinst  #任务历史表(只有节点是UserTask的时候, 该表中存在数据)

select * from act_hi_actinst  #所有活动节点的历史表

############################################
#流程变量
select * from act_ru_variable  #正在执行的流程变量表

select * from act_hi_varinst  #历史的流程变量表
```



#### 关于几个对象

- **ProcessDefinition**: 流程定义对象, 可以从这里获取到资源文件.

- **ProcessInstance**: 流程实例对象, 代表流程定义的执行实例

  一个流程实例包括了所有的运行节点. 可以利用这个对象来了解当前流程实例的进度等信息. 流程实例就表示一个流程从开始到结束的最大的流程分支, 即一个流程中的流程实例只有一个

- **Execution**: 执行对象. 按照流程定义的规则执行一次的过程.  

  Activiti用这个对象去描述流程执行的每一个节点. 在没有并发的情况下, Execution就如同ProcessInstance. 流程按照流程定义的规则执行一次的过程, 就可以表示执行对象Execution. 

对应的表: ACT_RU_EXECUTION, ACT_HI_PROCINST, ACT_HI_ACTINST

ProcessInstance extends Execution.

总结: 

* 一个流程中, 执行对象可以存在多个, 但是流程实例只能有一个.
* 当流程按照规则只执行一次的时候, 那么流程实例就是执行对象.



**Task**: 任务, 执行到某任务环节时生成的任务信息.

对应的表ACT_RU_TASK, ACT_HI_TASKINST

此外还有: ACT_RU_EXECUTION, ACT_HI_PROCINST, ACT_HI_ACTINST



#### 关于几个Service

* RepositoryService: 管理流程定义.
* RuntimeService: 执行管理, 包括启动,推进, 删除流程实例等操作.
* TaskService: 任务管理
* HistoryService: 历史管理(执行完的数据的管理)
* IdentityService: 组织机构管理
* FormService: 一个可选服务, 任务表单管理
* ManagerService





