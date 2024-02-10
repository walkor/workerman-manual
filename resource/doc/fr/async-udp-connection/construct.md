# Méthode __construct
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
Crée un objet de connexion UDP.

AsyncUdpConnection permet à Workerman d'agir en tant que client pour transférer des données UDP vers un serveur distant.

## Paramètres
Paramètre : ``` remote_address ```

Adresse de la connexion, par exemple
``` udp://192.168.1.1:1234 ```
``` frame://192.168.1.1:8080 ```
``` text://192.168.1.1:8080 ```

## Exemple

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 seconde plus tard, lance un client UDP, se connecte au port 1234 et envoie la chaîne "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // Reçoit des données "hello" renvoyées par le serveur
            echo "recv $data\r\n";
            // Ferme la connexion
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // Reçoit des données envoyées par le client AsyncUdpConnection, renvoie la chaîne "hello"
    $connection->send("hello");
};
Worker::runAll();                 
```
Après exécution, affiche quelque chose comme :
```
recv hello
```
