# hinzufügen
**``` (Erfordert Workerman Version >= 3.3.0) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
Atomar hinzufügen. Wenn der Schlüssel bereits existiert, wird false zurückgegeben.

## Parameter

 ``` $key ```

Schlüsselwert. (zum Beispiel ```$global->abc```, hier ist ```abc``` der Schlüsselwert)


 ``` $value ```

Der zu speichernde Wert.

## Rückgabewert
Gibt true zurück, wenn erfolgreich, andernfalls false.

## Beispiel

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // $global->some_key erfolgreich zugewiesen
    echo "Hinzufügen erfolgreich: " , $global->some_key;
}
else
{
    // $global->some_key existiert bereits, Zuweisung fehlgeschlagen
    echo "Hinzufügen fehlgeschlagen: " , var_export($global->some_key);
}
```
