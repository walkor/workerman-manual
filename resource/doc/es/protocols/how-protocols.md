## Cómo personalizar un protocolo

En realidad, personalizar un protocolo es algo bastante sencillo. Un protocolo simple generalmente consta de dos partes:
* Identificación de los límites de los datos
* Definición del formato de los datos

## Un ejemplo

### Definición del protocolo
Supongamos que el símbolo que identifica los límites de los datos es el salto de línea "\n" (es importante notar que los datos de la solicitud en sí no pueden contener el carácter de salto de línea). El formato de los datos es JSON. A continuación, se muestra un ejemplo de un paquete de solicitud que cumple con esta regla.

```json
{"type":"message","content":"hello"}
```

Es importante observar que al final de los datos de la solicitud hay un carácter de salto de línea (representado por la cadena "\n" en PHP con **comillas dobles**), que indica el final de la solicitud.

### Pasos de implementación
En Workerman, si deseas implementar el protocolo mencionado anteriormente (supongamos que se llama JsonNL) en un proyecto llamado MyApp, debes seguir los siguientes pasos:

1. Colocar el archivo del protocolo en la carpeta Protocols del proyecto, por ejemplo, el archivo sería MyApp/Protocols/JsonNL.php.

2. Implementar la clase JsonNL con el espacio de nombres `namespace Protocols;`, y asegurarse de implementar tres métodos estáticos: input, encode y decode.

Es importante destacar que Workerman llamará automáticamente a estos tres métodos estáticos para implementar la división, el empaquetado y el desempaquetado. Para obtener más detalles sobre el flujo de ejecución, consulta la explicación detallada a continuación.

### Flujo de interacción entre Workerman y la clase de protocolo
1. Supongamos que el cliente envía un paquete de datos al servidor. Cuando el servidor recibe los datos (puede ser parte de los datos), inmediatamente llama al método `input` del protocolo para verificar la longitud de este paquete. El método `input` devuelve el valor de longitud `$length` a Workerman.
2. Una vez que Workerman recibe este valor de `$length`, comprueba si la longitud de los datos en el búfer actual es igual o mayor que `$length`. Si no es así, continúa esperando datos hasta que la longitud del búfer sea igual o mayor que `$length`.
3. Cuando la longitud del búfer es suficiente, Workerman corta los datos del búfer con una longitud de `$length` (es decir, **divide el paquete**) y llama al método `decode` del protocolo para **desempaquetar** los datos. El resultado del desempaquetado se almacena en la variable `$data`.
4. Una vez desempaquetados, Workerman pasa los datos `$data` al negocio mediante la devolución de llamada `onMessage($connection, $data)`. El negocio puede utilizar la variable `$data` para obtener los datos completos y desempaquetados enviados por el cliente.
5. Si el negocio necesita enviar datos al cliente mediante la llamada a `$connection->send($buffer)`, Workerman automáticamente utiliza el método `encode` del protocolo para **empaquetar** `$buffer` antes de enviarlo al cliente.

### Implementación específica

**Implementación de JsonNL en MyApp/Protocols/JsonNL.php**

```php
namespace Protocols;
class JsonNL
{
    public static function input($buffer)
    {
        $pos = strpos($buffer, "\n");
        if($pos === false)
        {
            return 0;
        }
        return $pos+1;
    }

    public static function encode($buffer)
    {
        return json_encode($buffer)."\n";
    }

    public static function decode($buffer)
    {
        return json_decode(trim($buffer), true);
    }
}
```

Con esto, se completa la implementación del protocolo JsonNL, el cual puede ser utilizado en el proyecto MyApp como se muestra a continuación.

Archivo: MyApp\start.php

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data representa los datos enviados por el cliente, los cuales ya han sido procesados por JsonNL::decode
    echo $data;
    
    // Los datos de $connection->send se empaquetarán automáticamente mediante JsonNL::encode antes de ser enviados al cliente
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
```

> **Nota**
> Workerman intentará cargar el protocolo dentro del espacio de nombres `Protocols`, por ejemplo, con `new Worker('JsonNL://0.0.0.0:1234')` intentará cargar el protocolo `Protocols\JsonNL`. Si se produce un error como "Clase 'Protocols\JsonNL' no encontrada", consulta la implementación de [carga automática](../faq/autoload.md) para hacerla automáticamente.

### Explicación de la interfaz del protocolo

Para el desarrollo en Workerman, la clase del protocolo debe implementar tres métodos estáticos: input, encode y decode. Para obtener más detalles sobre la interfaz del protocolo, consulta Workerman/Protocols/ProtocolInterface.php, que se define de la siguiente manera:

```php
namespace Workerman\Protocols;
use \Workerman\Connection\ConnectionInterface;

interface ProtocolInterface
{
    public static function input($recv_buffer, ConnectionInterface $connection);
    public static function decode($recv_buffer, ConnectionInterface $connection);
    public static function encode($data, ConnectionInterface $connection);
}
```

## Nota:
En Workerman, no es estrictamente necesario que la clase del protocolo implemente la interfaz ProtocolInterface. En realidad, solo se requiere que la clase del protocolo contenga los tres métodos estáticos: input, encode y decode.
