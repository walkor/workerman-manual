# Exemple de développement simple

## Installation

**Installer Workerman**
Exécutez dans un répertoire vide
`composer require workerman/workerman`

## Exemple 1 : Fournir des services Web via le protocole HTTP
**Créez le fichier start.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Crée un worker écoutant sur le port 2345, utilisant le protocole http
$http_worker = new Worker("http://0.0.0.0:2345");

// Lance 4 processus pour fournir des services
$http_worker->count = 4;

// Lorsqu'il reçoit des données envoyées par le navigateur, il répond "hello world" au navigateur
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Envoie "hello world" au navigateur
    $connection->send('hello world');
};

// Lance le worker
Worker::runAll();
```

**Exécutez dans le terminal (pour les utilisateurs de Windows, utilisez [l'invite de commande cmd](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn))**
```shell
php start.php start
```

**Test**

Supposons que l'IP du serveur soit 127.0.0.1

Accédez à l'URL http://127.0.0.1:2345 dans un navigateur

 **Remarque :**

1. Si l'accès échoue, veuillez vous référer à la section [Raisons de l'échec de la connexion du client](../faq/client-connect-fail.md) pour déboguer.
2. Le serveur utilise le protocole http, et peut uniquement communiquer avec le protocole http. Il n'est pas possible de communiquer directement avec d'autres protocoles tels que websocket.


## Exemple 2 : Fournir des services via le protocole WebSocket
**Créez le fichier ws_test.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Remarque : ici, contrairement à l'exemple précédent, le protocole utilisé est websocket
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// Lance 4 processus pour fournir des services
$ws_worker->count = 4;

// Lorsqu'il reçoit des données de la part du client, il renvoie "hello $data" au client
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Envoie "hello $data" au client
    $connection->send('hello ' . $data);
};

// Lance le worker
Worker::runAll();
```

**Exécutez dans le terminal**
```shell
php ws_test.php start
```

**Test**

Ouvrez Google Chrome, appuyez sur F12 pour ouvrir la console de débogage, saisissez (ou insérez le code ci-dessous dans une page html et exécutez-le avec javascript)

```javascript
// Supposons que l'IP du serveur soit 127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("Connexion réussie");
    ws.send('tom');
    alert("Envoi d'une chaîne au serveur : tom");
};
ws.onmessage = function(e) {
    alert("Message reçu du serveur : " + e.data);
};
```

  **Remarque :**

1. Si l'accès échoue, veuillez vous référer à la section [Problèmes courants - Echec de la connexion](../faq/client-connect-fail.md) pour déboguer.
2. Le serveur utilise le protocole websocket, et peut uniquement communiquer avec le protocole websocket. Il n'est pas possible de communiquer directement avec d'autres protocoles tels que http.


## Exemple 3 : Utiliser directement la transmission de données TCP
**Créez le fichier tcp_test.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Crée un Worker écoutant sur le port 2347, sans utiliser de protocole de couche application
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// Lance 4 processus pour fournir des services
$tcp_worker->count = 4;

// Lorsqu'il reçoit des données du client
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Envoie "hello $data" au client
    $connection->send('hello ' . $data);
};

// Lance le worker
Worker::runAll();
```

**Exécutez dans le terminal**
```shell
php tcp_test.php start
```

**Test : exécutez dans le terminal**
(L'effet en ligne de commande sur Linux ci-dessous est légèrement différent de celui sous Windows)
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**Remarque :**

1. Si l'accès échoue, veuillez vous référer à la section [Problèmes courants - Echec de la connexion](../faq/client-connect-fail.md) pour déboguer.
2. Le serveur utilise le protocole TCP brut, et ne peut communiquer directement avec d'autres protocoles tels que websocket, http, etc.
