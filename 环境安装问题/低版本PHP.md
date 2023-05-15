## php@7.4安装失败

错误: php@7.4 has been disabled because it is a versioned formula!



解决:

brew install shivammathur/php/php@7.4(需要挂代理)

brew link --overwrite php@7.4(增加软连接)

brew-php-switcher 7.4(切换成功)

