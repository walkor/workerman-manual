# Aggiungi
**``` (Richiede Workerman versione >=3.3.0) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
Aggiunta atomica. Se la chiave esiste già, restituirà false.

## Parametri

``` $key ```

Chiave. (Ad esempio, ```$global->abc```, in questo caso ```abc``` è la chiave)

``` $value ```

Valore da memorizzare.

## Valore restituito
Restituisce true in caso di successo, altrimenti restituisce false.


## Esempio

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // Assegnazione riuscita per $global->some_key
    echo "aggiunta riuscita " , $global->some_key;
}
else
{
    // $global->some_key già esistente, assegnazione fallita
    echo "aggiunta fallita " , var_export($global->some_key);
}
```
