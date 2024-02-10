# 在Windows作業系統中如何初始化多個 Worker

在Windows操作系统中，無法在一個php檔案中初始化多個Worker，

舉例如下面的test.php檔案
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
在Windows下啟動會報錯
```
multi workers init in one php file are not support
```
## 解決方法
解決方法是使用多個啟動腳本，每個腳本啟動實例化一個Worker，或者說每個端口一個啟動檔案。

假設初始化兩個Worker實例（tcp和websocket）和一個WebServer實例，則需要創建三個啟動檔案 start\_socket\_server.php 和 start\_websocket\_server.php start\_webserver.php

例如：

檔案start\_socket\_server.php

```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

檔案start\_websocket\_server.php

```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

檔案start\_webserver.php

```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

啟動時就可以像這樣，直接啟動三個腳本，（在 [Windows cmd命令行](https://baike.baidu.com/view/756438.htm) 中運行）

```shell
   php start_socket_server.php start_websocket_server.php start_webserver.php
```
