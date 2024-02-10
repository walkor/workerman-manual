Windowsオペレーティング・システムでは、1つのPHPファイルで複数のWorkerを初期化することはできません。

たとえば、以下のtest.phpがあります。
```php
<?php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
...
```
Windowsで起動すると、エラーが発生します。
```
multi workers init in one php file are not support
```
## 解決方法
解決方法は、複数の起動スクリプトを使用することです。それぞれのスクリプトで1つのWorkerをインスタンス化するか、ポートごとに1つの起動ファイルを使用します。

例えば、

ファイルstart\_socket\_server.php

``` 
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

ファイルstart\_websocket\_server.php

``` 
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
...
```

ファイルstart\_webserver.php

``` 
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
...
```

次のように直接3つのスクリプトを起動します（[Windowsコマンドプロンプト](https://baike.baidu.com/view/756438.htm)で実行）。

``` 
php start_socket_server.php start_websocket_server.php start_webserver.php
```
