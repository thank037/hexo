---
title: Python写网络爬虫(一)
date: 2016-03-31 12:39:04
categories: Python
tags:
  - Python
  - 爬虫
---



[TOC]

#### 关于Python

学过C. 学过C++. 最后还是学Java来吃饭. <!-- more -->

一直在Java的小世界里混迹.

有句话说: “Life is short, you need Python!” 翻译过来就是: 人生苦短, 我用Python



究竟它有多么强大, 多么简洁?

抱着好奇心, 趁在公司不忙的几天. 还是忍不住的小学了一下.(- - 其实学了还不到两天)

随便用一个”HelloWorld”的例子

```java
//Java
class Main{
	public static void main(String[] args){
		String str = "HelloWorld!";
		System.out.println(str);
	}
}
```

```python
# python
str = 'HelloWorld'
print str
```

乍一看Python确实很爽! 简明了断 节省时间

至于效率. 我相信不管是任何语言, 作为开发者重要是的还是思维的精简!

也有人说: “Python一时爽, 重构火葬场”

但是Python的应用场景那么多. 根据自己的需要来选择, 相信那么多没错.

Python的确是一门值得学习的课程.



#### 关于网络爬虫

这个东西我也也不懂… 随便抓个解释, 淡淡的理解下

网络爬虫（又被称为网页蜘蛛，网络机器人，在FOAF社区中间，更经常的称为网页追逐者），是一种按照一定的规则，自动地抓取万维网信息的程序或者脚本。

另外一些不常使用的名字还有蚂蚁、自动索引、模拟程序或者蠕虫。

真正的复杂而强大的爬虫有很多爬行算法和策略.

我写的这个例子简直是简之又简.

Python基础还没学完, 就迫不及待的想做个东西来看看

于是就想到了写个网络爬虫. 来小小的练习一下

我会经常看母校的学校新闻, 于是就试着爬学校新闻, 没事的时候拿出来看看

因为学的甚少…这个爬虫的功能还是非常简单. 就是把学校官网中最新的新闻下载下来, 保存成网页. 想看多少页都可以.

这里我就抓了最新的前4页新闻

第一次写爬虫. 一个很简单的功能, 我把它分了3步:

- 第一步: 先爬一条新闻, 把它下载保存!
- 第二步: 把这一页的所有新闻都爬下来, 下载保存!
- 第三步: 把x页的所有新闻爬下来, 下载保存!

网页爬虫很重要的一点就是分析网页元素.

#### 一: 爬一个url

先爬一条新闻, 把它下载保存!

```python
#crawler: 甘肃农业大学新闻网板块的->学校新闻.
#爬这个页面的第一篇新闻
#http://news.gsau.edu.cn/tzgg1/xxxw33.htm

#coding:utf-8
import urllib

str = '<a class="c43092" href="../info/1037/30577.htm" target="_blank" title="双联行动水泉乡克那村工作组赴联系村开展精准扶贫相关工作">双联行动水泉乡克那村工作组赴联系村开展精准扶贫相关工作</a>'

hrefBeg = str.find('href=')
hrefEnd = str.find('.htm', hrefBeg)
href = str[hrefBeg+6: hrefEnd+4]
print href
href = href[3:]
print href

titleBeg = str.find(r'title=')
titleEnd = str.find(r'>', titleBeg)
title = str[titleBeg+7: titleEnd-1]
print title

url = 'http://news.gsau.edu.cn/' + href 
print 'url: '+url

content = urllib.urlopen(url).read()
#print content

filename = title + '.html'
#将抓取的页面content写入filename保存本地目录中
open(filename, 'w').write(content)
```

这个里面不用分析太多网页元素. 就字符串拼阿截阿就好了.



#### 二: 爬取一个页面的新闻

爬取本页面的所有新闻, 每页23篇

这个时候就要小小的分析下, 这23个url, 每个url怎么找?

![python爬虫-爬取一个页面](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/python1.jpg)

这里可以先锁定一个元素, 进行查找. 再就是注意每次find时的规律, 其实就是查找的顺序起始

这里我把每个url都保存在一个数组中, 检索完成后, 对数组里的url进行下载.

```python
#crawler甘肃农业大学新闻网板块的->学校新闻.
#http://news.gsau.edu.cn/tzgg1/xxxw33.htm

#<a class="c43092" href="../info/1037/30567.htm" target="_blank" title="双联行动水泉乡克那村工作组赴联系村开展精准扶贫相关工作">双联行动水泉乡克那村工作组赴联系村开展精准扶贫相关工作</a>
#coding:utf-8
import urllib
import time
import stat, os

pageSize = 23
articleList = urllib.urlopen('http://news.gsau.edu.cn/tzgg1/xxxw33.htm').read()
urlList = [' ']*pageSize
#锁定class="c43092"
hrefClass = articleList.find('class="c43092"')
hrefBeg = articleList.find(r'href=', hrefClass)
hrefEnd = articleList.find(r'.htm', hrefBeg)
href=articleList[hrefBeg+6: hrefEnd+4][3:]
print href

#url = 'http://news.gsau.edu.cn/' + href 
#print 'url: '+url

i = 0
while href!=-1 and i<pageSize:
    urlList[i] = 'http://news.gsau.edu.cn/' + href
    
    hrefClass = articleList.find('class="c43092"', hrefEnd)
    hrefBeg = articleList.find(r'href=', hrefClass)
    hrefEnd = articleList.find(r'.htm', hrefBeg)
    href=articleList[hrefBeg+6: hrefEnd+4][3:]
    print urlList[i]
    i = i+1
else:
    print r'本页所有URL已爬完!!!'

#将本页每一篇新闻下载到本地(已新闻标题文件名存储)
#title: <HTML><HEAD><TITLE>酒泉市市长都伟来校对接校地合作事宜-新闻网</TITLE>
j = 0
while j<pageSize:
    content = urllib.urlopen(urlList[j]).read()
    titleBeg = content.find(r'<TITLE>')
    titleEnd = content.find(r'</TITLE>', titleBeg)
    title = content[titleBeg+7: titleEnd]
    print title
    print urlList[j]+r'正在下载...'
    time.sleep(1)
    open(r'GsauNews' + os.path.sep + title.decode('utf-8').encode('gbk')+'.html', 'w+').write(content)
    j = j + 1
else:
    print r'当页全部url下载完毕!'
```



#### 三: 爬取所有新闻

爬取N个页面的所有新闻

这里要爬取N个页面, 首先就要分析你要爬取的是最新的, 而不能是固定的某几页

所以要分析下分页的数据, 正好主页最下面也给出了分页数据, 直接用它!

看下最近几页的url:

```
http://news.gsau.edu.cn/tzgg1/xxxw33.htm 第一页
http://news.gsau.edu.cn/tzgg1/xxxw33/221.htm 第二页
http://news.gsau.edu.cn/tzgg1/xxxw33/220.htm 第三页
http://news.gsau.edu.cn/tzgg1/xxxw33/219.htm 第四页
```

对比分页数据, 很容易发现规律, 就是: `fenyeCount-pageNo+1`

这里很烦的一点是不知道为什么, 除了第一页以外的其他页, 都会有不是本页而是前一页的一部分网页数据会掺进来. 导致我找了半天

做了很多判断才检索出来

```python
#crawler甘肃农业大学新闻网板块的->学校新闻.

#coding:utf-8
import urllib
import time
import stat, os

pageCount = 4
pageSize = 23
pageNo = 1
urlList = [' ']*pageSize*pageCount

#分析分页的网页元素
#<td width="1%" align="left" id="fanye43092" nowrap="">共5084条  1/222 </td>
indexContent = urllib.urlopen('http://news.gsau.edu.cn/tzgg1/xxxw33.htm').read()
fenyeId = indexContent.find('id="fanye43092"') #这里锁定分页的id进行查找
fenyeBeg = indexContent.find('1/', fenyeId)
fenyeEnd = indexContent.find(' ', fenyeBeg)
fenyeCount = int(indexContent[fenyeBeg+2: fenyeEnd])


i = 0
while pageNo <= pageCount:
    if pageNo==1:
        articleUrl = 'http://news.gsau.edu.cn/tzgg1/xxxw33.htm'
    else:
        articleUrl = 'http://news.gsau.edu.cn/tzgg1/xxxw33/'+ str(fenyeCount-pageNo+1) + '.htm'


    print r'--------共爬取'+ str(pageCount) + '页  当前第' + str(pageNo) + '页  URL:' + articleUrl

    articleList = urllib.urlopen(articleUrl).read()
    
    while i<pageSize*pageNo:

        if pageNo == 1:
            #i = 0,23,46...时,从头找, 其余从上一个url结束位置开找
            if i == pageSize*(pageNo-1):
                hrefId = articleList.find('id="line43092_0"')
            else:
                hrefId = articleList.find('class="c43092"', hrefEnd)   
        else:
            if i == pageSize*(pageNo-1):
                hrefId = articleList.find('id="lineimg43092_16"')
            else:
                hrefId = articleList.find('class="c43092"', hrefEnd)
        

        
        hrefBeg = articleList.find(r'href=', hrefId)
        hrefEnd = articleList.find(r'.htm', hrefBeg)
        if pageNo == 1:
            href=articleList[hrefBeg+6: hrefEnd+4][3:]
        else:
            href=articleList[hrefBeg+6: hrefEnd+4][6:]
        urlList[i] = 'http://news.gsau.edu.cn/' + href
    
        print urlList[i]
        i = i+1
    else:
        print r'========第'+str(pageNo)+'页url提取完成!!!'
    
    pageNo = pageNo + 1
print r'============所有url提取完成!!!============'+'\n'*3


print r'==========开始下载到本地==========='
j = 0
while j < pageCount * pageSize:
    content = urllib.urlopen(urlList[j]).read()
    titleBeg = content.find(r'<TITLE>')
    titleEnd = content.find(r'</TITLE>', titleBeg)
    title = content[titleBeg+7: titleEnd]
    print title
    print urlList[j]+r'正在下载...'+'\n'
    time.sleep(1)
    open(r'GsauNews' + os.path.sep + title.decode('utf-8').encode('gbk')+'.html', 'w+').write(content)
    j = j + 1
else:
    print r'下载完成, 共下载'+str(pageCount)+'页, '+str(pageCount*pageSize)+'篇新闻'
```

这就爬完了…看下爬完的效果

```
==================== RESTART: D:\python\CSDNCrawler03.py ====================
——–共爬取4页 当前第1页 URL:http://news.gsau.edu.cn/tzgg1/xxxw33.htm
http://news.gsau.edu.cn/info/1037/30596.htm
http://news.gsau.edu.cn/info/1037/30595.htm
http://news.gsau.edu.cn/info/1037/30593.htm
http://news.gsau.edu.cn/info/1037/30591.htm
http://news.gsau.edu.cn/info/1037/30584.htm
http://news.gsau.edu.cn/info/1037/30583.htm
http://news.gsau.edu.cn/info/1037/30580.htm
http://news.gsau.edu.cn/info/1037/30577.htm
http://news.gsau.edu.cn/info/1037/30574.htm
http://news.gsau.edu.cn/info/1037/30573.htm
http://news.gsau.edu.cn/info/1037/30571.htm
http://news.gsau.edu.cn/info/1037/30569.htm
http://news.gsau.edu.cn/info/1037/30567.htm
http://news.gsau.edu.cn/info/1037/30566.htm
http://news.gsau.edu.cn/info/1037/30565.htm
http://news.gsau.edu.cn/info/1037/30559.htm
http://news.gsau.edu.cn/info/1037/30558.htm
http://news.gsau.edu.cn/info/1037/30557.htm
http://news.gsau.edu.cn/info/1037/30555.htm
http://news.gsau.edu.cn/info/1037/30554.htm
http://news.gsau.edu.cn/info/1037/30546.htm
http://news.gsau.edu.cn/info/1037/30542.htm
http://news.gsau.edu.cn/info/1037/30540.htm
========第1页url提取完成!!!
——–共爬取4页 当前第2页 URL:http://news.gsau.edu.cn/tzgg1/xxxw33/221.htm
http://news.gsau.edu.cn/info/1037/30536.htm
...

==========开始下载到本地===========
校长吴建民带队赴广河县开展精准扶贫和双联工作-新闻网
http://news.gsau.edu.cn/info/1037/30574.htm正在下载…

动物医学院赴园子村开展精准扶贫相关工作-新闻网
http://news.gsau.edu.cn/info/1037/30573.htm正在下载…

……….

```



![python爬虫-效果](https://blog-md-pic-1259135436.cos.ap-chengdu.myqcloud.com/%E5%85%B6%E5%AE%83/python1.jpg)

一分多钟爬了90个网页.

当然代码还可以优化好多.

例如用正则, 匹配自己想要的内容,而不是像这样泛泛全保存.

以后的学习中继续改进. 爬些更有意思的东西

做这个例子只是为了看看, 初学Python能为我做什么.