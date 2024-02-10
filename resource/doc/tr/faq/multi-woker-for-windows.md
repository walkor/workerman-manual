# Birden fazla Worker nasıl başlatılır – Windows işletim sistemi

Windows işletim sistemi altında bir PHP dosyasında birden fazla Worker başlatılamaz.

Örneğin, test.php aşağıdaki gibi olsun:
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
Windows'ta bu şekilde başlatmaya çalışıldığında hata alınır:
```
multi workers init in one php file are not support
```
## Çözüm Yolu
Çözüm, birden fazla başlatma betiği kullanmaktır; her betik bir Worker'ı başlatır veya her port için bir başlatma dosyasıdır.

Örneğin:

Dosya start\_socket\_server.php
```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

Dosya start\_websocket\_server.php
```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
...
```

Dosya start\_webserver.php
```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
...
```

Başlatılırken, direkt üç betiği başlatmak için (Windows cmd komut istemcisinde çalıştırıldığında) şu şekilde kullanılabilir:
```bash
php start_socket_server.php start_websocket_server.php start_webserver.php
```
