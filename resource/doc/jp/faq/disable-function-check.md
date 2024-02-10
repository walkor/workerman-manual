# 禁用函数检查

このスクリプトを使用して、禁止された関数があるかどうかを確認します。以下のコマンドをコマンドラインで実行してください。```curl -Ss https://www.workerman.net/check | php```

もし```Function 関数名 may be disabled. Please check disable_functions in php.ini```というメッセージが表示されたら、Workermanの依存関数が無効になっている可能性があります。そのため、php.ini ファイルで無効化を解除する必要があります。

解除するためには、以下の2つの方法のいずれかを選んで実行してください。

## 方法一：スクリプトで解除

スクリプト ```curl -Ss https://www.workerman.net/fix | php``` を実行して、禁止を解除します。

## 方法二：手動で解除

**手順：**

1. `php --ini` を実行し、PHP CLI が使用する php.ini ファイルの場所を見つけます。
2. php.ini を開き、対応する関数の無効化を無効にします。

**依存関数**
Workermanを使用するためには、以下の関数の無効化を解除する必要があります。
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
