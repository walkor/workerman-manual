# 禁用函數檢查

使用此腳本檢查是否有禁用函數。命令列運行```curl -Ss https://www.workerman.net/check | php```

如果提示```Function 函數名 may be disabled. Please check disable_functions in php.ini```，表示 workerman 依賴的函數已被禁用，需要在 php.ini 中解除禁用才能正常使用 workerman。要解除禁用，可以參考以下兩種方法之一。

## 方法一：腳本解除

執行腳本 ```curl -Ss https://www.workerman.net/fix | php``` 以解除禁用。

## 方法二：手動解除

**步驟如下：**

1、執行 `php --ini` 找到 PHP CLI 所使用的 php.ini 文件位置。

2、開啟 php.ini，找到 `disable_functions` 一項並解除對應函數的禁用。

**依賴的函數**
使用 workerman 需要解除以下函數的禁用
```
stream_socket_server
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
