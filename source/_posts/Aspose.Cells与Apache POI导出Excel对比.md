---
title: Aspose.Cells与Apache POI导出Excel对比
date: 2019-5-20 15:39:21
categories: 工作
tags:
  - freemarker
---


[TOC]

### 1. 文档说明

使用freemarker导出Excel, 无法直接操作Excel

需要先将Excel模版转换为xml可读格式作为freemarker的模板文件(`.ftl`), 再进行模版元素和表达式替换

<!-- more -->

所以在打开文件时会有如下提示
```
文件格式和扩展名不匹配, 文件可能已经损坏或不安全
```
需点击确认后才能打开, 影响用户体验

#### 目的
对比以下两款产品

- Apache POI - Java Excel APIs
- Aspose - Aspose.Cells for java

对比在操作 Excel 的区别, 解决上述问题, 并考虑两种产品功能性, 开发上的区别

---



### 2. Apache POI

> Apache POI - the Java API for Microsoft Documents

[Apache POI](http://poi.apache.org/index.html) 是 Apache 提供操作 Microsoft Office 的开源库, 支持多种平台

#### 环境

JDK环境: 1.5 或以上

引入maven依赖, 目前最新的Release版本为`4.1.0`, 测试使用POI版本为 `3.11`
```xml
<dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi-ooxml</artifactId>
	<version>3.11</version>
	<type>jar</type>
	<scope>compile</scope>
</dependency>

<dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi-scratchpad</artifactId>
	<version>3.11</version>
	<type>jar</type>
	<scope>compile</scope>
</dependency>
```

#### 代码示例

<span id="code1">代码示例</span>

```java
public static void main(String[] args) throws Exception {

	HSSFWorkbook workbook = new HSSFWorkbook();
	HSSFSheet sheet = workbook.createSheet();

	HSSFCell cell = sheet.createRow(0).createCell(0);
	cell.setCellValue("---用户信息---");
	HSSFCellStyle cellStyle = cell.getCellStyle();

	// 标题行合并单元格
	sheet.addMergedRegion(new CellRangeAddress(0, 0, 0, 3));

	// 边框, 单元格样式等设置略...
	List<Map<String, Object>> userList = getUserList();
	for (int i = 0; i < userList.size(); i++) {
		Map<String, Object> user = userList.get(i);
		HSSFRow row = sheet.createRow(i + 1);
		HSSFCell cell1 = row.createCell(0);
		HSSFCell cell2 = row.createCell(1);
		HSSFCell cell3 = row.createCell(2);
		cell1.setCellValue(i + 1);
		cell2.setCellValue(String.valueOf(user.get("name")));
		cell3.setCellValue(String.valueOf(user.get("address")));

		cell1.setCellStyle(cellStyle);
		cell2.setCellStyle(cellStyle);
		cell3.setCellStyle(cellStyle);
	}

	File outFile = new File("G:\\poi_output.xls");
	FileOutputStream outputStream = new FileOutputStream(outFile);
	workbook.write(outputStream);
}
```

![poi导出Excel-用户信息](https://i.loli.net/2019/05/20/5ce23ec10d27851109.png)

---

### 3. Aspose Cells

**Aspose for Java**

>  [Category Archive: Aspose.Total Product Family](https://blog.aspose.com/category/total/)

- Aspose为开发人员提供了广泛的Java API，用于创建和管理处理文件和不同格式的各种类型的应用程序
- 包含丰富组件: 
   - Aspose.Cells处理与Microsoft Excel ® 和OpenOffice电子表格。
   - Aspose.Words for Microsoft Word和OpenOffice文档。
   - Aspose.Slides for Microsoft PowerPoint和OpenOffice演示文稿文件。
   - Aspose.Pdf用于创建和操作PDF文档。
   - Aspose.BarCode用于生成和识别条形码。
   - etc...

**Aspose.Cells for Java**

>  [Aspose.Cells](https://products.aspose.com/cells)

Aspose.Cells for Java是一个Excel®电子表格报告组件，它使Java应用程序无需使用MicrosoftExcel®即可创建和管理Excel®电子表格。

Aspose.Cells for Java是一个功能非常丰富的组件，它提供的不仅仅是基本的数据导出功能。



#### 环境

JDK环境: JDK 1.6 或以上

依赖: 在maven构建的项目中, 可直接引入Aspose.Cells相关依赖, 有两种方式

**方式一**

- 添加Repository

  ```xml
  <repository>
  	<id>AsposeJavaAPI</id>
  	<name>Aspose Java API</name>
  	<url>https://repository.aspose.com/repo/</url>
  </repository>
  ```

  > Aspose.Cells for Java非开源
  >
  > 需要单独引入aspose的远程仓库地址, 才能引入aspose相关jar包, 未提供source包

- 引入依赖(目前最新版本为 `19.4)`

  ```xml
  <dependency>
  	<groupId>com.aspose</groupId>
  	<artifactId>aspose-cells</artifactId>
  	<version>19.4</version>
  </dependency>
  <dependency>
  	<groupId>com.aspose</groupId>
  	<artifactId>aspose-cells</artifactId>
  	<version>19.4</version>
  	<classifier>javadoc</classifier>
  </dependency>
  <dependency>
  	<groupId>org.bouncycastle</groupId>
  	<artifactId>bcprov-jdk16</artifactId>
  	<version>1.46</version>
  </dependency>
  ```

- 修改maven settings

  如果maven配置文件 `settings.xml` 中的 `mirrorOf` 配置为 `*`, 需要修改该配置项, 使其能够从pom中添加的repository下载依赖, 否则不生效, 例如: 

  ```xml
  <mirror>
  	...
  	<mirrorOf>*,!AsposeJavaAPI</mirrorOf>
  	...
  </mirror>
  ```

  > `mirrorOf` 中指定的是aspose仓库的id



**方式二**

从Aspose官网直接下载 Aspose.Cells for Java 对应版本的jar, 添加至项目lib中: Add as Library

适用与非maven构建项目

>  下载地址: https://downloads.aspose.com/cells/java



#### 代码示例 

<span id="code2">代码示例</span>

```java
public static void main(String[] args) throws Exception {
	String title = "巡检记录";
	List<String> columns = Lists.newArrayList(
			"序号", "巡检编号", "开始时间", "结束时间", "巡线人", "巡线总时长", "巡线里程(公里)", "事件上报");
	outFileToStream(title, columns, fetchData());
}

public static void exportExcel(String title, List<String> columns, List<Map<String, String>> dataList) throws Exception {

	Workbook workbook = new Workbook();
	WorksheetCollection worksheets = workbook.getWorksheets();
	Worksheet worksheet = worksheets.get(0);
	Cells cells = worksheet.getCells();

	// 设置标题和列名样式: 文字加粗, 字体, 大小, 颜色, 居中, 单元格内容自动换行
	Style titleStyle = workbook.createStyle();
	titleStyle.getFont().setBold(true);
	titleStyle.setName("宋体");
	titleStyle.getFont().setSize(15);
	titleStyle.setHorizontalAlignment(TextAlignmentType.CENTER);
	titleStyle.setTextWrapped(true);
	wrapperStyle(titleStyle);

	//设置内容样式: 居中, 文字字体, 文字大小, 单元格内容自动换行
	Style contentStyle = workbook.createStyle();
	contentStyle.setHorizontalAlignment(TextAlignmentType.CENTER);

	// 设置边框
	wrapperStyle(contentStyle);

	Cell titleCell = cells.get(0, 0);
	titleCell.setValue(title);
	titleCell.setStyle(titleStyle);
	cells.setRowHeight(0, 20);
	cells.setStandardWidth(25);
	cells.merge(0, 0, 1, 8);

	// 列名
	for (int i = 0; i < columns.size(); i++) {
		Cell cell = cells.get(1, i);
		cell.setValue(columns.get(i));
		cell.setStyle(titleStyle);
	}

	for (int i = 0; i < dataList.size(); i++) {
		Map<String, String> data = dataList.get(i);
		cells.get(i + 2, 0).setValue(i + 1);
		cells.get(i + 2, 0).setStyle(contentStyle);
		cells.get(i + 2, 1).setValue("code" + i);
		cells.get(i + 2, 1).setStyle(contentStyle);
		cells.get(i + 2, 2).setValue(data.get("time"));
		cells.get(i + 2, 2).setStyle(contentStyle);
		cells.get(i + 2, 3).setValue(data.get("time"));
		cells.get(i + 2, 3).setStyle(contentStyle);
		cells.get(i + 2, 4).setValue(data.get("inspector"));
		cells.get(i + 2, 4).setStyle(contentStyle);
		cells.get(i + 2, 5).setValue(data.get("wholeTime"));
		cells.get(i + 2, 5).setStyle(contentStyle);
		cells.get(i + 2, 6).setValue(data.get("mileage"));
		cells.get(i + 2, 6).setStyle(contentStyle);
		cells.get(i + 2, 7).setValue(data.get("eventCount"));
		cells.get(i + 2, 7).setStyle(contentStyle);
	}

	workbook.save("G:\\巡检记录_export_by_aspose.xls");
}
```



效果如下

![aspose.cell导出Excel-巡检记录](https://i.loli.net/2019/05/20/5ce23f2197fb156460.png)



#### Excel转换PDF

Aspose.Cells for Java 从 2.3.0 版本开始引入了Microsoft Excel到PDF的转换, 无需单独引入Aspose.PDF, 也无须安装 Microsoft Office环境

以Excel转PDF为例

**代码示例**
```java
public static void main(String[] args) throws Exception {
	String PATH_LINUX = "/root/office/";
	Workbook wb = new Workbook(PATH_LINUX + "巡检监控数据库表.xlsx");
	File pdfFile = new File(PATH_LINUX + "test.pdf");
	FileOutputStream fileOS = new FileOutputStream(pdfFile);
	wb.save(fileOS, SaveFormat.PDF);
}
```

**说明**

Aspose.Cells 中 Excel到PDF的转换有一些限制。电子表格中指定的某些格式设置可能会丢失，并且不支持所有图形对象

官网列出了一些限制: [Conversion Attributes](https://docs.aspose.com/display/cellsjava/Converting+Workbook+to+Different+Formats#ConvertingWorkbooktoDifferentFormats-ConvertingExceltoPDFFiles)

> 实际测试中, 转换PDF会出现错行



### 4. 其他

#### 以Excel模版方式导出

除了以上示例的通过API方式操作 Excel 元素样式并导出, Apache POI 和 Aspose.Cells for java 都支持以Excel模版方式导出

> 适用于样式复杂的Excel导出场景

**代码示例**
```java
public static void main(String[] args) throws Exception {

	File file = ResourceUtils.getFile("classpath:export\\userinfo.xls");
	FileInputStream inputStream = new FileInputStream(file);
	
	HSSFWorkbook workbook = new HSSFWorkbook(inputStream);
	HSSFSheet sheet = workbook.getSheetAt(0);
	
	HSSFCell cell = sheet.getRow(0).getCell(0);
	cell.setCellValue("---用户信息---");
	HSSFCellStyle cellStyle = cell.getCellStyle();
	
	List<Map<String, Object>> userList = getUserList();
	for (int i = 0; i < userList.size(); i++) {
		Map<String, Object> user = userList.get(i);
		HSSFRow row = sheet.createRow(i + 2);
		HSSFCell cell1 = row.createCell(0);
		HSSFCell cell2 = row.createCell(1);
		HSSFCell cell3 = row.createCell(2);
		cell1.setCellValue(i + 1);
		cell2.setCellValue(String.valueOf(user.get("name")));
		cell3.setCellValue(String.valueOf(user.get("address")));
	}
	
	File outFile = new File("G:\\poi_output.xls");
	FileOutputStream outputStream = new FileOutputStream(outFile);
	workbook.write(outputStream);
}
```



#### Aspose评估版本&授权&破解

上面使用的Apose Maven repos中引入的jar是Aspose.Cells for Java 的评估版。

评估版提供与产品的许可版本完全相同的功能, 但是有两个限制

- 第一个限制：已打开文件的数量
  运行程序时，只能打开100个Excel文件。如果您的应用程序超过此数量，将引发异常。

- 第二个限制：会自动生成并聚焦到一个有警告信息的sheet页

  ![licensing_evalwarningsheet1.png](https://i.loli.net/2019/05/20/5ce24a1e78df157921.png)

> 生成预览文件(如PDF)也会自动添加水印

**授权方式** 

- 在[Aspose网站](https://purchase.aspose.com/)上购买许可证

- 添加许可文件到项目中

- 在程序调用Aspose.Cells for Java API之前设置Licene授权, 例如: 

  ```java
  License license = new License();
  license.setLicense("license.xml");
  // 操作Excel API 相关代码...
  ```

- 验证License

  ```java
  Boolean License.isLicenseSet() // 查看是否授权
  Date License.getSubscriptionExpireDate() // 查看过期时间
  ```



**破解版说明**

Aspose.Cells for Java 网上的破解方案一般有两种

- 方式一: 原始jar + license许可文件

  较老版本中可以找到类似的license许可文件, 例如 `license.xml`

  ```xml
  <License>
    <Data>
      <Products>
        <Product>Aspose.Total for Java</Product>
        <Product>Aspose.Words for Java</Product>
      </Products>
      <EditionType>Enterprise</EditionType>
      <SubscriptionExpiry>20991231</SubscriptionExpiry>
      <LicenseExpiry>20991231</LicenseExpiry>
      <SerialNumber>8bfe198c-7f0c-4ef8-8ff0-acc3237bf0d7</SerialNumber>
    </Data>
    <Signature>sNLLKGMUdF0r8O1kKilWAGdgfs2BvJb/2Xp8p5iuDVfZXmhppo+d0Ran1P9TKdjV4ABwAgKXxJ3jcQTqE/2IRfqwnPf8itN8aFZlV3TJPYeD3yWE7IT55Gz6EijUpC7aKeoohTb4w2fpox58wWoF3SNp6sK6jDfiAUGEHYJ9pjU=</Signature>
  </License>
  ```

- 方式二: 修改jar + license许可文件

  通过修改并替换原始jar中的字节码文件, 改变license验证相关逻辑, 达到破解的目的



### 5. 总结

Apache POI Java Excel APIs 与 Aspose.Cells for Java 都能欧解决 打开 freemarker 导出的 Excel 文件时出现的警告提示问题

#### 对比

**1. 常用导出场景支持**

都支持两种常用的导出场景: 
- 使用API操作元素导出: 适用于Excel样式不复杂
- 基于Excel模版导出: 适用于导出样式复杂的报表场景, 减少编码量

**2. 文档转换(Excel转PDF)**

Apache POI 无法直接调用API转换, 需要使用例如jacob调用Microsoft Office或WPS才能实现转换

Aspose.Cells for Java 从2.3.0版本后提供了PDF转换功能, 无需引入其它依赖, 也无需安装MS环境

但不适用于转换效果要求高, 实际测试中转换后的PDF会出现错行

**3. 开发便捷**

两者都提供了maven依赖, 可以在maven构建的项目中快速集成, 在开发上, 两者直接操作Excel的API类似, 并无太大区别, 详细功能代码差异参考

- [代码示例1](#code1), [代码示例2](#code2)

- [Code Comparison for Common Features in Aspose.Cells and Apache POI](https://docs.aspose.com/display/cellsjava/Code+Comparison+for+Common+Features+in+Aspose.Cells+and+Apache+POI+-+HSSF+and+XSSF)

**4. 功能**

除了基本的导入, 导出场景, 列举一些在Aspose.Cell中提供, 而Apache POI中没有的功能

数据处理: 
- 搜索查找数据: 根据值查找单元格
- 计算值: 通过API使用编程的方式, 计算单元格中的值
- 解析数组/集合中的元素导入工作表 & 从工作表中导出数据
- Aspose.Cells提供了大量内置函数和公式, 类似Excel中的计算工时
- 对数据进行排序

工作表: 
- 给Excel添加水印
- 转换为HTML, PDF
- 显示和隐藏工作簿的滚动条, 选项卡, 更改选项卡颜色
- 加密Excel
- 使用多种编码读取CSV文件
- 创建数据透视表

<details>
<summary>展开查看</summary>
<pre><code>
  public class void main () {
   sss
  }
</code></pre>
</details>

|                  | Aspose.Cells | Apache POI |
| ---------------- | ------------ | ---------- |
| MS Office环境    | 不需要       | 不需要     |
| 支持基于模版导出 | 支持         | 支持       |
| 开源             | 否, 收费     | 是, 免费   |
| 使用难易         | 易           | 易         |
| 功能             | 丰富         | 基本       |





