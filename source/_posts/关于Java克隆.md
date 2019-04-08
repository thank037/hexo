---
title: 关于Java克隆
date: 2016-11-18 12:39:04
categories: Java
tags:
  - Java
---

补充关于Java克隆的一点点东西
<!-- more -->

[TOC]

### 开始

先来看一段代码

```java
int a = 1;
int b = a;
b = 2;
System.out.println(a);  //1
System.out.println(b);  //2
```
a的值并没有因为复制给b而改变.

这就实现了b对a的一份拷贝. 对于byte, short, char等基本数据类型, 也同样适用. (可以理解为深拷贝)

但是如果要复制一份对象, 就无法适用了. (可以理解为浅拷贝)



**何为深浅呢?**
一个重要的标识:是否完全复制一个新的对象，需要申请新的内存空间

对象的直接拷贝只是两个在栈不同的引用指向堆中的同一块存储空间.

所以一个引用的操作是会影响原数据的.这就是浅拷贝.

> 可见，对于基本类型，都是深拷贝，这由栈处理变量的机制有关；
>
> 对于自定义的对象，由于引用和对象空间分配在不同的存储中，取决于你在拷贝时是否选择在堆中去申请新的对象空间。
>
> 对于String，Integer这种，如果直接赋值，String str = “qwqw” 遵循基本类型，如果利用new去创建，遵循自定义对象。



### 浅拷贝

```java
package com.xie.core.base;

import java.io.IOException;

class School {

	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}

class Student implements Cloneable {

	private School school;

	private String name;

	public School getSchool() {
		return school;
	}

	public void setSchool(School school) {
		this.school = school;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	/**
	 * 浅拷贝
	 */
	 @Override
	 protected Object clone() throws CloneNotSupportedException{
	 return super.clone();
	 }

}

public class CloneTest01 {
	
	public static void main(String[] args) throws CloneNotSupportedException, ClassNotFoundException, IOException {
		
		School school = new School(); school.setName("Thank School");
		
		Student student1 = new Student(); student1.setName("Thank"); 
		student1.setSchool(school); 
		
		Student student2 = (Student) student1.clone();
		student2.setName("Chunhua");
		student2.getSchool().setName("Chunhua School");

		System.out.println(student1.getName() + ", " + student1.getSchool().getName());
		System.out.println(student2.getName() + ", " + student2.getSchool().getName());
		
	}
}
```

输出结果为:

```
Thank, Chunhua School
Chunhua, Chunhua School
我分别打印了school的toString():
student1 --> school: com.xie.core.base.School@762efe5d
student2 --> school: com.xie.core.base.School@762efe5d
```

可以看到, student2的拷贝只是对String类型name实现了拷贝, 而对School对象没有实现拷贝
这就是浅拷贝.



### 深拷贝

再来看看深拷贝

```java
package com.xie.core.base;

import java.io.IOException;

class School implements Cloneable{

	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	/**
	 * 深拷贝
	 */
	@Override
	protected Object clone() {
		Object o = null;
		try {
			o = super.clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}
		return o;
	}
}

class Student implements Cloneable{

	private School school;

	private String name;

	public School getSchool() {
		return school;
	}

	public void setSchool(School school) {
		this.school = school;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	/**
	 * 深拷贝
	 */
	@Override
	protected Object clone() throws CloneNotSupportedException {
		Student o = null;
		try {
			o = (Student) super.clone();
			// 调用School的clone方法实现深拷贝
			o.school = (School) school.clone();
		} catch (Exception e) {
			e.printStackTrace();
		}
		return o;
	}

}

public class CloneTest01 {
	
	public static void main(String[] args) throws CloneNotSupportedException, ClassNotFoundException, IOException {
		
		School school = new School(); school.setName("Thank School");
		
		Student student1 = new Student(); student1.setName("Thank"); 
		student1.setSchool(school); 
		
		Student student2 = (Student) student1.clone();
		student2.setName("Chunhua");
		student2.getSchool().setName("Chunhua School");

		System.out.println(student1.getName() + ", " + student1.getSchool().getName());
		System.out.println(student2.getName() + ", " + student2.getSchool().getName());
		
	}
}
```

```
输出结果为:
Thank, Thank School
Chunhua, Chunhua School

我分别打印了school的toString():
student1 --> school: com.xie.core.base.School@762efe5d
student2 --> school: com.xie.core.base.School@41a4555e
```

是两个不同的对象. 符合’完全复制一个新的对象，需要申请新的内存空间’ 的定义, 这就是深拷贝



### 序列化拷贝

再来介绍一种实现深度拷贝的方法. 利用序列化实现对象的拷贝

> 把对象写到流里的过程是串行化（Serilization）过程，又叫对象序列化，
>
> 而把对象从流中读出来的（Deserialization）过程叫反序列化。
>
> 应当指出的是，写在流里的是对象的一个拷贝，而原对象仍然存在于JVM里面，
>
> 因此在Java语言里深复制一个对象，常常可以先使对象实现Serializable接口，
>
> 然后把对象（实际上只是对象的一个拷贝）写到一个流里，再从流里读出来便可以重建对象。

此时不用实现Cloneable的clone(), 但是必须要实现Serializable接口

```java
package com.xie.core.base;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

class School implements Serializable { //实现序列化接口

	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}

class Student implements Serializable{ //实现序列化接口

	private School school;

	private String name;

	public School getSchool() {
		return school;
	}

	public void setSchool(School school) {
		this.school = school;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	/**
	 * 串行化深复制
	 */
	public Object deepClone() throws IOException, ClassNotFoundException{
		ByteArrayOutputStream bo = new ByteArrayOutputStream();
		ObjectOutputStream oos = new ObjectOutputStream(bo);
		oos.writeObject(this);
		
		ByteArrayInputStream bi = new ByteArrayInputStream(bo.toByteArray());
		ObjectInputStream osi = new ObjectInputStream(bi);
		
		return osi.readObject();
	}
}

public class CloneTest01 {
	
	public static void main(String[] args) throws CloneNotSupportedException, ClassNotFoundException, IOException {
		
		School school = new School();
		school.setName("Thank School");
		
		Student student1 = new Student();
		student1.setName("Thank");
		student1.setSchool(school); 
		
		Student student2 = (Student) student1.deepClone();
		student2.setName("Chunhua");
		student2.getSchool().setName("Chunhua School");

		System.out.println(student1.getName() + ", " + student1.getSchool().getName());
		System.out.println(student2.getName() + ", " + student2.getSchool().getName());
		
	}
}
```



### 总结

在内存中通过字节流的拷贝是比较容易实现的。把母对象写入到一个字节流中，再从字节流中将其读出来，这样就可以创建一个新的对象了，并且该新对象与母对象之间并不存在引用共享的问题，真正实现对象的深拷贝。