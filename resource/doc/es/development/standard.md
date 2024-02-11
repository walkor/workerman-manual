# Normas de desarrollo

## Directorio de la aplicación

El directorio de la aplicación puede ubicarse en cualquier lugar.

## Archivo de entrada

Al igual que en las aplicaciones PHP bajo nginx+PHP-FPM, las aplicaciones en Workerman también necesitan un archivo de entrada, cuyo nombre no tiene restricciones, y este archivo de entrada se ejecuta como PHP Cli.

El archivo de entrada contiene el código relacionado con la creación de procesos de escucha, como el fragmento de código basado en Worker a continuación.

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Crear un Worker que escuche en el puerto 2345 utilizando el protocolo http
$http_worker = new Worker("http://0.0.0.0:2345");

// Iniciar 4 procesos para proporcionar servicios externos
$http_worker->count = 4;

// Cuando se recibe un mensaje del navegador, responder con "hello world"
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Enviar "hello world" al navegador
    $connection->send('hello world');
};

Worker::runAll();
```

## Normas de código en Workerman

1. Los nombres de clase se escriben en mayúscula la primera letra de cada palabra, y el nombre del archivo de la clase debe ser igual al nombre interno de la clase, para permitir la carga automática. Por ejemplo:
```php
class UserInfo
{
...
```

2. Utilice espacios de nombres que coincidan con la ruta del directorio, tomando como base el directorio raíz del proyecto del desarrollador.

Por ejemplo, en el proyecto MyApp/, el archivo de clase MyApp/MyClass.php, al estar en el directorio raíz del proyecto, no necesita un espacio de nombres. Mientras que el archivo de clase MyApp/Protocols/MyProtocol.php, al estar en el directorio Protocols dentro del proyecto MyApp, debe incluir el espacio de nombres ```namespace Protocols;```, de la siguiente manera:

```php
namespace Protocols;
class MyProtocol
{
....
```

3. Los nombres de funciones y variables comunes se escriben en minúscula con guiones bajos. Por ejemplo:
```php
$connection_list = array();
function get_connection_list()
{
....
```

4. Los miembros de clase y los métodos de clase se escriben en minúscula la primera letra de la primera palabra, utilizando el formato de camello. Por ejemplo:
```php
public $connectionList;
public function getConnectionList();
```

5. Los argumentos de funciones y métodos de clase se escriben en minúscula con guiones bajos. Por ejemplo:
```php
function get_connection_list($one_param, $tow_param)
{
....
```
