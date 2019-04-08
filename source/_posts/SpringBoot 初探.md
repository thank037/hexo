---
title: Spring Boot 初探
date: 2016-11-01 12:39:04
categories: Spring Boot系列
tags:
  - Spring
  - Spring Boot
---


### 前言

工作中要用到Spring Boot和Spring Cloud了! <!-- more -->

但是现在网上关于Spring Boot成体系的东西真的太少了, 只找到了翟勇超写的一个系列

下面对Spring Boot中我认为比较特色的归纳




### 启动类

`@SpringBootApplication` 开启组件扫描和自动配置. 其中包含三个常用注解:

1. `@Configuration` : 表明基于Java配置

2. `@ComponentScan` : 开启组件扫描, 自动发现控制器类和其他组件并注册为应用程序上下文的Bean

3. `@EnableAutoConfiguration` : 开启Spring Boot自动配置功能

  所以它一般加在需要启动引导和自动配置的类

负责启动引导应用程序

```java 
SpringApplication.run(Object source, String… args); 
```



### 一 起步依赖

Gradle和Maven都是Spring Boot为应用程序提供的构建工具

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>{springBootVersion}</version>
	<relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
	...
</dependencies>
```

父起步依赖:从spring-boot-starter-parent中继承版本号, 把它作为上一级, 利用Maven的依赖管理.

所以在下面的dependency中就不用写版本了.

> 除了手工添加的一些以外, 其他是’spring-boot-starter-‘打头的都是Spring Boot的起步依赖, 有助于我们程序的构建, 如果没有这些依赖, 你可能需要考虑需要哪个功能需要哪些模块, 它们之间是否兼容等等问题. 事实上我们需要关注的, 仅仅是功能而已. 所以, 通过传递依赖, 就=加了一大堆需要的库.

此外, 还支持你排除和覆盖某个起步依赖



### 二 自动配置

Spring Boot的自动配置是一个运行时（更准确地说，是应用程序启动时）的过程.

每当应用程序启动的时候，Spring Boot的自动配置都要做将近200个这样的决定，涵盖安全、

集成、持久化、Web开发等诸多方面。所有这些自动配置就是为了尽量不让你自己写配置。

**自动配置** 让我们专注于应用程序代码



### 三 条件化配置

spring-boot-autoconfigure中提供了用于Spring MVC, Spring Data JPA, Thymeleaf等等的各种配置.

这些配置存在于应用程序配置当中, 但可以根据你的需要在满足某个条件时忽略这个配置.

例如要对JdbcTemplate进行自动配置(该条件为:只有在Classpath里存在 JdbcTemplate 时才会生效)

- 首先实现Condition接口, 覆盖Matches() method

  ```java
  public class JdbcTemplateCondition implements Condition {
  	@Override
  	public boolean matches(ConditionContext context,AnnotatedTypeMetadata metadata) {
  		try {
  			context.getClassLoader().loadClass("org.springframework.jdbc.core.JdbcTemplate");
  			return true;
  		} catch (Exception e) {
  			return false;
  		}
  	}
  }
  ```

- 再来声明Bean时

  ```java
  @Conditional(JdbcTemplateCondition.class)
  	public MyService myService() {
  	...
  }
  ```


表示只有当JdbcTemplateCondition类中的条件成立(classpath中有JdbcTemplate)后才能创建myService这个Bean.

`@Conditional`只是Spring Boot中简单的一个注解条件, 除此之外还有很多丰富的条件注解.

Spring Boot自动配置承担起了配置Spring的重任，因此你能专注于编写自己的应用程序。



### 四: 自定义配置

#### 4.1

例如我们要覆盖Spring Secuirty的安全配置,首先要创建自定义的安全配置:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {...} 就行了
```

这么简单? 什么原理呢?

举个例子Spring Boot的 DataSourceAutoConfiguration 中定义的 JdbcTemplate Bean

```java
@Bean
@ConditionalOnMissingBean(JdbcOperations.class)
public JdbcTemplate jdbcTemplate() {
	return new JdbcTemplate(this.dataSource);
}
```

Spring Boot会跳过自动的安全配置，转而使用我们的SecurityConfig

还记得标题三中通过注解的条件配置, 其中`@ConditionalOnMissingBean`注解是覆盖自动配置的关键.

要求当前不存在 JdbcOperations类型（ JdbcTemplate 实现了该接口）的Bean时才生效。

如果当前已经有一个 JdbcOperationsBean了，条件即不满足，不会执行 `jdbcTemplate()` 方法。

**什么情况下有一个了呢?**

Spring Boot的设计是加载应用级配置， 随后再考虑自动配置类。

因此，如果你已经配置了一个 JdbcTemplate Bean，那么在执行自动配置时就已经存在一个 JdbcOperations 类型的Bean了，于是忽略自动配置的 JdbcTemplate Bean。



#### 4.2(常用:自动配置微调)

Spring Boot自动配置的Bean提供了300多个用于微调的属性, 而且能从多种属性源获得属性.

其中就包括`application.properties`呀.



#### 4.3(Profile配置)

如果一个自定义的安全验证只用在生产环境,怎么做?

- step1.在你自定义的配置类上加入`@Profile(value=”production”)`
- step2.在`application.properties中`加入`spring.profiles.active=production`

注意: 如果没有找到. 这个自定义的安全配置就会被忽略!

如果每个环境分别需要不同的配置,怎么做?

例如生产环境中，你只关心 WARN 或更高级别的日志项，想把日志写到日志文件里。

在开发环境中，你只想把日志输出到控制台，记录 DEBUG或更高级别。

创建额外的properties文件, 遵守`application-{profile}.properties`的命名规范:

```properties
application-development.properties 开发环境
logging.level.root=DEBUG
application-production.properties 生产环境
logging.path=/var/logs/
logging.file=BookWorm.log
logging.level.root=WARN
```