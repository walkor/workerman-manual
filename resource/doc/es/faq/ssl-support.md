# Transferencia de cifrado-ssl/tls

**Pregunta:**

¿Cómo garantizar la seguridad de la comunicación entre Workerman?

**Respuesta:**

Una forma conveniente es agregar una capa de cifrado [SSL](https://baike.baidu.com/item/ssl) en la parte superior del protocolo de comunicación, como wss y el protocolo [https](https://baike.baidu.com/item/https) que se basan en la transmisión encriptada [SSL](https://baike.baidu.com/item/ssl), lo que es muy seguro. Workerman admite [SSL](https://baike.baidu.com/item/ssl) por sí mismo (```se requiere Workerman>=3.3.7```), solo es necesario configurar algunas propiedades para habilitar [SSL](https://baike.baidu.com/item/ssl).

Por supuesto, los desarrolladores también pueden implementar su propio mecanismo de cifrado utilizando ciertos algoritmos de cifrado y descifrado.

## Método para habilitar SSL en Workerman:

**Preparativos:**

1. Versión de Workerman no menor a 3.3.7.

2. Instalación de la extensión openssl en PHP.

3. Certificado emitido (archivos pem/crt y archivo de clave) almacenados en /etc/nginx/conf.d/ssl.

**Código:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Es recomendable utilizar un certificado emitido
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // También puede ser un archivo crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Si es un certificado autofirmado, esta opción debe estar habilitada
    )
);
// Aquí se establece el protocolo WebSocket, también se puede utilizar el protocolo HTTP u otros protocolos
$worker = new Worker('websocket://0.0.0.0:443', $context);
// Configuración para habilitar ssl
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Método para habilitar [SNI (Indicación del Nombre del Servidor)](https://baike.baidu.com/item/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA) en Workerman 

Puede implementar la vinculación de varios certificados en una misma IP y puerto.

**Combinación de archivos .pem y .key del certificado:**

Combine el contenido de los archivos .pem y .key de cada certificado, agregando el contenido del archivo .key al final del archivo .pem (en caso de que la .pem ya contenga la clave privada, este paso puede omitirse).

**Tenga en cuenta que se trata de un solo certificado, no de copiar todos los certificados en un solo archivo.**

Por ejemplo, después de combinar el contenido del archivo .pem de *host1.com*, el contenido del archivo pem combinado sería algo así:

```text
-----BEGIN CERTIFICATE-----
MIIGXTCBA...
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIFBzCCA...
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAA....
-----END RSA PRIVATE KEY-----
```

**Código:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // Habilitar SNI
        'SNI_server_certs' => [ // Configurar varios certificados
            'host1.com' => '/path/host1.com.pem', // Certificado 1
            'host2.com' => '/path/host2.com.pem', // Certificado 2
        ],
        'local_cert' => '/path/default.com.pem', // Certificado predeterminado
        'local_pk'   => '/path/default.com.key',
    )
);
$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
