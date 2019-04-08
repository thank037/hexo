---
title: 总结Eclipse设置
date: 2015-06-15 12:39:04
categories: 工具
tags:
  - 工具
  - IDE
---

总结一些Eclipse常用的设置, 不设置不好用

<!-- more -->

##### **去掉红色波浪线的拼写检查:**

> Windows->Preferences->General->Editors->Text Editors-> Spelling
> 将Enable spell checking取消即可.

##### **去掉语法提示:**

> General-Editors-Text Editors-Annotations-Errors将Show in的三个选项都去掉.

##### **调整字体:**

> Windows -> Preferences -> Color and Font -> Basic -> TextFont

##### **调整背景色:**

> Windows -> Editors -> Text Editors -> 下面BackGround选择:色调85，饱和度90，亮度205.(豆沙绿)

##### **去掉自动添括号:**

> Windows -> Java -> Editor -> Content Assist -> 把Insert single proposals automatically前面的勾去掉

##### **去掉自动添括号:**

> Server Locations:
> 第二个: User Tomcat installation(takes control of Tomcat installation)
> 部署deploy可改为原来的: webapps, 不改的话另外创建一个wtpwebapps

##### **修改Timeouts**

> 默认的太小了, 如果你的项目在这个时间内还跑不起来, 就会报错, 所以seconds改大点.

##### 后续补充

> TODO