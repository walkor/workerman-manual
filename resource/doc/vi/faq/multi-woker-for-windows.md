# Cách khởi tạo nhiều Worker trên hệ điều hành Windows

Trên hệ điều hành Windows, không thể khởi tạo nhiều Worker trong một tệp PHP.

Ví dụ về tệp test.php dưới đây:
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
Khi chạy trên Windows sẽ báo lỗi:
```
multi workers init in one php file are not support
```

## Phương pháp giải quyết
Giải pháp là sử dụng nhiều tập lệnh khởi động, mỗi tập lệnh khởi động sẽ khởi tạo một Worker hoặc một tập lệnh khởi động cho mỗi cổng.

Giả sử khởi tạo hai trường hợp Worker (tcp và websocket) và một trường hợp WebServer, bạn sẽ cần tạo ba tập lệnh khởi chạy là start\_socket\_server.php, start\_websocket\_server.php và start\_webserver.php.

Ví dụ:

Tệp start\_socket\_server.php
```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

Tệp start\_websocket\_server.php
```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

Tệp start\_webserver.php
```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

Khi khởi động, bạn có thể khởi chạy ba tập lệnh như sau (trong cửa sổ dòng lệnh [cmd của Windows](https://baike.baidu.com/view/756438.htm)):
```php
   php start_socket_server.php start_websocket_server.php start_webserver.php
```
