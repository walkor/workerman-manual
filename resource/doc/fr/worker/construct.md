# Constructeur __construct

## Description:
```php
Worker::__construct([string $listen , array $context])
```

Initialise une instance de conteneur Worker, permet de définir les propriétés du conteneur et les interfaces de rappel pour accomplir des fonctionnalités spécifiques.

## Paramètres
#### **``` $listen ```** (optionnel, si non spécifié, aucun port n'est écouté)

Si le paramètre ```$listen``` est défini, alors l'écoute du socket sera activée.

Le format de $listen est <protocole>://<adresse d'écoute>

**<protocole> peut être l'un des formats suivants :**

tcp : par exemple ```tcp://0.0.0.0:8686```

udp : par exemple ```udp://0.0.0.0:8686```

unix : par exemple ```unix:///tmp/my_file``` ```(nécessite Workerman>=3.2.7)```

http : par exemple ```http://0.0.0.0:80```

websocket : par exemple ```websocket://0.0.0.0:8686```

text : par exemple ```text://0.0.0.0:8686``` ```(text est un protocole texte intégré à Workerman, compatible avec telnet, voir la section Protocole Texte de l'annexe)```

ainsi que d'autres protocoles personnalisés, voir la section Personnalisation du protocole de ce manuel pour plus de détails.

**<adresse d'écoute> peut être l'un des formats suivants :**

Si c'est un socket Unix, l'adresse est un chemin local sur le disque.

Pour les autres types de socket, le format de l'adresse est <adresse IP locale>:<numéro de port>

<adresse IP locale> peut être ```0.0.0.0``` pour écouter sur toutes les cartes réseau de l'ordinateur, y compris les adresses IP locales, externes et la boucle locale 127.0.0.1

<adresse IP locale> en tant que ```127.0.0.1``` signifie qu'il n'écoute que sur la boucle locale, accessible uniquement à partir de l'ordinateur local, pas de connexion externe.

<adresse IP locale> en tant qu'adresse IP locale, similaire à ```192.168.xx.xx```, signifie qu'il n'écoute que sur une adresse IP locale, donc les utilisateurs externes ne peuvent pas y accéder.

Si la valeur de <adresse IP locale> n'est pas une adresse IP de l'ordinateur hôte, l'écoute ne peut pas être exécutée, et affiche une erreur ```Cannot assign requested address```

**Remarque :** <numéro de port> ne peut pas être supérieur à 65535, et s'il est inférieur à 1024, les privilèges root sont nécessaires pour écouter. Le port écouté doit être un port non utilisé sur l'ordinateur hôte, sinon l'écoute ne peut pas être activée et affiche une erreur ```Address already in use```

#### **``` $context ```**

Un tableau. Utilisé pour transmettre les options de contexte de socket, voir [Options de contexte de socket](https://php.net/manual/zh/context.socket.php)

## Exemples

Worker agissant en tant que conteneur http écoutant et traitant les requêtes http
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send("hello");
};

// Exécuter le conteneur
Worker::runAll();
```

Worker agissant en tant que conteneur websocket écoutant et traitant les requêtes websocket
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Exécuter le conteneur
Worker::runAll();
```

Worker agissant en tant que conteneur tcp écoutant et traitant les requêtes tcp
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Exécuter le conteneur
Worker::runAll();
```

Worker agissant en tant que conteneur udp écoutant et traitant les requêtes udp
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Exécuter le conteneur
Worker::runAll();
```

Conteneur Worker écoutant le socket Unix domain ```(nécessite Workerman version>=3.2.7)```
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Exécuter le conteneur
Worker::runAll();
```

Conteneur Worker sans aucune écoute, utilisé pour traiter des tâches planifiées
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // Exécuter toutes les 2.5 secondes
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "tâche en cours d'exécution\n";
    });
};

// Exécuter le conteneur
Worker::runAll();
```

**Conteneur Worker écoutant sur un port avec un protocole personnalisé**

Structure de répertoire finale
```
├── Protocols              // Création du répertoire Protocols
│   └── MyTextProtocol.php // Fichier de protocole personnalisé à créer
├── test.php  // Script à créer
└── Workerman // Répertoire source de Workerman, ne pas modifier le code à l'intérieur
```

1. Créer le répertoire Protocols et un fichier de protocole
Protocols/MyTextProtocol.php (conforme à la structure de répertoire précédente)

```php
// L'espace de noms des protocoles personnalisés est uniformément défini en tant que Protocols
namespace Protocols;
// Protocole texte simple, format de protocole : texte + saut de ligne
class MyTextProtocol
{
    // Fonction de paquetage, retourne la longueur du paquet actuel
    public static function input($recv_buffer)
    {
        // Recherche du saut de ligne
        $pos = strpos($recv_buffer, "\n");
        // Si aucun saut de ligne n'est trouvé, cela signifie qu'il ne s'agit pas d'un paquet complet, retourne 0 et attend la réception de données supplémentaires
        if($pos === false)
        {
            return 0;
        }
        // Si le saut de ligne est trouvé, retourne la longueur du paquet actuel, y compris le saut de ligne
        return $pos+1;
    }

    // Après réception d'un paquet complet, le décodage automatique s'effectue via la fonction decode, qui dans ce cas, se contente de supprimer le saut de ligne
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // Avant d'envoyer des données au client, le processus d'encodage s'effectue automatiquement via la fonction encode, qui ajoute ici un saut de ligne
    public static function encode($data)
    {
        return $data."\n";
    }
}
```

2. Utiliser le protocole MyTextProtocol pour écouter et traiter les requêtes

Créez le fichier test.php conformément à la structure de répertoire finale ci-dessus

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### Conteneur MyTextProtocol worker ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * Après réception de données complètes (terminées par un saut de ligne), l'exécution automatique de MyTextProtocol::decode('les données reçues')
 * Les résultats sont transmis à la fonction onMessage via $data
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * Envoi de données au client, ce qui déclenche automatiquement MyTextProtocol::encode('hello world') pour le codage du protocole,
     * puis l'envoi au client
     */
    $connection->send("hello world");
};

// Exécuter tous les travailleurs
Worker::runAll();
```

3. Test

Ouvrez un terminal, accédez au répertoire où se trouve test.php, exécutez ```php test.php start```
```
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

Ouvrez un terminal, utilisez telnet pour tester (il est recommandé d'utiliser telnet sur un système Linux)

Supposons que le test soit effectué sur la machine locale,
exécutez telnet 127.0.0.1 5678 dans le terminal
puis entrez "hi" et appuyez sur Entrée
vous recevrez le message "hello world\n"
```
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
