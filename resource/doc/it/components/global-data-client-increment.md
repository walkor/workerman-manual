# increment
**``` (richiede la versione di Workerman >= 3.3.0) ```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
Incremento atomico. Incrementa un elemento numerico del valore specificato dal parametro step. Se il valore dell'elemento non è di tipo numerico, viene considerato come 0 e poi incrementato. Se l'elemento non esiste, restituisce false.

## Parametri

 ``` $key ```

Chiave dell'elemento. (ad esempio ```$global->abc```, ```abc``` è la chiave)

 ``` $value ```

Dimensione dell'incremento dell'elemento.

## Valore restituito
Restituisce true in caso di successo, altrimenti restituisce false.

## Esempio

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// Incremento non atomico
$global->some_key++;

echo $global->some_key."\n";

// Incremento atomico
$global->increment('some_key');

echo $global->some_key."\n";
```
