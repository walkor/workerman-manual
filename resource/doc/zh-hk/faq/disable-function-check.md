# 禁用函式檢查

使用此腳本檢查是否有禁用函式。命令行執行```curl -Ss https://www.workerman.net/check | php```

如果提示```Function 函数名 may be disabled. Please check disable_functions in php.ini```表示workerman依賴的函式已被禁用，需在php.ini中解除禁用才能正常使用workerman。解除禁用可參考以下兩種方法，選擇其中一種即可。

## 方法一：腳本解除

執行腳本 `curl -Ss https://www.workerman.net/fix | php` 以解除禁用

## 方法二：手動解除

**步驟如下：**

1、執行`php --ini`找到php cli所使用的php.ini檔案位置

2、打開php.ini，找到`disable_functions`一項解除對應函式的禁用

**依賴的函式**
使用workerman需要解除以下函式的禁用
```stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
