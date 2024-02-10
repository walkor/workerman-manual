# Comment initialiser plusieurs travailleurs sur Windows

Il n'est pas possible d'initialiser plusieurs travailleurs dans un seul fichier PHP sous le système d'exploitation Windows.

Par exemple, dans le test.php ci-dessous
```php
<?php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on.....
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on.....
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
...
```
Lancer cela sous Windows entraînera une erreur :
```multi workers init in one php file are not support```

## Solution
La solution consiste à utiliser plusieurs scripts de démarrage, chaque script instanciant un travailleur, ou plutôt un fichier de démarrage par port.

Supposons que nous initialisions deux instances de travailleurs (tcp et websocket) et une instance de WebServer, nous aurons besoin de créer trois fichiers de démarrage : start\_socket\_server.php, start\_websocket\_server.php et start\_webserver.php.

Par exemple :

Fichier start\_socket\_server.php

```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on.....
....
```

Fichier start\_websocket\_server.php

```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on.....
....
```

Fichier start\_webserver.php

```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
....
```

Pour démarrer, vous pouvez simplement exécuter ces trois scripts dans une fenêtre de commande Windows à l'aide de la commande suivante :

```php start_socket_server.php start_websocket_server.php start_webserver.php```
