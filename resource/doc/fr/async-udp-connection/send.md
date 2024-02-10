La méthode send
```php
void AsyncUdpConnection::send(string $data)
```
Effectue une opération de connexion asynchrone. Cette méthode retourne immédiatement.

### Paramètres
```$data```
Les données à envoyer au serveur. La taille des données ne peut pas dépasser 65507 octets (la taille maximale de transmission d'un paquet de données UDP est de 65507 octets), sinon l'envoi échouera.

### Valeur retournée
Aucune valeur retournée

### Exemple

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 seconde plus tard, démarrer un client UDP, se connecter au port 1234 et envoyer la chaîne de caractères "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Reçoit les données renvoyées par le serveur "hello"
            echo "recv $data\r\n";
            // Fermer la connexion
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Reçoit les données envoyées par le client AsyncUdpConnection, puis renvoie la chaîne de caractères "hello"
    $connection->send("hello");
};
Worker::runAll(); 
```

Lors de l'exécution, affiche quelque chose de similaire à:
```bash
recv hello
```
