# Le rôle des protocoles de communication
Comme le TCP est basé sur un flux, les données de requête envoyées par le client arrivent au serveur comme un cours d'eau. Une fois les données détectées par le serveur, il doit vérifier si ces dernières sont complètes, car il est possible qu'une partie seulement de la requête soit arrivée, voire que plusieurs requêtes soient arrivées en même temps. Afin de déterminer si la requête est entièrement arrivée ou de séparer les requêtes provenant de connexions multiples, il est nécessaire d'établir un ensemble de protocoles de communication.

## Pourquoi établir des protocoles dans WorkerMan ?

Traditionnellement, le développement PHP est basé sur le Web et utilise principalement le protocole HTTP. Le traitement et l'analyse du protocole HTTP sont entièrement pris en charge par le serveur Web, de sorte que les développeurs n'ont pas à se soucier des aspects protocolaires. Cependant, lorsque nous devons développer en dehors du protocole HTTP, les développeurs doivent commencer à se préoccuper des protocoles.

## Protocoles actuellement pris en charge par WorkerMan
Actuellement, WorkerMan prend en charge les protocoles suivants : HTTP, websocket, protocole text (voir l'annexe pour plus de détails), protocole frame (voir l'annexe pour plus de détails), ws (voir l'annexe pour plus de détails). Lorsque vous avez besoin de communiquer en utilisant l'un de ces protocoles, vous pouvez les utiliser directement. La méthode consiste à spécifier le protocole lors de l'initialisation du Worker, par exemple :

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket://0.0.0.0:2345 indique l'écoute du port 2345 en utilisant le protocole websocket
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// Protocole text
$text_worker = new Worker('text://0.0.0.0:2346');

// Protocole frame
$frame_worker = new Worker('frame://0.0.0.0:2347');

// Worker TCP, transmission directe utilisant le socket sans aucun protocole applicatif
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// Worker UDP, aucun protocole applicatif utilisé
$udp_worker = new Worker('udp://0.0.0.0:2349');

// Worker de domaine Unix, aucun protocole applicatif utilisé
$unix_worker = new Worker('unix:///tmp/wm.sock');
```

## Utilisation d'un protocole personnalisé
Lorsque les protocoles de communication intégrés à WorkerMan ne répondent pas aux besoins de développement, les développeurs peuvent créer leur propre protocole de communication. La méthode de personnalisation est expliquée dans la section suivante.

**Conseil :**

Workerman intègre un protocole text avec un format de texte suivi d'un caractère de retour à la ligne. Le protocole text est très simple à utiliser pour le développement et le débogage, et peut être utilisé dans la plupart des scénarios de protocoles personnalisés. Il prend également en charge le débogage telnet. Si les développeurs doivent développer leur propre protocole applicatif, ils peuvent directement utiliser le protocole text et éviter de devoir le développer eux-mêmes.

Veuillez consulter l'annexe pour plus de détails sur le protocole text.
