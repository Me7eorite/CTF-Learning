# PHP底层调试

## 0x01 下载编译并代码

```php
git clone https://github.com/php/php-src
cd php-src/
git checkout PHP-7.3.6
```

如果需要更换其他的版本，只需要通过checkout进行修改。

在Mac中需要安装一下最新版的bison,按顺序执行一下两句命令即可替换成功。

```php
brew install bison

export PATH=/opt/homebrew/Cellar/bison/3.8.2/bin:$PATH  //注意这句话是一次性的，关闭终端后效果就没了。
```

然后接下来就是编译PHP源码

```php
 ./buildconf
 ./configure --disable-all --enable-debug --prefix=/path/to/target  //如果bison版本太低会报错
 make
 make install
```

编译的时候遇到报错第一个是readdir_r的问题，修改reentrancy.c文件的一个函数调用`readdir_r(dirp, entry);`修改方式如下

```
readdir_r(dirp, entry); --> readdir_r(dirp, entry, &entry);
```

第二个错误是提示 zend_language_parser.y:1317:5: error: **implicit** declaration of function 'yystpcpy' **is** invalid...修改Zend下的zend_language_parser.c文件。

```c
#define yyparse         zendparse
#define yylex           zendlex
#define yyerror         zenderror
#define yydebug         zenddebug
#define yynerrs         zendnerrs
#define yystpcpy        stpcpy
#define yystrlen        strlen
```

## 0x02 启动debug

在debug位置

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        
        {
            "name": "debuug php source",
            "type": "cppdbg",
            "request": "launch",
            "program": "/Users/me7eorite/Documents/CTF/PHP/php7.3.6/bin/php",
            "args": ["-f","//Users/me7eorite/Documents/CTF/PHP/1.php"],
            "stopAtEntry": true,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "lldb"
        }
        

    ]
}
```

- program 可执行的PHP文件的路径(编译生成的php文件)
- args 传给php的参数列表，像我上面所填写的执行的就是`php -f /1.php`，也可以使用-r来指定需要的执行的代码。
- cwd 当前目录，如果调试web应用，可以改成web根目录的路径
- stopAtEntry 是否在main函数的时候断下