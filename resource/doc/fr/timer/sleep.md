```php
int \Workerman\Timer::sleep(float $delay)
```
La fonction `Timer::sleep()` de la classe est similaire à la fonction `sleep()` intégrée de PHP, à la différence que `Timer::sleep()` est non bloquante (elle ne bloque pas le processus actuel).

> **Remarque**
> Cette fonctionnalité nécessite workerman>=5.0
> Cette fonctionnalité nécessite l'installation de composer require revolt/event-loop ^1.0.0, ou l'utilisation de Swoole/Swow comme pilote d'événements


### Paramètres
``` delay ```

Le temps avant l'exécution, en secondes, avec prise en charge des décimales pouvant être précisées jusqu'à 0,001, c'est-à-dire au niveau de la milliseconde.

### Valeur retournée
Aucune valeur retournée

### Exemple

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // Envoyer les données avec un délai de 1,5 seconde
    Timer::sleep(1.5);
    // Envoyer les données
    $connection->send('hello workerman');
};

Worker::runAll();
```
