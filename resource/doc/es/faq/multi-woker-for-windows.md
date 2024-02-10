# Cómo inicializar múltiples Workers en el sistema operativo Windows

En el sistema operativo Windows, no es posible inicializar múltiples Workers en un solo archivo PHP.

Por ejemplo, en el archivo test.php a continuación:

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

Al intentar iniciar en Windows, se mostrará un error:

```
multi workers init in one php file are not support
```

## Solución
La solución es utilizar múltiples scripts de inicio, donde cada script inicia una instancia de Worker o, en otras palabras, un archivo de inicio por cada puerto.

Supongamos que se inicializan dos instancias de Worker (tcp y websocket) y una instancia de WebServer, entonces se necesitarían tres archivos de inicio: start_socket_server.php, start_websocket_server.php y start_webserver.php.

Por ejemplo:

Archivo start_socket_server.php

```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

Archivo start_websocket_server.php

```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

Archivo start_webserver.php

```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

Para iniciar, se pueden ejecutar directamente los tres scripts (en el [símbolo del sistema de Windows](https://baike.baidu.com/view/756438.htm)) de la siguiente manera:

```php start_socket_server.php start_websocket_server.php start_webserver.php```
