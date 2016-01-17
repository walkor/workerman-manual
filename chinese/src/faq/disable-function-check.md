# 禁用函数检查

使用这个脚本检查是否有禁用函数。命令行运行```curl -Ss http://www.workerman.net/check.php | php```

如果有提示```Function stream_socket_server may be disabled. Please check disable_functions in php.ini```说明workerman依赖的函数被禁用，需要在php.ini中解除禁用才能正常使用workerman。

步骤如下：

1、运行php --ini 找到php cli所使用的php.ini文件位置

2、打开php.ini，找到disable_functions一项解除stream_socket_server的禁用


