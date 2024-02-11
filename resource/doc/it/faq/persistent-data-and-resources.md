# Persistenza degli oggetti e delle risorse

Nello sviluppo Web tradizionale, gli oggetti, i dati e le risorse creati da PHP vengono rilasciati completamente al termine della richiesta, rendendo difficile garantirne la persistenza. Workerman, invece, consente di raggiungere questo obiettivo in modo semplice.

In Workerman, se si desidera mantenere permanentemente determinate risorse in memoria, è possibile inserirle in una variabile globale o in un membro statico di una classe.

Ad esempio, nel seguente codice:

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Variabile globale per mantenere il numero di connessioni dei client in questo processo
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // Quando si stabilisce una nuova connessione client, incrementa il numero di connessioni
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // Quando un client si disconnette, decrementa il numero di connessioni
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};
```

## Vedere la documentazione PHP sulla portata delle variabili:
https://php.net/manual/it/language.variables.scope.php
