# Как инициализировать несколько Worker на операционной системе Windows

На операционной системе Windows невозможно инициализировать несколько Worker в одном файле PHP.

Например, в файле test.php:

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

При запуске в Windows появится ошибка:

```
multi workers init in one php file are not support
```

## Решение

Решением является использование нескольких запускающих скриптов, где каждый скрипт запускает свой экземпляр Worker, или же по одному запускающему файлу на каждый порт.

Предположим, что инициализируются два экземпляра Worker (tcp и websocket) и один экземпляр WebServer. В этом случае необходимо создать три запускающих файла: start\_socket\_server.php, start\_websocket\_server.php и start\_webserver.php.

Например:

Файл start\_socket\_server.php:

```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

Файл start\_websocket\_server.php:

```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

Файл start\_webserver.php:

```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

При запуске можно запустить эти три скрипта следующим образом (в командной строке [Windows cmd](https://baike.baidu.com/view/756438.htm)):

```shell
php start_socket_server.php start_websocket_server.php start_webserver.php
```
