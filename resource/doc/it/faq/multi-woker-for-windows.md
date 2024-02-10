# Come inizializzare più Worker su un sistema Windows

Su un sistema Windows non è possibile inizializzare più Worker in un singolo file PHP.

Ad esempio, nel file test.php seguente
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

Avviando su Windows si otterrà un errore
```
multi workers init in one php file are not support
```

## Soluzione
La soluzione è utilizzare più script di avvio, ognuno per istanziare un Worker, o uno script di avvio per ogni porta.

Considerando l'inizializzazione di due istanze Worker (tcp e websocket) e un'istanza WebServer, saranno necessari tre file di avvio: start_socket_server.php, start_websocket_server.php e start_webserver.php.

Ad esempio:

File start_socket_server.php
```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

File start_websocket_server.php
```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

File start_webserver.php
```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

All'avvio, è possibile avviare i tre script direttamente, eseguendo il comando seguente nel [prompt dei comandi di Windows](https://baike.baidu.com/view/756438.htm)
```bash
php start_socket_server.php start_websocket_server.php start_webserver.php
```
