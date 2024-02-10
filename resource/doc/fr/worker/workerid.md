# id
Requis (workerman >= 3.2.1)

## Description:
```php
int Worker::$id
```

Le numéro d'identification actuel du processus worker, compris entre ```0``` et ```$worker->count-1```.

Cette propriété est très utile pour différencier les processus worker. Par exemple, s'il y a plusieurs processus dans une instance worker et que le développeur souhaite définir une minuterie dans l'un de ces processus, il peut le faire en identifiant le numéro du processus, par exemple en ne définissant la minuterie que dans le processus numéro 0 de l'instance worker (voir l'exemple).

## Remarque:

Le numéro d'identification du processus reste inchangé après un redémarrage du processus.

L'attribution des numéros d'identification des processus est basée sur chaque instance worker. Chaque instance worker commence à attribuer des numéros d'identification à ses propres processus, donc les numéros d'identification des processus entre les instances worker peuvent être en double, mais les numéros d'identification des processus dans une seule instance worker ne se répéteront pas. Par exemple:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// L'instance worker 1 a 4 processus, les numéros d'identification des processus seront respectivement 0, 1, 2, 3
$worker1 = new Worker('tcp://0.0.0.0:8585');
// Définir le démarrage de 4 processus
$worker1->count = 4;
// Après le démarrage de chaque processus, afficher le numéro d'identification du processus actuel ($worker1->id)
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// L'instance worker 2 a 2 processus, les numéros d'identification des processus seront respectivement 0, 1
$worker2 = new Worker('tcp://0.0.0.0:8686');
// Définir le démarrage de 2 processus
$worker2->count = 2;
// Après le démarrage de chaque processus, afficher le numéro d'identification du processus actuel ($worker2->id)
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// Exécuter le worker
Worker::runAll();
```
L'affichage sera similaire à
```
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

Remarque : Comme le système Windows ne prend pas en charge le réglage du nombre de processus count, il n'y a qu'un seul processus avec le numéro 0. De plus, sous Windows, il n'est pas possible d'initialiser deux écouteurs Worker à partir du même fichier, donc cet exemple ne peut pas être exécuté sous Windows.

## Exemple
Une instance worker a 4 processus, et seule la minuterie est définie dans le processus avec le numéro d'identification 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // Définit la minuterie uniquement dans le processus avec le numéro d'identification 0, les autres processus numéro 1, 2 et 3 ne définissent pas de minuterie
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 processus worker, seule la minuterie est définie dans le processus numéro 0\n";
        });
    }
};
// Exécuter le worker
Worker::runAll();
```
