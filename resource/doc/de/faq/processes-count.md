# Wie viele Prozesse sollten gestartet werden

## Festlegung der Prozessanzahl
Die Anzahl der Prozesse wird durch das Attribut ```count``` bestimmt (Windows-Systeme unterstützen keine Prozessanzahl), wie im folgenden Code dargestellt:
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// ## Starten von 4 Prozessen für den externen Service ##
$http_worker->count = 4;

... 

```

## Bei der Festlegung der Prozessanzahl sind folgende Punkte zu berücksichtigen
1. Anzahl der CPU-Kerne
2. Größe des Arbeitsspeichers
3. Neigung des Geschäfts zur IO-Intensität oder CPU-Intensität

## Grundsätze für die Festlegung der Prozessanzahl
1. Die Gesamtsumme des von jedem Prozess belegten Speichers sollte kleiner sein als der Gesamtspeicher (normalerweise beträgt der Speicherbedarf pro Geschäftsprozess etwa 40 MB)
2. Bei IO-intensiven Geschäften, d.h. Geschäfte, die einige **blockierende** IO-Vorgänge beinhalten, wie z.B. der Zugriff auf Speicher wie Mysql, Redis usw., die blockierende Vorgänge sind, kann die Anzahl der Prozesse erhöht werden, beispielsweise auf das 3-fache der Anzahl der CPU-Kerne. Wenn das Geschäft viele blockierende Wartezeiten beinhaltet, kann die Anzahl der Prozesse angemessen erhöht werden, z.B. das 8-fache oder sogar mehr der Anzahl der CPU-Kerne. Beachten Sie, dass **nicht blockierende** IO zu den CPU-intensiven Geschäften gehört und nicht zu den IO-intensiven Geschäften.
3. Für CPU-intensive Geschäfte, d.h. Geschäfte, die keine **blockierenden** IO-Kosten haben, wie z.B. das asynchrone Lesen von Netzwerkressourcen, bei dem die Prozesse nicht durch den Geschäftscode blockiert werden, kann die Anzahl der Prozesse auf die Anzahl der CPU-Kerne festgelegt werden.


## Richtwerte für die Festlegung der Prozessanzahl
Wenn der Geschäftscode zur IO-Intensität neigt, sollte die Anzahl der Prozesse entsprechend der Intensität der IO festgelegt werden, z.B. das 3- bis 8-fache der Anzahl der CPU-Kerne.

Wenn der Geschäftscode zur CPU-Intensität neigt, kann die Anzahl der Prozesse auf die Anzahl der CPU-Kerne festgelegt werden.

## Beachten Sie
Das IO von Workerman selbst ist nicht blockierend, z.B. ```Connection->send``` ist nicht blockierend und gehört zu den CPU-intensiven Operationen. Wenn Sie sich nicht sicher sind, ob Ihr Geschäft zur IO- oder zur CPU-Intensität neigt, können Sie die Anzahl der Prozesse auf etwa das 3-fache der Anzahl der CPU-Kerne festlegen. Darüber hinaus gilt, dass eine höhere Anzahl von Prozessen nicht unbedingt besser ist, denn zu viele Prozesse erhöhen die Umschaltkosten und beeinträchtigen die Leistung.
