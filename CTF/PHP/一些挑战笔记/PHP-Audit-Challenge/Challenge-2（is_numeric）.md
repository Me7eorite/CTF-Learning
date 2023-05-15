### 考点
- is_numeric() 的利用
### 源码
```php
<?php
show_source(__FILE__);
$flag = "xxxx";
if(isset($_GET['time'])){ 
        if(!is_numeric($_GET['time'])){ 
                echo 'The time must be number.'; 
        }else if($_GET['time'] < 60 * 60 * 24 * 30 * 2){ 
                        echo 'This time is too short.'; 
        }else if($_GET['time'] > 60 * 60 * 24 * 30 * 3){ 
                        echo 'This time is too long.'; 
        }else{ 
                sleep((int)$_GET['time']); 
                echo $flag; 
        } 
                echo '<hr>'; 
}
```
### Solution
```python
>>> 60 * 60 * 24 * 30 * 2      
5184000                        
>>> 60 * 60 * 24 * 30 * 3      
7776000                        
>>> hex(5184000)               
'0x4f1a00'                     
>>> hex(7776000)               
'0x76a700'     
```
我们通过GET或者POST传入的参数，是作为字符串保存的。`is_numeric()`支持**普通数字型字符串**、**科学记数法型字符串**、**部分支持十六进制0x型字符串**。而强制类型转换int，**不能正确转换的类型有十六进制型字符串、科学计数法型字符串（部分）**。
这里给一个测试代码，自行理解
```php
<?php
show_source(__FILE__);
$temp = $_GET['temp'];
echo (int)$temp;
?> 
```