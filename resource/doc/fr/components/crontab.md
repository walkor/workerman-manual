# workerman/crontab

# Description
`workerman/crontab` est un programme de tâches planifiées basé sur workerman, similaire à crontab sous Linux. `workerman/crontab` prend en charge la planification jusqu'au niveau de la seconde.

> Utiliser `workerman/crontab` nécessite de définir préalablement le fuseau horaire de PHP, sinon les résultats de l'exécution pourraient différer des attentes.

## Explication du temps
```text
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ jour de la semaine (0 - 6) (dimanche=0)
|   |   |   |   +------ mois (1 - 12)
|   |   |   +-------- jour du mois (1 - 31)
|   |   +---------- heure (0 - 23)
|   +------------ minute (0 - 59)
+-------------- seconde (0-59) [optionnel, si absent, la granularité minimale est la minute]
```

# Installation
```text
composer require workerman/crontab
```

# Exemple
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// Définir le fuseau horaire pour éviter les résultats d'exécution inattendus
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // Exécution chaque minute à la première seconde.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // Exécution à 7 heures 50 chaque jour, notez ici l'omission de la seconde.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> Remarque : Les tâches planifiées ne s'exécutent pas immédiatement, toutes les tâches planifiées commenceront à être exécutées à la prochaine minute.

# Interface
**Crontab::destroy()**

Détruire le minuteur
```text
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```
