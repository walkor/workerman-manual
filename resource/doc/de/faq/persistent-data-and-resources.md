# Objekte und Ressourcenpersistenz
In der herkömmlichen Webentwicklung werden von PHP erstellte Objekte, Daten und Ressourcen nach Abschluss der Anfrage freigegeben, was die Persistenz sehr schwierig macht. Mit Workerman ist es jedoch einfach, dies zu erreichen.

In Workerman können Daten und Ressourcen dauerhaft im Speicher gespeichert werden, indem sie in globale Variablen oder statische Klassenmember platziert werden.

Zum Beispiel der folgende Code:

Speichern Sie die Anzahl der Client-Verbindungen für den aktuellen Prozess in der globalen Variablen ```$connection_count```.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Globale Variable, um die Anzahl der Client-Verbindungen für den aktuellen Prozess zu speichern
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // Wenn eine neue Client-Verbindung hergestellt wird, wird die Anzahl der Verbindungen um 1 erhöht
    global $connection_count;
    ++$connection_count;
    echo "Verbindungsanzahl jetzt: $connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // Beim Schließen des Clients wird die Anzahl der Verbindungen um 1 verringert
    global $connection_count;
    $connection_count--;
    echo "Verbindungsanzahl jetzt: $connection_count\n";
};
```

## Weitere Informationen zum PHP-Variablenbereich finden Sie unter:
https://php.net/manual/zh/language.variables.scope.php
