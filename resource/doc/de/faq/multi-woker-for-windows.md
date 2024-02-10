# Wie initialisiert man mehrere Worker unter Windows?

Unter Windows kann man keine mehreren Worker in einer PHP-Datei initialisieren.

Zum Beispiel in der Datei test.php:

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

Beim Starten unter Windows tritt folgender Fehler auf:

```multi workers init in one php file are not support```

## Lösung

Die Lösung besteht darin, mehrere Startskripte zu verwenden, wobei jedes Skript eine Instanz eines Workers initialisiert oder besser gesagt, ein Startskript pro Port.

Angenommen, es sollen zwei Worker-Instanzen (TCP und WebSocket) sowie eine WebServer-Instanz initialisiert werden, dann müssen drei Startdateien erstellt werden: start\_socket\_server.php, start\_websocket\_server.php und start\_webserver.php.

Zum Beispiel:

Datei start\_socket\_server.php

```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

Datei start\_websocket\_server.php

```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
...
```

Datei start\_webserver.php

```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
...
```

Beim Starten können dann einfach alle drei Skripte direkt gestartet werden (im [Windows CMD-Befehlszeilenfenster](https://baike.baidu.com/view/756438.htm)):

```php start_socket_server.php start_websocket_server.php start_webserver.php```
