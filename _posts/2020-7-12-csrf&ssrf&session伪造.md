---
layout: post 
title: "csrf&ssrf&session伪造问题"
date: 2020-07-12
author: su29029
tags: Web
---

## csrf&ssrf&session伪造问题

今天学习的内容是csrf，ssrf，session伪造这三个内容。复现平台dvwa&pikachu。

### 0x01 csrf

跨站请求伪造（Cross-site request forgery），也被称为one-click attack 或者session riding，通常缩写为CSRF 或者XSRF， 是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。

实施 CSRF 攻击，必须满足以下三个条件

1. **攻击者能执行相关的流程。**
2. **基于 Cookie 的会话机制.**
3. **没有不可预测的成分.**

一些常见的 CSRF 产生原因：

1. 仅从 HTTP 请求格式来选择性验证 Token. 比如对于 POST 型请求进行 Token 验证，但忽略 GET 型.
2. 仅对出现的 Token 进行验证，若请求中忽略了 Token，则不校验.
3. Token 没有和 Session 对应起来. 也就是其他用户的 Token 可以用于该用户建立的 Session 中，攻击者可以用自己的 Session 自己获取一个合法 Token，然后将其用于攻击.
4. CSRF 与非 Session 的 Cookie 关联，相当于也没有与 Session 挂钩.
5. CSRF 出现在 Cookie 中.
6. 仅仅依靠 `Referer` 来应对 CSRF，判断用户是不是从相同网站跳转来.

#### 靶场Pikachu

##### GET

登录后点击修改发现url上出现修改链接，那么只需要点击这个链接，内容就会被瞬间修改。

##### POST

登录后点击修改抓包发现跟GET无区别，那么只需要一个钓鱼网站，信息即可全部被修改。

##### Token

这次用了token，不能再直接操作了，需要靠xss等方式来获取token。

#### 靶场DVWA

##### Low

点击修改密码发现信息在url中出现，那么只需要点击这个url，密码直接被修改。

##### Medium

审计代码发现检查了HTTP_REFERER是否包含SERVER_NAME，Referer修改成服务器就绕过了。

##### High

这次有了user_token，token正确才可以修改密码，但是依然是没有验证用户是否直到原来的密码，如果我们通过xss等方式得到了token，那么验证也就失效了。(注意提交的时候依然使用的是GET请求，存在信息泄露隐患)。

##### Impossible

注意到这次表单多了一项当前密码，增大的攻击者的难度：哪怕得到一个可用的 Token，还需进一步拿到用户的密码才能修改。

#### CSRF 的防御

目前最好的方法就是使用 CSRF Token. 但是要求**安全**地生成 Token 并且**正确**地使用.

这意味着，产生 Token 时:

1. 随机数发生器必须达到足够的安全性，并且不会重复生成/使用相同的随机值，生成的随机数不能被预测. 推荐 `PRNG`.
2. 使用的 Hash 算法必须够强，不能是 `MD5` 等安全性较差的 Hash 算法.

传输和使用 Token 时需要注意的是： 

1. 尽量使用 POST 方法. 使用 GET 方法会导致: - Token 会被记录 - Token 可能传输到 HTTP Referer 的第三方服务器. - Token 可以在浏览器的地址栏中被显示. 
2. Token 不应通过 Cookie 传输.
3. Token 必须趁早放置在 HTML 中，并且设置为 `hidden`.

还可以通过验证码来强制交互。

### 0x02 SSRF

> 服务器端请求伪造 (Server-Side Request Forgery, SSRF)
>
> 其形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能,但又没有对目标地址做严格过滤与限制导致攻击者可以传入任意的地址来让后端服务器对其发起请求,并返回对该目标地址请求的数据.
>
> 比如可以突破防火墙，搞一些代理之类的；也可以向服务器发起请求，绕开 CDN 找到真实 IP.

PHP 中一些容易引发 SSRF 工具的函数:

1. `file_get_contents()`
2. `fsockopen()`
3. `curl_exec()`

#### curl_exec()

打开页面，点击链接发现url处有个url参数明显有问题，改个路径试试：

```
url=https://www.baidu.com
```

发现页面被正常渲染显示。随后尝试文件协议：

```
url=file:///etc/passwd
```

发现成功读取。。。

> curl支持的协议有：
>
> dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtmp rtsp smb smbs smtp smtps telnet tftp
>
> 其中最主要的几个：
>
> 1. `file://` 随便读取文件
> 2. `dict://` 扫描端口
> 3. `gopher://` 万金油协议，危害最大.
> 4. `telnet://` 连接主机

> 参考文献:
>
> ```
> # 利用file协议查看文件
> curl -v 'file:///etc/passwd'
> 
> # 利用dict探测端口
> curl -v 'dict://127.0.0.1:22'
> curl -v 'dict://127.0.0.1:6379/info'
> 
> # Gopher 协议可以做很多事情，特别是在 SSRF 中可以发挥很多重要的作用。利用此协议可以攻击内网的 FTP、Telnet、Redis、Memcache，也可以进行 GET,POST 请求。这极大拓宽了 SSRF 的攻击面。例如，利用gopher协议反弹shell。
> ```

#### file_get_contents()

跟上一个同理，url处可以随意构造。这里使用了file_get_contents()，故可以使用php伪协议，例如将/etc/passwd的内容输出：

```
http://localhost:8081/vul/ssrf/ssrf_fgc.php?file=php://filter/read=convert.base64-encode/resource=/etc/passwd
```

成功读取，base64解码即可。

#### fsockopen()

fsockopen的作用主要是与对端服务器建立socket连接。

```php
<?php
function GetFile($host,$port,$link) {
    $fp = fsockopen($host, intval($port), $errno, $errstr, 30);
    if (!$fp) {
        echo "$errstr (error number $errno) \n";
    } else {
        $out = "GET $link HTTP/1.1\r\n";
        $out .= "Host: $host\r\n";
        $out .= "Connection: Close\r\n\r\n";
        $out .= "\r\n";
        fwrite($fp, $out);
        $contents='';
        while (!feof($fp)) {
            $contents.= fgets($fp, 1024);
        }
        fclose($fp);
        return $contents;
    }
}
?>
```

> 这段代码使用 fsockopen 函数实现获取用户制定 URL 的数据（文件或者 HTML）。这个函数会使用 socket 跟服务器建立 TCP 连接，传输原始数据。

最简单的应用可以设置 DNS log 绕过 CDN 看真实 IP.

#### 防范 SSRF 的一些手段

1. 服务器开启 OpenSSL 无法进行交互利用
2. 服务端需要鉴权（Cookies & User: Pass）不能完美利用
3. 限制请求的端口为 http 常用的端口，比如，80, 443, 8080, 8090.
4. 禁用不需要的协议,仅仅允许 http 和 https 请求。可以防止类似于 `file://,gopher://,ftp://` 等引起的问题。
5. 统一错误信息，避免用户可以根据错误信息来判断远端服务器的端口状态.

### 0x03 session伪造

#### session简介和常见session的问题

Session 会话机制是一种服务器端机制，它使用类似于哈希表（可能就是哈希表）的结构来保存信息。当程序需要为客户端的请求创建会话时，服务器首先检查客户端的请求是否包含会话标识符（称为会话ID）。如果包含它，它先前已为此客户端创建了一个会话。服务器根据会话ID检索会话（无法检索，将创建新会话），如果客户端请求不包含会话ID，则为客户端创建会话并生成与会话关联的会话ID。 Seeion ID 应该是一个既不重复也不容易被复制的字符串。会话ID将返回给客户端以保存此响应。

session如果使用不当依然会造成严重的安全问题，主要有：

1. 会话预测: 即 Session ID 生成方式太简单，能够被预测. 攻击者可以预测有效的 Session ID 并获得对应用程序的访问权限.
2. 会话劫持: 通过利用各种手段获取用户 Session ID 后，使用该 Session ID 登录网站，获取目标用户的操作权限.
3. 会话重用: 服务器端Session未失效，攻击者可利用此Session向服务器继续发送服务请求. 测试方法：登录后将会话注销，再次重放登录时的数据包仍然可正常登录系统.
4. 会话失效时间过长: Session ID 的保存是比较麻烦，有的应用将 Session ID 的失效时间设置得过长，不仅增大服务器的负荷，还导致同一个 Session ID 可以被多次利用。 测试方法：系统登录后会话长时间不失效，使用系统功能，仍可正常使用。
5. 会话固定: 在用户进入登录页面，但还未登录时，就已经产生了一个 Session ID，用户输入信息，登录以后，Session ID 不会改变，也就是说没有建立新会话，原来的会话也没有被销毁. 测试方法：系统登录前和登录后，用户的 Session ID 保持不变。

#### DVWA 复现

##### Low

点Generate发现Http Header里的cookie值：

```
Cookie: dvwaSession=1; PHPSESSID=sml91451sl2vms4iftcson2fm2; security=low
```

再点一下：

```
Cookie: dvwaSession=2; PHPSESSID=sml91451sl2vms4iftcson2fm2; security=low
```

太明显了，dvwaSession每次加一。

##### Medium

点击后查看cookie：

```
Cookie: dvwaSession=1594535470; PHPSESSID=lmf4ka7t6ofsau9um2nl3jjsc7; security=mediu
```

还是太明显了...时间戳。

##### High

点击后查看cookie：

```
Cookie: dvwaSession=c4ca4238a0b923820dcc509a6f75849b; PHPSESSID=lmf4ka7t6ofsau9um2nl3jjsc7; security=high
```

这回不那么明显了，不过看着像md5，解密一下https://www.cmd5.com，发现就是数字从1递增，加了一层md5.....还是没有安全性可言。

##### Impossible

这回解密发现无法成功了。查看源码：

```php
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $cookie_value = sha1(mt_rand() . time() . "Impossible");
    setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], true, true);
}
?>
```

这回session的生成基本是无法破解了，将随机数，时间和 `Impossible` 凭借生成的字符串再加强 Hash. 而且采用比较安全的 POST 型提交, 减少 GET 型带来的潜在风险。这个方法是安全的。
