# Componente de Variables Globales Compartidas GlobalData

**```(Requiere versión de Workerman >= 3.3.0)```**

Código fuente: https://github.com/walkor/GlobalData

## Nota
GlobalData requiere una versión de Workerman >= 3.3.0

## Instalación

```composer require workerman/globaldata```

## Principio

Utiliza los métodos mágicos ```__set __get __isset __unset``` de PHP para desencadenar la comunicación con el servidor GlobalData, donde las variables reales se almacenan en el servidor GlobalData. Por ejemplo, cuando se le asigna una propiedad inexistente a una clase cliente, se desencadena el método mágico ```__set```, y la clase cliente envía una solicitud al servidor GlobalData para almacenar una variable. Cuando se accede a una variable inexistente de la clase cliente, se desencadena el método ```__get``` de la clase, y el cliente envía una solicitud al servidor GlobalData para leer este valor, completando así el intercambio de variables entre procesos.

```php
require_once __DIR__ . '/vendor/autoload.php';

// Conectarse al servidor Global Data
$global = new GlobalData\Client('127.0.0.1:2207');

// Desencadena $global->__isset('somedata') para consultar al servidor si existe una clave llamada somedata almacenada
isset($global->somedata);

// Desencadena $global->__set('somedata',array(1,2,3)), notificando al servidor para almacenar el valor array(1,2,3) correspondiente a somedata
$global->somedata = array(1,2,3);

// Desencadena $global->__get('somedata'), consultando al servidor el valor correspondiente a somedata
var_export($global->somedata);

// Desencadena $global->__unset('somedata'), notificando al servidor para eliminar somedata junto con su valor correspondiente
unset($global->somedata);
```
