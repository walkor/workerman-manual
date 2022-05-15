# windows操作系统下如何初始化多个Worker

windows操作系统下无法在一个php文件里初始化多个Worker，


例如下面test.php
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
在windows下启动会报错
```
multi workers init in one php file are not support
```
## 解决方法
解决方法是使用多个启动脚本，每个脚本启动实例化一个Worker，或者说每个端口一个启动文件。

假设初始化两个Worker实例（tcp和websocket）和一个WebServer实例，则需要创建三个启动文件 start\_socket\_server.php 和 start\_websocket\_server.php start\_webserver.php

例如：

文件start\_socket\_server.php

```
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

文件start\_websocket\_server.php

```
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

文件start\_webserver.php

```
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

启动时就可以像这样，直接启动三个脚本，（在[windows cmd命令行](https://baike.baidu.com/view/756438.htm)中运行）

```
   php start_socket_server.php start_websocket_server.php start_webserver.php
```
