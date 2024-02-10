# cas
**``` (Requires Workerman version >= 3.3.0) ```**
```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
Reemplazo atómico, reemplaza ```$old_value``` con ```$new_value```.
Sólo puede escribir el valor si, después de que el cliente actual haya obtenido el valor por última vez, el valor asociado a esa clave no ha sido modificado por otro cliente.

## Parámetros

 ``` $key ```

Clave. (Por ejemplo, si es ```$global->abc```, entonces ```abc``` es la clave)

 ``` $old_value ```

Valor antiguo

 ``` $new_value ```

Nuevo valor

## Valor devuelto
Devuelve true si el reemplazo fue exitoso, de lo contrario devuelve false.

## Notas:

Cuando múltiples procesos operan simultáneamente sobre una misma variable compartida, a veces es necesario considerar problemas de concurrencia.

Por ejemplo, los procesos A y B agregan un miembro a la lista de usuarios al mismo tiempo.
La variable de lista de usuarios para los procesos A y B es actualmente ```$global->user_list = array(1,2,3)```.
El proceso A modifica la variable ```$global->user_list``` y agrega el usuario 4.
El proceso B modifica la variable ```$global->user_list``` y agrega el usuario 5.
El proceso A establece la variable como ```$global->user_list = array(1,2,3,4)``` con éxito.
El proceso B establece la variable como ```$global->user_list = array(1,2,3,5)``` con éxito.
En este punto, la variable establecida por el proceso B sobrescribe la variable establecida por el proceso A, causando la pérdida de datos.

Esto se debe a que la lectura y la escritura no son operaciones atómicas, lo que resulta en problemas de concurrencia.
Para resolver este tipo de problemas de concurrencia, se puede utilizar la interfaz de reemplazo atómico (cas).
Antes de cambiar un valor, la interfaz cas verifica si otro proceso ha modificado ese valor basado en ```$old_value```.
Si ha habido modificaciones, el valor no se reemplaza y devuelve false. De lo contrario, se reemplaza y devuelve true.
Consulte el siguiente ejemplo.

**Nota:** 
En algunos casos, es aceptable que los datos compartidos sean sobrescritos concurrentemente, como el precio más alto actual en un sistema de subastas o el inventario actual de un producto.

## Ejemplo

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// Inicializar la lista
$global->user_list = array(1,2,3);

// Añadir de forma atómica un valor a user_list
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
