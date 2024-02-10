# Processus de base
(Exemple d'un serveur de chat WebSocket simple)

#### 1. Créer un répertoire de projet n'importe où
Par exemple SimpleChat/
Allez dans le répertoire et exécutez `composer require workerman/workerman`

#### 2. Inclure `vendor/autoload.php` (généré après l'installation de composer)
Créez start.php, incluez `vendor/autoload.php`
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. Choisir un protocole
Ici, nous choisissons le protocole de texte (un protocole personnalisé dans Workerman, au format texte + saut de ligne)

(Actuellement, Workerman prend en charge les protocoles HTTP, WebSocket, et Text. Si vous avez besoin d'utiliser un autre protocole, veuillez vous référer au chapitre sur le protocole pour développer le vôtre)

#### 4. Écrire un script de démarrage en fonction des besoins
Par exemple, voici un fichier d'entrée simple pour une salle de chat.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// Lorsqu'un client se connecte, attribue un uid, enregistre la connexion, et notifie tous les clients
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // Attribuer un uid à cette connexion
    $connection->uid = ++$global_uid;
}

// Lorsqu'un client envoie un message, le transmet à tout le monde
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// Lorsqu'un client se déconnecte, diffuser à tous les clients
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}

// Créer un Worker avec un protocole texte écoutant sur le port 2347
$text_worker = new Worker("text://0.0.0.0:2347");

// Démarrer uniquement 1 processus, cela facilite la transmission des données entre les clients
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();
```

#### 5. Tester
Vous pouvez tester le protocole Text avec la commande telnet
```shell
telnet 127.0.0.1 2347
```
