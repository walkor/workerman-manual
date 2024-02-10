```php
void Connection::close(mixed $data = '')
```

Fermer la connexion en toute sécurité et déclencher le rappel ```onClose``` de la connexion.

Bien que l'udp soit sans connexion, l'objet AsyncUdpConnection correspondant est toujours conservé en mémoire. Il est nécessaire d'appeler la méthode close pour libérer l'objet de connexion udp correspondant, sinon cet objet de connexion udp restera en mémoire, entraînant des fuites de mémoire.

## Paramètres

 ``` $data ```

Paramètre facultatif, les données à envoyer (si un protocole spécifique est spécifié, la méthode encode du protocole sera automatiquement appelée pour empaqueter les données ```$data```), lorsque les données sont envoyées, la connexion est fermée, puis le rappel onClose est déclenché.

La taille des données ne peut pas dépasser 65507 octets, sinon l'envoi échouera.

### Exemple 

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 seconde plus tard, démarrer un client udp, se connecter au port 1234 et envoyer la chaîne de caractères hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Réception des données renvoyées par le serveur hello
            echo "recv $data\r\n";
            // Fermer la connexion
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Recevoir les données envoyées par le client AsyncUdpConnection, renvoyer la chaîne de caractères hello
    $connection->send("hello");
};
Worker::runAll();             
```

Après l'exécution, cela affichera quelque chose comme:
``` 
recv hello
```
