# Componente di condivisione delle variabili GlobalData
**``` (Richiede Workerman versione >=3.3.0) ```**

Indirizzo del codice sorgente: https://github.com/walkor/GlobalData

## Nota
GlobalData richiede Workerman versione >=3.3.0

## Installazione

`composer require workerman/globaldata`

## Principio

Utilizza i metodi magici ```__set __get __isset __unset``` di PHP per innescare la comunicazione con il server GlobalData, con le variabili effettivamente memorizzate sul server GlobalData. Ad esempio, quando si imposta un attributo inesistente per la classe client, viene innescato il metodo magico ```__set```, e la classe client invia una richiesta al server GlobalData per memorizzare una variabile. Quando si accede a una variabile inesistente per la classe client, viene innescato il metodo ```__get``` della classe, e il client invia una richiesta al server GlobalData per leggere questo valore, così da completare la condivisione delle variabili tra i processi.

```php
require_once __DIR__ . '/vendor/autoload.php';

// Connessione al server Global Data
$global = new GlobalData\Client('127.0.0.1:2207');

// Innescare $global->__isset('somedata') per controllare se il server ha memorizzato il valore associato alla chiave somedata
isset($global->somedata);

// Innescare $global->__set('somedata',array(1,2,3)) per notificare al server di memorizzare il valore associato a somedata come array(1,2,3)
$global->somedata = array(1,2,3);

// Innescare $global->__get('somedata') per leggere il valore associato a somedata dal server
var_export($global->somedata);

// Innescare $global->__unset('somedata') per notificare al server di eliminare somedata e il valore associato
unset($global->somedata);
```
