---
layout: post 
title: "Python爬虫学习与实战(cnvd网信息爬取)"
date: 2020-07-02
author: su29029
tags: Web
---

## Python爬虫学习与实战(cnvd网信息爬取)

#### 0x00 爬虫基础

##### 1.python requests库

基础使用：

```python
import requests
try:
	r = requests.get("https://www.baidu.com/")
	if r.status_code == 200:
		r.encoding = "utf-8"
        print(r.text)
except:
	print("error")
```

几个主要使用方法：

```python
request.get(url, params=payload)
request.post(url, data=payload)
request.request(HTTP_METHODS, url, **kwargs)
'''
HTTP_METHODS:http请求方法
url:请求url
kwargs:控制参数，主要使用的有：
1.params字典序列(主要用于get)
2.data字典，字节序列或文件对象(主要用于post)
3.cookies字典或者CookieJar，Request中的cookie
4.timeout设定超时时间，秒为单位
'''
```

##### 2.正则表达式基础

| 字符        | 意义                                                         |
| ----------- | ------------------------------------------------------------ |
| \           | 将下一个字符标记为一个特殊字符、或一个原义字符、或一个向后引用、或一个八进制转义符。 |
| ^           | 字符串开始                                                   |
| $           | 字符串结束                                                   |
| *           | 匹配前面的子表达式零次或多次，例如，zo*能匹配“`z`”以及“`zoo`“ |
| +           | 匹配前面的子表达式一次或多次。例如，“`zo+`”能匹配“`zo`”以及“`zoo`”，但不能匹配“`z`” |
| ?           | 匹配前面的子表达式零次或一次。例如，“`do(es)?`”可以匹配“`does`”或“`does`”中的“`do`” |
| {n}         | 匹配n次，例如，“`o{2}`”不能匹配“`Bob`”中的“`o`”，但是能匹配“`food`”中的两个o。 |
| {n,}        | 至少匹配n次                                                  |
| {n,m}       | 至少匹配n次，最多m次                                         |
| .           | 匹配除“`\`*`n`*”之外的任何单个字符。                         |
| (pattern)   | 匹配pattern并获取这一匹配。                                  |
| (?:pattern) | 匹配pattern但不获取匹配结果                                  |
| x\|y        | 匹配x或y                                                     |
| [xyz]       | 匹配字符集合，例如，“`[abc]`”可以匹配“`plain`”中的“`a`”      |
| [a-z]       | 匹配字符范围，例如，“`[a-z]`”可以匹配“`a`”到“`z`”范围内的任意小写字母字符。 |
| [^a-z]      | 负值字符范围。匹配任何不在指定范围内的任意字符。例如，“`[^a-z]`”可以匹配任何不在“`a`”到“`z`”范围内的任意字符。 |
| \d          | 匹配一个数字                                                 |
| \D          | 匹配一个非数字                                               |
| \n          | 匹配换行                                                     |
| \r          | 匹配回车                                                     |
| \s          | 匹配任何空白字符，包括空格、制表符、换页符等等。             |
| \S          | 匹配任何非空白字符                                           |
| \t          | 匹配制表符                                                   |
| \v          | 匹配垂直制表符                                               |
| \w          | 匹配包括下划线的任何单词字符。等价于“`[A-Za-z0-9_]`”。       |

##### 3.python re库

基础使用：

```python
import re
strings = ['I love python', 'I love China', 'I like running']
pattern = r'^(I)(\s)(love)(\s)(\w+)$'
for i in range (0, 3):
    res = re.findall(pattern, strings[i])
    if len(res) > 0:
        print(strings[i])
        
'''
output:
I love python
I love China
'''
```

几个主要使用方法：

```
re.search(pattern, string, flag=0) # 在一个字符串中搜索匹配的第一个位置，返回match对象
re.match(pattern, string, flag=0) # 在一个字符串的开始位置开始匹配，返回match对象
re.findall(pattern,string) # 搜索字符串，以列表类型返回全部匹配字串
re.split(pattern, string, maxsplit=0, flag=0) # 将一个字符串按照匹配结果进行分割，返回列表类型
re.sub(pattern, replace, string, count=0, flag=0) # 在字符串中替换所有匹配的字串，返回替换后的结果
```

#### 0x01 实战(爬取cnvd网的所有漏洞信息)

代码：

```python
import requests, threading, re
def run(n1, n2):
    for i in range (n1, n2):
        try:
            r = requests.get("https://www.cnvd.org.cn/webinfo/show/{}".format(i))
            if (r.status_code == 200):
                isExist = re.findall(r'(<title>)([\u0000-\uFFFF]+)(</title>)',r.text)[0]
                if (isExist[1] != "出错了...."):
                    title = re.findall(r'(blkContainerSblk")([\u0000-\uFFFF]+)(<h1>)([\u0000-\uFFFF]+)(</h1>)',r.text)[0]
                    content = re.findall(r'(blkContainerSblkCon clearfix)([\u0000-\uFFFF]+)(<p>)([\u0000-\uFFFF]+)(</p>)(<p>\u53c2\u8003\u94fe\u63a5)',r.text)[0]
                    print(i, title[3], content[3])
                    fp = open("res/{}.html".format(i), 'w')
                    fp.write(title[3])
                    fp.write("\n")
                    fp.write(content[3])
                    fp.close()

            else:
                print(r.status_code)
                print("No")
        except Exception as e:
            print(i, e)
th = []
for i in range (1,100):
    th.append(threading.Thread(target=run, args=((i-1)*100, i*100)))

for t in th:
    t.start()
```

注释：

1.首先对网页进行源代码审计，寻找需要的目标信息的位置和特征

2.编写正则表达式，进行目标信息匹配

3.先进行requests.get()直接访问网页，注意网页的反爬措施

4.随后根据目标网页集合的特征，进行循环访问

5.整理代码

6.提高爬取速度，采用多线程进行爬取

7.汇总结果

最终结果将汇总在res/i.html中。
