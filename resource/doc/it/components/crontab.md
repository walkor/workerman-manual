# workerman/crontab

# Descrizione
`workerman/crontab` è un programma di attività pianificata basato su workerman, simile a crontab di Linux. `workerman/crontab` supporta la pianificazione fino al secondo.

>L'uso di `workerman/crontab` richiede di impostare prima il fuso orario di PHP, altrimenti i risultati potrebbero non corrispondere alle aspettative.

## Descrizione del tempo
```plaintext
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ giorno della settimana (0 - 6) (domenica=0)
|   |   |   |   +------ mese (1 - 12)
|   |   |   +-------- giorno del mese (1 - 31)
|   |   +---------- ora (0 - 23)
|   +------------ minuto (0 - 59)
+-------------- secondo (0-59) [opzionale, se assente, la granularità temporale minima è il minuto]
```

# Installazione
```plaintext
composer require workerman/crontab
```

# Esempio
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// Imposta il fuso orario per evitare discrepanze nei risultati
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // Esegui ogni minuto al primo secondo.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // Esegui alle 7:50 ogni giorno, nota che qui abbiamo omesso i secondi.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> Nota: le attività pianificate non verranno eseguite immediatamente, tutte le attività pianificate inizieranno a essere eseguite al passaggio al minuto successivo.

# Interfacce
**Crontab::destroy()**

Distrugge il timer
```plaintext
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```
