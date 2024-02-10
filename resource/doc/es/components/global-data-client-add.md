# add
**``` (Requires Workerman version>=3.3.0) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
Agrega de forma atómica. Si la clave ya existe, devolverá false.

## Parámetros

 ``` $key ```

Clave. (por ejemplo, si es ```$global->abc```, entonces ```abc``` es la clave)


 ``` $value ```

Valor a almacenar.

## Valor de retorno
Devuelve true si tiene éxito, de lo contrario devuelve false.


## Ejemplo

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // Se asignó con éxito la variable $global->some_key
    echo "Agregado con éxito " , $global->some_key;
}
else
{
    // La variable $global->some_key ya existe, no se realizó la asignación
    echo "Falló al agregar " , var_export($global->some_key);
}
```
