# increment
**```(requires Workerman version >= 3.3.0)```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
Atomare Erhöhung. Erhöht ein numerisches Element um die angegebene Größe im Parameter "step". Wenn der Wert des Elements kein numerischer Typ ist, wird er als 0 behandelt. Wenn das Element nicht existiert, wird "false" zurückgegeben.

## Parameter

``` $key ```

Schlüssel. (zum Beispiel ```$global->abc```, hier ist ```abc``` der Schlüssel)

``` $value ```

Die Größe, um die das Element erhöht werden soll.

## Rückgabewert
TRUE bei Erfolg, andernfalls FALSE.

## Beispiel

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// Nicht-atomare Erhöhung
$global->some_key++;

echo $global->some_key."\n";

// Atomare Erhöhung
$global->increment('some_key');

echo $global->some_key."\n";
```
