# SSE 
**Cette fonctionnalité nécessite workerman>=4.0.0**

SSE, ou Server-sent Events, est une technique de push côté serveur. Son essence est que, après qu'un client envoie une demande http avec l'en-tête `Accept: text/event-stream`, la connexion reste ouverte et le serveur peut continuellement pousser des données vers le client sur cette connexion.

La différence avec le websocket est la suivante :
* SSE ne peut que pousser du serveur vers le client ; WebSocket permet une communication bidirectionnelle.
* SSE prend en charge par défaut la reconnexion en cas de déconnexion ; WebSocket nécessite une implémentation personnalisée.
* SSE ne peut transmettre que du texte en UTF-8, les données binaires doivent être encodées en UTF-8 avant d'être transmises ; WebSocket prend en charge par défaut l'envoi de données en UTF-8 et en binaire.
* SSE possède des types de messages intégrés ; WebSocket nécessite une implémentation personnalisée.

### Exemple
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\ServerSentEvents;
use Workerman\Protocols\Http\Response;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Si l'en-tête Accept est text/event-stream, alors il s'agit d'une demande SSE
    if ($request->header('accept') === 'text/event-stream') {
        // Envoyer tout d'abord une réponse avec l'en-tête Content-Type: text/event-stream
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // Pousser des données au client de manière régulière
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // Supprimer le timer lorsque la connexion est fermée, pour éviter une accumulation continue de timers et des fuites de mémoire
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // Envoyer un événement "message", avec les données "hello" et un identifiant de message facultatif
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// Démarrer le worker
Worker::runAll();
```

Code JavaScript côté client 
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // Output: hello
}, false);
```
