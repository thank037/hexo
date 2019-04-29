---
title: easyui下拉选择树(ComboTree)
date: 2016-09-04 12:39:04
categories: 工作
---

### 前言
> 很多时候都会用到下拉框,有时候为了显示更好的效果,会用到树型下拉框,实际就是下拉框中带了一个树型的控件,所以它自然继承自tree和combobox.<!-- more -->
>
> 后台没什么问题, 但是easyui我是真的不会阿



### 一: 前台

1 首先从前台来讲, 先把html元素写好

```html
<select id="cc" value="01" style="width:200px;" ></select>
<a href="#" id="btn" class="easyui-linkbutton" onclick="getValue()">getValue</a>
```

这里用js加载方式:

```js
$('#cc').combotree({
    url:'tree_data.json',
});
```

这里的url是跟html统一文件夹下的一个json格式文件.是一个模拟的json数据.大致结构如下:

```js
[{	
	id: 1,	
	text: 'Languages',	
	children: [{		
		id: 11,		
		text: 'Java'	
		},{		
		id: 12,		
		text: 'C++'	
	}]
}]
```

一个简单基础的下拉树就形成了:

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/easyui1.png)

在下拉树旁边写了一个按钮, 这个按钮有一个普通的onclick事件,就是获取下拉树的选择的值.代码如下:

```js
function getValue(){
	var value = $('#cc').combotree('getValue');
	alert(value);
}
```

还有下拉树实现多选: 实际就是加了个multiple属性,代码如下

```js
$('#ccm').combotree({
	url:'tree_data.json',
	multiple: true,
	//cascadeCheck: false
	//onlyLeafCheck: true
});
```

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/easyui2.png)

如果要获取它选择的值只需要把前面呢个getValue换成getValues

> var value = $(‘#ccm’).combotree(‘getValues’);
> //会返回id值的数组,逗号连接这种方式getValues默认是级联的,也就是选择子节点,他的父节点也可能get上,所以有一个属性cascadeCheck.

cascadeCheck是定义是否层叠选中状态,也就是前面的小方框会不会自动级联,如图变成false就是不级联,但是这样很怪.

onlyLeafCheck是定义是否只在末级节点之前显示复选框,也就是只有子节点才会有复选框,也就是只能选择叶子节点,没什么级联了.

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/easyui3.png)



### 二:数据

假如下拉树是要选择班级下的学生信息, 那么随便造个数据.

![img](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/easyui4.png)



### 三:后台

既然前台的url接收是一个json格式的东西.可以看到这里面有三个关键属性, id, text, children.三者缺一不可.

那数据中的两个表在程序中的persistence层对应着两个实体. Entity Class和Entity Student.

除此之外,还需要一个Model.

```java
public class ComboTreeModel {
	private String id;
	private String text;
	private List<ComboTreeModel> children;
	
	//setter and getter
}
```

而最后传给url的,应该是json格式的:

下面就是通过业务关联, 将两个实体组合,查出需要数据,放到Model模型.

```java
@RequestMapping(value="/getSelectData")
@ResponseBody
public List<ComboTreeModel> getTreeData(HttpServletRequest request, HttpServletResponse response,ComboTreeModel treeModel){
	List classList = this.studentService.queryClassList();
	//[{eventid:"c001",classname:'奥赛班',eventid:"c002",classname:'直播班',eventid:"c003",classname:'火箭班'}]
	List<ComboTreeModel> list = new ArrayList<ComboTreeModel>();
	for(int i=0; i<classList.size(); i++){
		ComboTreeModel model = new ComboTreeModel();
		Map classMap = (Map) classList.get(i);
		//{eventid:"c001",classname:'奥赛班'}
		String classId = (String) classMap.get("eventid");
		String className = (String)classMap.get("classname");
		model.setId(classId); 
		model.setText(className);
		List studentList = this.studentService.queryByClassId(classId);
		
		if(studentList.size() > 0){
			List<ComboTreeModel> childrenList = new ArrayList<ComboTreeModel>();
			for(int j = 0;j<studentList.size();j++){
				ComboTreeModel model2 = new ComboTreeModel();
				Map studentMap = (Map)studentList.get(j);
				String studentEventId = (String) studentMap.get("eventid");
				String studentName = (String) studentMap.get("studentname");
				model2.setId(studentEventId);
				model2.setText(studentName);
				childrenList.add(model2);
			}
			model.setChildren(childrenList);
		}
		list.add(model); 
	}
	return list;
}
```

ResponseBody 会自动返回JSON格式的数据.