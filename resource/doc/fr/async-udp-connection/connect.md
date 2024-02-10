# Méthode connect
```php
void AsyncUdpConnection::connect()
```
Effectue une opération de connexion asynchrone. Cette méthode retourne immédiatement.

### Paramètres
Aucun paramètre

### Valeur de retour
Aucune valeur de retour

### Exemple
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Démarrer un client UDP une seconde plus tard, connectez-vous au port 1234 et envoyez la chaîne "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Recevoir les données renvoyées par le serveur "hello"
            echo "recv $data\r\n";
            // Fermer la connexion
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Recevoir les données envoyées par le client AsyncUdpConnection, renvoyer la chaîne "hello"
    $connection->send("hello");
};
Worker::runAll();  
```

Après l'exécution, l'impression ressemblera à ceci :
```
recv hello
```
