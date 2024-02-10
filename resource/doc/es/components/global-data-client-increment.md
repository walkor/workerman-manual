# increment
**``` (Requiere Workerman versión >= 3.3.0) ```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
Incremento atómico. Aumenta el valor de un elemento numérico por el tamaño especificado en el parámetro step. Si el valor del elemento no es de tipo numérico, se considera como 0 antes de realizar el incremento. Si el elemento no existe, devuelve false.

## Parámetros

 ``` $key ```

Clave del elemento. (Por ejemplo, en ```$global->abc```, ```abc``` es la clave)

 ``` $value ```

Tamaño por el cual incrementar el valor del elemento.

## Valor de retorno
Devuelve true si tiene éxito, de lo contrario devuelve false.

## Ejemplo

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// Incremento no atómico
$global->some_key++;

echo $global->some_key."\n";

// Incremento atómico
$global->increment('some_key');

echo $global->some_key."\n";
```
