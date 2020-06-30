---
layout: post 
title: "python爬虫学习(BeautifulSoup)"
date: 2020-06-30
author: su29029
tags: Web
---

今天学习的内容是python爬虫，利用BeautifulSoup库。

### 0x00 基本使用

安装：

```
pip3 install beautifulsoup4
```

使用：

我们先定义一个简单的index.html文件：

```
<html>
<body>
	<h1>hello beautifulsoup</h1>
	<div>
		<a href="#">123</a>
		<p class="hello">hello</p>
	</div>
</body>
</html>
```

python引用BeautifulSoup：

```python
from bs4 import BeautifulSoup
demo = 
soup = BeautifulSoup(open("index.html"), "html.parser")
print(soup.prettify())
'''
输出：
<html>
 <body>
  <h1>
   hello beautifulsoup
  </h1>
  <div>
   <a href="#">
    123
   </a>
   <p class="hello">
    hello
   </p>
  </div>
 </body>
</html>
'''
```

基本操作：

```python
soup.prettify()   # 美化输出html/xml文档内容
soup.a # 获取html中的第一个a标签
soup.title # 获取html中的title标签
soup.a['href'] # 获取html的第一个a标签的href属性
soup.p['class'] # 获取html的第一个p标签的class属性
soup.name # u'[document]'
soup.a.name # 'a'
soup.a.contents # 将html中第一个a标签的所有子节点以列表的方式输出
soup.contents[0] # 获取html标签所有子节点列表中的第一个元素
soup.a.contents[0].contents # AttributeError,字符串没有.contents属性，因为字符串没有子节点
soup.a.parent # 通过.parent属性来获取某个元素的父节点
soup.title.string.parent # 'title' 文档title的字符串的父亲结点存在，为<title>标签
for parent in soup.a.parents:
	if parent is None:
		print(parent)
	else:
		print(parent.name)

# 通过元素的.parents属性递归得到元素的所有父亲节点
soup.a.next_sibling 
soup.a.previous_sibling  # 使用.next_sibling和previous_sibling查询兄弟节点
for sibling in soup.a.next_siblings:
    print(repr(sibling))

for sibling in soup.a.previous_siblings:
    print(repr(sibling))
    
# 通过.next_siblings和.previous_siblings属性对当前节点的兄弟节点迭代输出
```

### 0x01 进阶用法

#### 正则表达式

找出所有以b开头的标签(<body>,<b>)：

```python
import re
for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
```

#### find_all()函数

```python
函数原型：find_all( name, attrs, recursive, text, **kwargs)
常见示例：
soup.find_all('a') # 获取html中的所有a标签
soup.find_all('a','b') # 获取html中的所有a标签和b标签
soup.find_all(href="#") # 获取html中所有href属性为"#"的标签
soup.find_all(id=True) # 获取html中所有包含id属性的标签
soup.find_all('a', limit=2) # 获取html中所有的a标签，返回最多2个
soup.find_all('title', recursive=False) # 搜索title标签的直接子节点(recursive默认为True, 将搜索该标签的所有子孙节点)
```
