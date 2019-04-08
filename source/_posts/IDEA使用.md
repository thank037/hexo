---
title: IDEA使用
date: 2017-12-15 12:39:04
categories: 工具
thumbnail: /thumbnails/idea.png
tags:
  - 工具
  - IDE
---


## 前言

> 用了一段时间IDEA, 不想再用Eclipse了, 记录一些快捷键和好用插件
<!-- more -->


[TOC]

## 快捷键

### 切换功能

`Ctrl + PgUp/PgDn`: 切换文件
`Ctrl + Alt + [/]`: 切换项目
`Ctrl + Shift + 向上箭头/向下箭头`: 切换方法(上一个/下一个)
`Alt + 数字`: 切换窗口, IDE窗口每个都带有数字标号

### 查找功能

`Ctrl + Shift + A`: 查找选项
`Ctrl + Shift + R`: 查找文件 (如果要包括本项目外的 按2下R)
`Ctrl + Shift + T`: 查找class (如果要包括本项目外的 按2下T)
`Ctrl + H`: 全文查找指定字符串`:
`Ctrl + O `: 方法列表

### 其他
`Ctrl + Alt + S`: 设置
`Ctrl + Alt + Shilt + S`: 项目设置(包括project, module...)

`Shift + ESC`: 关闭IDEA中的当前窗口
`ESC`: 从其他窗口跳回代码编辑区

`Ctrl + Shift + U`: 大小写转换
`Alt + Enter 万能建议`:
`Alt + Insert`: 生成(例如setter/getter, 构造方法, Override方法等)

### 自定义的
`Ctrl + Q`: 跳转到上一个修改的地方
`Alt + Q`: 跳转到下一个修改的地方
`F10`: 添加到喜爱(方法和类都能添加)
`F11/F12`: 添加代码行到书签
`Alt + 2`: 调出Favourites窗口, 里面有书签, 喜爱, 断点
`Alt + J`: AceJump Word

## 好用插件

IDEA旗舰版本身就已经很多插件了, 不想安装太多, 但以下几个很喜欢

- **Alibaba Java Coding Guidelines**: 阿里巴巴代码检查插件
- **EJS**: 我的博客是node js, 需要打开`ejs`文件的插件
- **emacsIDEAs**: emacs神器, 不用多说, 脱离鼠标的利器
- **Grep Console**: 控制台日志根据级别变色输出, 很好看, 很实用
- **Lombok Plugin**: IDEA中要使用Lombok, 需要安装这个
- **Maven Helper**: 查看Maven依赖很方便
- **Rainbow Brackets**: 代码的选中高亮, 很好看很有趣
- **Translation**: 翻译插件, 很强大
