# Como inicializar vários Workers em um sistema operacional Windows

No sistema operacional Windows, não é possível inicializar vários Workers em um único arquivo PHP.

Por exemplo, no arquivo test.php a seguir:
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
Ao inicializar no Windows, um erro será gerado:
```
multi workers init in one php file are not support
```

## Solução
A solução é usar vários scripts de inicialização, onde cada script inicia uma instância de Worker, ou seja, um arquivo de inicialização para cada porta.

Supondo a inicialização de duas instâncias de Worker (tcp e websocket) e uma instância de WebServer, seria necessário criar três arquivos de inicialização: start_socket_server.php, start_websocket_server.php e start_webserver.php.

Por exemplo:

Arquivo start_socket_server.php
```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

Arquivo start_websocket_server.php
```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

Arquivo start_webserver.php
```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

Durante a inicialização, é possível iniciar os três scripts de forma direta, executando-os no prompt de comando do Windows, da seguinte maneira:
```sh
php start_socket_server.php
php start_websocket_server.php
php start_webserver.php
```
