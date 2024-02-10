# How to initialize multiple Workers in Windows operating system

It is not possible to initialize multiple Workers in a single PHP file on the Windows operating system.

For example, in the following test.php:
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
Starting this on Windows will result in an error:
```
multi workers init in one php file are not support
```

## Solution
The solution is to use multiple startup scripts, with each script instantiating a Worker, or in other words, one startup file for each port.

For example, in order to initialize two Worker instances (tcp and websocket) and one WebServer instance, three startup files need to be created: start\_socket\_server.php, start\_websocket\_server.php, and start\_webserver.php.

For example:

File start\_socket\_server.php
```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

File start\_websocket\_server.php
```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

File start\_webserver.php
```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

To start, simply run three scripts in the Windows CMD command line like this:
```php
   php start_socket_server.php start_websocket_server.php start_webserver.php
```
