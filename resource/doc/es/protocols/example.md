# Algunos ejemplos

## Ejemplo 1

### Definición del protocolo
  * Los primeros 10 bytes están fijos para almacenar la longitud de todo el paquete de datos, rellenando con ceros si es necesario
  * El formato de los datos es XML

### Ejemplo del paquete de datos
```xml
0000000121<?xml version="1.0" encoding="ISO-8859-1"?>
<request>
    <module>user</module>
    <action>getInfo</action>
</request>
```
Donde 0000000121 representa la longitud del paquete de datos completo, seguido por el contenido del cuerpo del paquete en formato XML.

### Implementación del protocolo
```php
namespace Protocols;
class XmlProtocol
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < 10)
        {
            // Insuficientes 10 bytes, devuelve 0 para esperar más datos
            return 0;
        }
        $total_len = base_convert(substr($recv_buffer, 0, 10), 10, 10);
        return $total_len;
    }

    public static function decode($recv_buffer)
    {
        $body = substr($recv_buffer, 10);
        return simplexml_load_string($body);
    }

    public static function encode($xml_string)
    {
        $total_length = strlen($xml_string)+10;
        $total_length_str = str_pad($total_length, 10, '0', STR_PAD_LEFT);
        return $total_length_str . $xml_string;
    }
}
```

## Ejemplo 2

### Definición del protocolo
  * Los primeros 4 bytes son un entero sin signo en orden de bytes de red que indica la longitud de todo el paquete
  * La parte de los datos es una cadena JSON

### Ejemplo del paquete de datos
<pre>
****{"type":"message","content":"hello all"}
</pre>
Donde los primeros cuatro bytes representan un entero sin signo en orden de bytes de red, que es un carácter no visible, seguido directamente por los datos del paquete en formato JSON

### Implementación del protocolo
```php
namespace Protocols;
class JsonInt
{
    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer)<4)
        {
            return 0;
        }
        $unpack_data = unpack('Ntotal_length', $recv_buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($recv_buffer)
    {
        $body_json_str = substr($recv_buffer, 4);
        return json_decode($body_json_str, true);
    }

    public static function encode($data)
    {
        $body_json_str = json_encode($data);
        $total_length = 4 + strlen($body_json_str);
        return pack('N',$total_length) . $body_json_str;
    }
}
```

## Ejemplo 3 (Carga de archivos usando un protocolo binario)

### Definición del protocolo
```C
struct
{
  unsigned int total_len;  // Longitud de todo el paquete, en orden de bytes de red
  char         name_len;   // Longitud del nombre del archivo
  char         name[name_len]; // Nombre del archivo
  char         file[total_len - BinaryTransfer::PACKAGE_HEAD_LEN - name_len]; // Datos del archivo
}
```
### Ejemplo del protocolo
<pre> *****logo.png****************** </pre>
Donde los primeros cuatro caracteres ***** representan un entero sin signo en orden de bytes de red, el quinto * es un byte que almacena la longitud del nombre del archivo, seguido por el nombre del archivo y luego los datos binarios del archivo

### Implementación del protocolo
```php
namespace Protocols;
class BinaryTransfer
{
    // Longitud de la cabecera del protocolo
    const PACKAGE_HEAD_LEN = 5;

    public static function input($recv_buffer)
    {
        if(strlen($recv_buffer) < self::PACKAGE_HEAD_LEN)
        {
            return 0;
        }
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        return $package_data['total_len'];
    }


    public static function decode($recv_buffer)
    {
        $package_data = unpack('Ntotal_len/Cname_len', $recv_buffer);
        $name_len = $package_data['name_len'];
        $file_name = substr($recv_buffer, self::PACKAGE_HEAD_LEN, $name_len);
        $file_data = substr($recv_buffer, self::PACKAGE_HEAD_LEN + $name_len);
         return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        return $data;
    }
}
```

### Ejemplo de uso del protocolo en el servidor

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('BinaryTransfer://0.0.0.0:8333');
// Guardar el archivo en tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("upload success. save path $save_path");
};

Worker::runAll();
```

### Cliente de archivos client.php (simulando la carga de archivos con PHP)

```php
<?php
/** Cliente para subir archivos **/
// Dirección para la carga
$address = "127.0.0.1:8333";
// Verificar el parámetro de la ruta del archivo a cargar
if(!isset($argv[1]))
{
   exit("utilice php client.php \$file_path\n");
}
// Ruta del archivo a cargar
$file_to_transfer = trim($argv[1]);
// El archivo local a cargar no existe
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer no existe\n");
}
// Establecer la conexión de socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
// Configurar como bloqueante
stream_set_blocking($client, 1);
// Nombre del archivo
$file_name = basename($file_to_transfer);
// Longitud del nombre del archivo
$name_len = strlen($file_name);
// Datos binarios del archivo
$file_data = file_get_contents($file_to_transfer);
// Longitud de la cabecera del protocolo 4 bytes + 1 byte de longitud del nombre del archivo
$PACKAGE_HEAD_LEN = 5;
// Paquete del protocolo
$package = pack('NC', $PACKAGE_HEAD_LEN  + strlen($file_name) + strlen($file_data), $name_len) . $file_name . $file_data;
// Ejecutar la carga
fwrite($client, $package);
// Imprimir el resultado
echo fread($client, 8192),"\n";
``` 

### Ejemplo de uso del cliente
Ejecutar en la línea de comandos ```php client.php <ruta_del_archivo>```

Por ejemplo ```php client.php abc.jpg```
## Ejemplo Cuatro (Carga de archivos mediante protocolo de texto)

### Definición del protocolo

Formato json seguido de un salto de línea, el json contiene el nombre del archivo y los datos del archivo codificados en base64 (aumenta el tamaño en 1/3).

### Ejemplo del protocolo

{"file_name":"logo.png","file_data":"PD9waHAKLyo......"}\n

Se debe tomar en cuenta que al final hay un salto de línea, en PHP se representa con la cadena de caracteres de doble comilla ```"\n"```

### Implementación del protocolo

```php
namespace Protocols;
class TextTransfer
{
    public static function input($recv_buffer)
    {
        $recv_len = strlen($recv_buffer);
        if($recv_buffer[$recv_len-1] !== "\n")
        {
            return 0;
        }
        return strlen($recv_buffer);
    }

    public static function decode($recv_buffer)
    {
        // Desempaquetar
        $package_data = json_decode(trim($recv_buffer), true);
        // Obtener el nombre del archivo
        $file_name = $package_data['file_name'];
        // Obtener los datos del archivo codificados en base64
        $file_data = $package_data['file_data'];
        // Decodificar para obtener los datos binarios originales
        $file_data = base64_decode($file_data);
        // Devolver los datos
        return array(
             'file_name' => $file_name,
             'file_data' => $file_data,
         );
    }

    public static function encode($data)
    {
        // Se puede codificar los datos a enviar al cliente según sea necesario, aquí solo se devuelven los datos de texto sin cambios
        return $data;
    }
}

```

### Ejemplo de uso del protocolo en el servidor
Nota: La manera de escribir es la misma que en la carga de archivos binarios, es decir, se puede cambiar el protocolo con muy pocos cambios en el código de negocio.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('TextTransfer://0.0.0.0:8333');
// Guardar el archivo en la carpeta tmp
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $save_path = '/tmp/'.$data['file_name'];
    file_put_contents($save_path, $data['file_data']);
    $connection->send("Carga exitosa. Ruta de guardado: $save_path");
};

Worker::runAll();
```

### Cliente de archivos textclient.php (usando PHP para simular la carga del cliente)

```php
<?php
/** Cliente para cargar archivos **/
// Dirección de carga
$address = "127.0.0.1:8333";
// Comprobar el parámetro de la ruta del archivo a cargar
if(!isset($argv[1]))
{
   exit("Usar php client.php \$file_path\n");
}
// Ruta del archivo a cargar
$file_to_transfer = trim($argv[1]);
// El archivo a cargar no existe localmente
if(!is_file($file_to_transfer))
{
    exit("$file_to_transfer no existe\n");
}
// Establecer la conexión por socket
$client = stream_socket_client($address, $errno, $errmsg);
if(!$client)
{
    exit("$errmsg\n");
}
stream_set_blocking($client, 1);
// Nombre del archivo
$file_name = basename($file_to_transfer);
// Datos binarios del archivo
$file_data = file_get_contents($file_to_transfer);
// Codificación en base64
$file_data = base64_encode($file_data);
// Paquete de datos
$package_data = array(
    'file_name' => $file_name,
    'file_data' => $file_data,
);
// Protocolo de paquete en formato json y salto de línea
$package = json_encode($package_data)."\n";
// Cargar
fwrite($client, $package);
// Imprimir el resultado
echo fread($client, 8192),"\n";
```

### Ejemplo de uso del cliente
Ejecutar en la línea de comandos ```php textclient.php <ruta_del_archivo>```

Por ejemplo ```php textclient.php abc.jpg```
