## Comment personnaliser un protocole

En fait, personnaliser votre propre protocole est assez simple. Un protocole simple comprend généralement deux parties :
 * Un identifiant pour distinguer les limites des données
 * La définition du format des données

## Un exemple

### Définition du protocole
Supposons que l'identifiant pour distinguer les limites des données soit le caractère de saut de ligne "\n" (notez que les données de la demande elles-mêmes ne peuvent pas contenir de caractère de saut de ligne), et que le format des données soit JSON. Voici un exemple de requête conforme à cette règle :

<pre>
{"type": "message", "content": "bonjour"}
</pre>

Notez que à la fin des données de la requête ci-dessus, il y a un caractère de saut de ligne (représenté par la chaîne de caractères **"\n"** entre guillemets doubles en PHP), ce qui indique la fin de la requête.

### Étapes de mise en œuvre
Pour implémenter le protocole ci-dessus dans WorkerMan, supposons que le nom du protocole soit JsonNL et qu'il soit situé dans le projet MyApp. Vous devrez suivre les étapes suivantes :

1. Placez le fichier de protocole dans le répertoire Protocols du projet, par exemple le fichier MyApp/Protocols/JsonNL.php.
2. Implémentez la classe JsonNL avec le namespace ```Protocols;```. Cette classe doit implémenter trois méthodes statiques : input, encode et decode.

Remarque: WorkerMan appellera automatiquement ces trois méthodes statiques pour réaliser la fragmentation, le décodage et l'encodage. Pour plus de détails, consultez les explications du flux d'exécution ci-dessous.

### Flux d'interaction entre WorkerMan et la classe de protocole
1. Supposons qu'un client envoie un paquet de données au serveur. Dès que le serveur reçoit les données (peut-être une partie des données), il appelle immédiatement la méthode ```input``` du protocole pour vérifier la longueur du paquet. La méthode ```input``` renvoie la valeur de longueur ```$length``` à WorkerMan.
2. Après avoir reçu la valeur ```$length```, WorkerMan vérifie si la longueur des données actuellement mise en mémoire tampon atteint ou dépasse cette longueur. Si ce n'est pas le cas, WorkerMan continue d'attendre des données jusqu'à ce que la longueur des données en mémoire tampon soit supérieure ou égale à ```$length```.
3. Lorsque la longueur des données en mémoire tampon est suffisante, WorkerMan extrait une portion de données de longueur ```$length``` de la mémoire tampon (c'est-à-dire **la fragmentation**) et appelle ensuite la méthode ```decode``` du protocole pour **décoder** ces données. Les données décodées sont stockées dans la variable ```$data```.
4. Une fois les données décodées, WorkerMan transmet le contenu de la variable ```$data```, en appelant le rappel ```onMessage($connection, $data)```, à la couche métier. La couche métier peut ainsi obtenir les données complètes et déjà décodées envoyées par le client à travers la variable ```$data```.
5. Lorsque la couche métier souhaite envoyer des données au client en appelant la méthode ```$connection->send($buffer)```, WorkerMan utilisera automatiquement la méthode ```encode``` du protocole pour **encoder** le ```$buffer```, puis enverra les données encodées au client.

### Implémentation concrète

**Implémentation de MyApp/Protocols/JsonNL.php** 

```php
namespace Protocols;
class JsonNL
{
    /**
     * Vérifie l'intégrité du paquet
     * Si la longueur du paquet peut être obtenue, retourne la longueur du paquet dans le tampon, sinon retourne 0 pour attendre plus de données
     * En cas de problème de protocole, retourne false, et la connexion actuelle sera interrompue
     * @param string $buffer
     * @return int
     */
    public static function input($buffer)
    {
        // Obtenir la position du caractère de saut de ligne "\n"
        $pos = strpos($buffer, "\n");
        // S'il n'y a pas de saut de ligne, la longueur du paquet est inconnue, retourne 0 pour attendre plus de données
        if ($pos === false)
        {
            return 0;
        }
        // S'il y a un saut de ligne, retourne la longueur actuelle du paquet (y compris le saut de ligne)
        return $pos + 1;
    }

    /**
     * Encodage, appelé automatiquement lors de l'envoi de données au client
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // Sérialisation JSON, et ajout d'un saut de ligne comme marqueur de fin de requête
        return json_encode($buffer) . "\n";
    }

    /**
     * Décodage, appelé automatiquement lorsque le nombre d'octets de données reçues est égal à la valeur renvoyée par input (une valeur supérieure à 0)
     * et transmis au paramètre $data du rappel onMessage
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // Supprime le saut de ligne et reconvertit en tableau
        return json_decode(trim($buffer), true);
    }
}
```

À ce stade, le protocole JsonNL est mis en œuvre et peut être utilisé dans le projet MyApp. Voici un exemple d'utilisation :

Fichier : MyApp\start.php 
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data représente les données envoyées par le client, et ces données ont été traitées par JsonNL::decode
    echo $data;
    
    // Les données envoyées via $connection->send seront automatiquement traitées par JsonNL::encode, puis envoyées au client
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
...
```

> **Note**
> Workerman tentera de charger le protocole ```Protocols\JsonNL``` lorsqu'il rencontre l'instruction ```new Worker('JsonNL://0.0.0.0:1234')```. Si une erreur du type ```Class 'Protocols\JsonNL' not found``` se produit, veuillez consulter [l'auto-chargement](../faq/autoload.md) pour implémenter le chargement automatique.

### Description de l'interface du protocole
Les classes de protocole développées dans WorkerMan doivent implémenter trois méthodes statiques : input, encode, decode. La description de l'interface du protocole est disponible dans le fichier Workerman/Protocols/ProtocolInterface.php, définie comme suit :

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

/**
 * Interface du protocole
* @author walkor <walkor@workerman.net>
 */
interface ProtocolInterface
{
    /**
     * Utilisé pour la fragmentation dans $recv_buffer
     *
     * Si la longueur du paquet peut être obtenue dans $recv_buffer, retourne la longueur totale du paquet
     * Sinon, retourne 0, ce qui signifie qu'il faut plus de données pour obtenir la longueur actuelle du paquet
     * Si false ou un nombre négatif est retourné, une erreur de demande est renvoyée, et la connexion sera interrompue
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * Utilisé pour le décodage de la demande
     *
     * Si input retourne une valeur supérieure à 0, et que WorkerMan a reçu suffisamment de données, decode est automatiquement appelé
     * Ensuite, le rappel onMessage est déclenché, et les données décodées sont transmises au deuxième paramètre du rappel onMessage
     * Autrement dit, lorsqu'une demande client complète est reçue, le décodage est automatiquement activé, et il n'est pas nécessaire d'appeler manuellement le décodage dans le code métier
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     * Utilisé pour l'encodage de la demande
     *
     * Lorsque vous avez besoin d'envoyer des données au client en appelant $connection->send($data);
     * les données seront automatiquement encodées une fois avec encode, pour obtenir un format de données conforme au protocole, puis envoyées au client
     * Autrement dit, les données envoyées au client seront automatiquement enregistrées par encode, et il n'est pas nécessaire d'appeler manuellement l'encodage dans le code métier
     * @param ConnectionInterface $connection
     * @param mixed $data
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## Remarque :
Dans WorkerMan, il n'est pas strictement nécessaire que la classe de protocole soit implémentée à partir de ProtocolInterface. En réalité, tant que la classe de protocole contient les trois méthodes statiques input, encode et decode, elle est acceptable.
