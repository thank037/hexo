---
title: freemarker导出复杂Excel
date: 2017-04-20 12:39:04
categories: 工作
tags:
  - freemarker
---

[TOC]

### 序言

用Freemarker做Excel导出确实很容易. 但是导出复杂Excel, 例如多行合并的还是费了一天时间

<!-- more -->



### 步骤

首先是pom依赖, 构建工具使用Maven

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```



#### 准备模板

接下来, 制作ftl模板

在Office中编辑一个Excel文件, 这就是最后要生成的模板.

![这里写图片描述](https://ww1.sinaimg.cn/large/007rAy9hly1g1n7bcb7hrj309402cmwz.jpg)

 

>  注意: 另存为xml格式. 不要直接改后缀名

拷贝到项目下, 修改后缀名为.ftl (这里我放到`resources/templates`目录下)

用xml方式打开这个.ftl文件. 找到如下代码

```xml
<Table ss:ExpandedColumnCount="4" ss:ExpandedRowCount="3" x:FullColumns="1"
x:FullRows="1" ss:DefaultColumnWidth="54" ss:DefaultRowHeight="13.5">
	<Row>
		<Cell ss:MergeAcross="3" ss:StyleID="s66"><Data ss:Type="String">人员列表</Data></Cell>
	</Row>
	<Row>
		<Cell ss:StyleID="s67"/>
		<Cell ss:StyleID="s67"><Data ss:Type="String">name</Data></Cell>
		<Cell ss:StyleID="s67"><Data ss:Type="String">age</Data></Cell>
		<Cell ss:StyleID="s67"><Data ss:Type="String">address</Data></Cell>
	</Row>
	<Row>
		<Cell ss:StyleID="s67"><Data ss:Type="Number">1</Data></Cell>
		<Cell ss:StyleID="s67"><Data ss:Type="String">zhangsan</Data></Cell>
		<Cell ss:StyleID="s67"><Data ss:Type="Number">22</Data></Cell>
		<Cell ss:StyleID="s67"><Data ss:Type="String">BeiJing</Data></Cell>
	</Row>
</Table>
```

主要看上面这段, 对比你的Excel很容易看出: 每个就是一行, 每个是一个单元格

找到要循环的一行, 添加表达式和标签

> 关于Freemarker的语法, 可以参考<http://freemarker.foofun.cn/index.html>

加完标签后如下:

```xml
<Table ss:ExpandedColumnCount="4" ss:ExpandedRowCount="${userListSize}" x:FullColumns="1"
x:FullRows="1" ss:DefaultColumnWidth="54" ss:DefaultRowHeight="13.5">
<Row>
	<Cell ss:MergeAcross="3" ss:StyleID="s66"><Data ss:Type="String">人员列表</Data></Cell>
</Row>
<Row>
	<Cell ss:StyleID="s67"/>
	<Cell ss:StyleID="s67"><Data ss:Type="String">姓名</Data></Cell>
	<Cell ss:StyleID="s67"><Data ss:Type="String">年龄</Data></Cell>
	<Cell ss:StyleID="s67"><Data ss:Type="String">地址</Data></Cell>
</Row>

<#assign index = 0 >
<#list userList as u>
<#assign index = index + 1 >
   <Row>
	<Cell ss:StyleID="s67"><Data ss:Type="Number">${index}</Data></Cell>
	<Cell ss:StyleID="s67"><Data ss:Type="String">${u.name}</Data></Cell>
	<Cell ss:StyleID="s67"><Data ss:Type="Number">${u.age}</Data></Cell>
	<Cell ss:StyleID="s67"><Data ss:Type="String">${u.address}</Data></Cell>
   </Row>
</#list>
</Table>
```

注意ExpandedRowCount的值可以变量传进来, 如果不好计算可以写一个很大的值.

自己写的值有什么影响我没测试…



#### 准备数据源

对于你要动态生成的List, 里面可以放Map, 也可以放对象, 都行.

这里我先用Map, 等下用对象

- 模拟获取数据

```java
/**
 * 构造user数据List<Map<String, Object>>
 */
private static List<Map<String, Object>> getUserList(){
	List<Map<String, Object>> returnList = new ArrayList<Map<String, Object>>();
	Map<String,Object> map1 = new HashMap<String, Object>();
	Map<String,Object> map2 = new HashMap<String, Object>();
	map1.put("name", "张三");
	map1.put("age", "18");
	map1.put("address", "广东");
	
	map2.put("name", "王五");
	map2.put("age", "22");
	map2.put("address", "北京");
	
	returnList.add(map1);
	returnList.add(map2);
	return returnList;
}
```



- 控制器

```java
@SuppressWarnings("unchecked")
@RequestMapping(value="/exportExcel", method=RequestMethod.GET)
public void exportExcelByFreeMarker(HttpServletRequest request, HttpServletResponse response) {
	try {
		List<Map<String, Object>> userList= this.getUserList();
		
		configuration.setDefaultEncoding("UTF-8");  
		configuration.setTemplateUpdateDelayMilliseconds(0);  
		configuration.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);  
		//获取模板
		Template template = configuration.getTemplate("userlist.ftl");
		Map<String,Object> root = new HashMap<String,Object>();
		root.put("userList", userList);
		root.put("userListSize", String.valueOf(userList.size()+2));
		
		SimpleDateFormat  sdf = new SimpleDateFormat("yyyyMMddHHmmss");
		Date today = new Date();
		String fileName = "人员列表" + sdf.format(today);//以今天的日期为文件名 
		response.setContentType("application/msexcel;charset=UTF-8");
		response.setHeader("Content-disposition","attachment;filename=\""+new String((fileName+".xls").getBytes("GBK"),"ISO8859-1")+"\"");
		//response字符流转换成字节流，template需要字节流作为输出
		OutputStream outputStream = response.getOutputStream();
		OutputStreamWriter outputWriter = new OutputStreamWriter(outputStream,"UTF-8");
		Writer writer = new BufferedWriter(outputWriter);
		template.process(root, writer);
		writer.flush();  
		writer.close();  
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```

在浏览器中输入请求地址就可以导出了.



#### 复杂Excel

生成Excel也有很多复杂的情况, 比如带图片, 跨行合并等.

这里例多行合并的, 例如:
![这里写图片描述](https://ww1.sinaimg.cn/large/007rAy9hly1g1n7f60at3j30b302ot8j.jpg)

生成的xml:

```xml
<Table ss:ExpandedColumnCount="5" ss:ExpandedRowCount="99999" x:FullColumns="1"
 x:FullRows="1" ss:DefaultColumnWidth="54" ss:DefaultRowHeight="13.5">
 <Row>
  <Cell ss:MergeAcross="3" ss:StyleID="s66"><Data ss:Type="String">人员列表</Data></Cell>
  <Cell ss:StyleID="s62"/>
 </Row>
 <Row>
  <Cell ss:StyleID="s67"/>
  <Cell ss:StyleID="s67"><Data ss:Type="String">姓名</Data></Cell>
  <Cell ss:StyleID="s67"><Data ss:Type="String">年龄</Data></Cell>
  <Cell ss:StyleID="s67"><Data ss:Type="String">孩子</Data></Cell>
  <Cell ss:StyleID="s69"><Data ss:Type="String">地址</Data></Cell>
 </Row>
 <Row>
  <Cell ss:MergeDown="1" ss:StyleID="m87387632"><Data ss:Type="Number">1</Data></Cell>
  <Cell ss:MergeDown="1" ss:StyleID="m87387612"><Data ss:Type="String">张三</Data></Cell>
  <Cell ss:MergeDown="1" ss:StyleID="m87387592"><Data ss:Type="Number">22</Data></Cell>
  <Cell ss:StyleID="s67"><Data ss:Type="String">小张</Data></Cell>
  <Cell ss:MergeDown="1" ss:StyleID="m87387572"><Data ss:Type="String">北京</Data></Cell>
 </Row>
 <Row>
  <Cell ss:Index="4" ss:StyleID="s67"><Data ss:Type="String">二张</Data></Cell>
 </Row>
</Table>
```

仔细观察这段代码, 跟上面的不带合并的xml进行对比:

- 每个带合并的Cell都带有一个属性MergeDown, 它的值为 (合并格数-1)
- 合并的格数是通过一个对象的循环来得到, 比如这里的”孩子”字段
- 扩展的Row会有一个Index属性, 它的值就是字段的列数, 这里是4


分析好之后开始加标签:

```xml
  <Table ss:ExpandedColumnCount="5" ss:ExpandedRowCount="4" x:FullColumns="1"
   x:FullRows="1" ss:DefaultColumnWidth="54" ss:DefaultRowHeight="13.5">
   <Row>
    <Cell ss:MergeAcross="3" ss:StyleID="s66"><Data ss:Type="String">人员列表</Data></Cell>
    <Cell ss:StyleID="s62"/>
   </Row>
   <Row>
    <Cell ss:StyleID="s67"/>
    <Cell ss:StyleID="s67"><Data ss:Type="String">姓名</Data></Cell>
    <Cell ss:StyleID="s67"><Data ss:Type="String">年龄</Data></Cell>
    <Cell ss:StyleID="s67"><Data ss:Type="String">孩子</Data></Cell>
    <Cell ss:StyleID="s69"><Data ss:Type="String">地址</Data></Cell>
   </Row>
   
<#assign index = 0 >
<#list userBoList as u>
<#assign index = index + 1 >
<#assign num = u.children?size-1>
<#if num lt 0>
<#assign num = 0>
</#if>
   <Row>
    <Cell ss:MergeDown="${num}" ss:StyleID="m87387632"><Data ss:Type="Number">${index}</Data></Cell>
    <Cell ss:MergeDown="${num}" ss:StyleID="m87387612"><Data ss:Type="String">${u.name}</Data></Cell>
    <Cell ss:MergeDown="${num}" ss:StyleID="m87387592"><Data ss:Type="Number">${u.age}</Data></Cell>
	<!-- 在children的List中取出第一个 -->
	<#list u.children as firstChildren>
	  <#if firstChildren_index == 0>
		<Cell ss:StyleID="s67"><Data ss:Type="String">${firstChildren.name}</Data></Cell>
	  </#if>
	</#list>
    <Cell ss:MergeDown="${num}" ss:StyleID="m87387572"><Data ss:Type="String">${u.address}</Data></Cell>
   </Row>
   
  <!-- 从第一个之后开始取 -->
  <#list u.children as c>
    <#if (u.children?size > 1 && c_index > 0 )>  
     <Row>
      <Cell ss:Index="4" ss:StyleID="s67"><Data ss:Type="String">${c.name}</Data></Cell>
     </Row>
    </#if>
  </#list>
  
</#list>

  </Table>
```

**注意:**
MergeDown这个属性中判断了是否小于0, 亲测在office中<0会导致Excel报错, WPS不会, 最好还是判断一下
ExpandedRowCount可以给个很大的值, 最好还是通过程序来传大小

```
在这个Excel中需要两个List:
User: List<UserBo>
Children: List<ChildrenBo>
User -> Children : 1->N
也可以是Map: List<Map<String, Object>>
```