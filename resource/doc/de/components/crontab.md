# workerman/crontab

# Erklärung
`workerman/crontab` ist ein auf Workerman basierendes Zeitplanungsprogramm, ähnlich wie der Linux-Crontab. `workerman/crontab` unterstützt Zeitpläne bis zur Sekundenebene.

> Bei der Verwendung von `workerman/crontab` muss die Zeitzone von PHP zuerst korrekt eingestellt werden, da andernfalls das Ausführungsergebnis von den Erwartungen abweichen kann.

## Zeitangaben
```php
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ Wochentag (0 - 6) (Sonntag=0)
|   |   |   |   +------ Monat (1 - 12)
|   |   |   +-------- Tag des Monats (1 - 31)
|   |   +---------- Stunde (0 - 23)
|   +------------ Minute (0 - 59)
+-------------- Sekunde (0-59) [kann weggelassen werden, falls Position 0 fehlt, ist die minimale zeitliche Auflösung die Minute]
```

# Installation
```php
composer require workerman/crontab
```

# Beispiel
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// Setzen der Zeitzone, um Abweichungen der Ausführungsergebnisse zu vermeiden
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // Jede Minute in der 1. Sekunde ausführen.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // Jeden Tag um 7:50 Uhr ausführen, beachten Sie, dass hier die Sekunde weggelassen wurde.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> Hinweis: Zeitpläne werden nicht sofort ausgeführt, alle Zeitpläne beginnen erst in der nächsten Minute zu zählen.

# Schnittstelle
**Crontab::destroy()**

Timer zerstören
```php
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```
