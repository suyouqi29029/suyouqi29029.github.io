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

##### 4.python多线程

Python中使用线程有两种方式：函数或者用类来包装线程对象

```
函数式：调用 _thread 模块中的start_new_thread()函数来产生新线程。语法如下:

_thread.start_new_thread ( function, args[, kwargs] )

参数说明:

function - 线程函数。
args - 传递给线程函数的参数,他必须是个tuple[数组]类型。
kwargs - 可选参数。
实例：
#!/usr/bin/python3

import _thread
import time

# 为线程定义一个函数
def print_time( threadName, delay):
   count = 0
   while count < 5:
      time.sleep(delay)
      count += 1
      print ("%s: %s" % ( threadName, time.ctime(time.time()) ))

# 创建两个线程
try:
   _thread.start_new_thread( print_time, ("Thread-1", 2, ) )
   _thread.start_new_thread( print_time, ("Thread-2", 4, ) )
except:
   print ("Error: 无法启动线程")

while 1:
   pass
执行结果：
    执行以上程序输出结果如下：

Thread-1: Wed Apr  6 11:36:31 2016
Thread-1: Wed Apr  6 11:36:33 2016
Thread-2: Wed Apr  6 11:36:33 2016
Thread-1: Wed Apr  6 11:36:35 2016
Thread-1: Wed Apr  6 11:36:37 2016
Thread-2: Wed Apr  6 11:36:37 2016
Thread-1: Wed Apr  6 11:36:39 2016
Thread-2: Wed Apr  6 11:36:41 2016
Thread-2: Wed Apr  6 11:36:45 2016
Thread-2: Wed Apr  6 11:36:49 2016
```

线程模块 Python3 通过两个标准库 _thread 和 threading 提供对线程的支持。

_thread 提供了低级别的、原始的线程以及一个简单的锁，它相比于 threading 模块的功能还是比较有限的。

```
threading 模块除了包含 _thread 模块中的所有方法外，还提供的其他方法：

threading.currentThread(): 返回当前的线程变量。
threading.enumerate(): 返回一个包含正在运行的线程的list。正在运行指线程启动后、结束前，不包括启动前和终止后的线程。
threading.activeCount(): 返回正在运行的线程数量，与len(threading.enumerate())有相同的结果。
除了使用方法外，线程模块同样提供了Thread类来处理线程，Thread类提供了以下方法:

run(): 用以表示线程活动的方法。
start():启动线程活动。
join([time]): 等待至线程中止。这阻塞调用线程直至线程的join() 方法被调用中止-正常退出或者抛出未处理的异常-或者是可选的超时发生。
isAlive(): 返回线程是否活动的。
getName(): 返回线程名。
setName(): 设置线程名。
```

###### 使用 threading 模块创建线程

我们可以通过直接从 threading.Thread 继承创建一个新的子类，并实例化后调用 start() 方法启动新线程，即它调用了线程的 run() 方法：

```
# -*- icoding:UTF-8 -*-
# 使用threading模块中的thread类来创建线程
# 本文件不能以threading.py为名，与python中的模块名称重复了
import threading
import time

# 定义一个线程类
class myThread(threading.Thread):
    def __init__(self,threadId,name,delay):
        threading.Thread.__init__(self)
        self.threadId = threadId
        self.name = name
        self.delay = delay
    def run(self):
        print("开始线程%s"%(self.name))
        work(self.name,self.delay)
        print("线程%s结束"%(self.name))
        
def work(name,delay):
    count = 0;
    while count < 5:
        time.sleep(delay)
        print("线程%s正在工作:%s"%(name,time.ctime()))
        count += 1
    
#使用threading模块中的Thread类创建线程
thread1 = myThread(1,"工作1",2)
thread2 = myThread(2,"工作2",2)

if __name__ == '__main__':
    thread1.start()
    print("当前线程开始阻塞，为已启动线程让位")
    thread1.join()
    thread2.start()
    print("已启动线程已运行完毕，启动第二线程，并且当前线程开始开始阻塞，以等待调用join方法的线程先运行完毕")
    thread2.join()
    print("主线程结束")
    
运行结果：
开始线程工作1
当前线程开始阻塞，为已启动线程让位
线程工作1正在工作:Fri Jul  3 10:21:56 2020
线程工作1正在工作:Fri Jul  3 10:21:58 2020
线程工作1正在工作:Fri Jul  3 10:22:00 2020
线程工作1正在工作:Fri Jul  3 10:22:02 2020
线程工作1正在工作:Fri Jul  3 10:22:04 2020
线程工作1结束
开始线程工作2
已启动线程已运行完毕，启动第二线程，并且当前线程开始开始阻塞，以等待调用join方法的线程先运行完毕
线程工作2正在工作:Fri Jul  3 10:22:06 2020
线程工作2正在工作:Fri Jul  3 10:22:08 2020
线程工作2正在工作:Fri Jul  3 10:22:10 2020
线程工作2正在工作:Fri Jul  3 10:22:12 2020
线程工作2正在工作:Fri Jul  3 10:22:14 2020
线程工作2结束
主线程结束
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