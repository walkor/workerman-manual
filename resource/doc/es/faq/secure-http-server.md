# Crear un servicio https

**Pregunta:**

¿Cómo puede Workerman crear un servicio [https](https://baike.baidu.com/item/https), de modo que los clientes puedan comunicarse a través del protocolo [https](https://baike.baidu.com/item/https)?

**Respuesta:**

El protocolo [https](https://baike.baidu.com/item/https) es en realidad una combinación de [http](https://baike.baidu.com/item/http) y [SSL](https://baike.baidu.com/item/ssl), que agrega una capa de [SSL](https://baike.baidu.com/item/ssl) sobre el [http](https://baike.baidu.com/item/http). Workerman es compatible con el protocolo [http](https://baike.baidu.com/item/http) y, a partir de la versión 3.3.7, también es compatible con [SSL](https://baike.baidu.com/item/ssl) (```se requiere Workerman versión >= 3.3.7``). Por lo tanto, solo es necesario habilitar [SSL](https://baike.baidu.com/item/ssl) sobre el protocolo [http](https://baike.baidu.com/item/http) para admitir el protocolo [https](https://baike.baidu.com/item/https).

Hay dos enfoques comunes para habilitar [https](https://baike.baidu.com/item/https) en Workerman, uno es habilitar directamente SSL en Workerman, y el otro es utilizar nginx como proxy de SSL. Solo se puede seleccionar uno de estos enfoques, no ambos a la vez.

## Habilitar SSL en Workerman

**Preparativos:**

1. Workerman versión >= 3.3.7
2. PHP con la extensión openssl instalada
3. Certificado válido (archivos pem/crt y clave) colocados en /etc/nginx/conf.d/ssl

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Es recomendable utilizar un certificado válido
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // también puede ser un archivo crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // debe activarse para certificados autofirmados
    )
);
// Configurar la conexión como http
$worker = new Worker('http://0.0.0.0:443', $context);
// Habilitar SSL como transporte, convirtiendo http en http+SSL (https)
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Con el código anterior, se crea un servicio https en Workerman, permitiendo que los clientes se conecten y comuniquen de forma segura a través del protocolo https.

**Prueba:**

Abrir un navegador y visitar ```https://dominio:443```.

**Nota:**

1. El puerto https solo puede ser accedido a través del protocolo https, no se admite http.
2. Dado que los certificados suelen estar vinculados a un nombre de dominio, al realizar pruebas, se debe usar un nombre de dominio en lugar de una dirección IP.
3. Si hay problemas para acceder a través de https, se debe verificar el firewall del servidor.

## Usar nginx como proxy SSL

Además de habilitar SSL directamente en Workerman, también se puede utilizar nginx como proxy SSL para ofrecer https.

> **Nota**
> Se debe elegir entre usar nginx como proxy SSL o habilitar SSL en Workerman, no ambas al mismo tiempo.

El funcionamiento y el flujo de comunicación son:

1. El cliente inicia una conexión https con nginx
2. Nginx convierte los datos del protocolo https a http y los envía al puerto http de Workerman
3. Workerman recibe los datos, realiza el procesamiento lógico y devuelve los datos del protocolo http a nginx
4. Nginx convierte los datos del protocolo http a https y los envía al cliente

### Configuración de nginx de referencia

**Requisitos previos y preparativos:**

1. Suponiendo que Workerman escucha en el puerto 8181 (protocolo http)
2. Se han obtenido un certificado (archivos pem/crt y clave) que se han colocado en /etc/nginx/conf.d/ssl
3. Se planea que nginx abra el puerto 443 para ofrecer un servicio de proxy wss (el puerto se puede modificar según sea necesario)

**La configuración de nginx sería similar a la siguiente**:

```
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}

server {
  listen 443;
  server_name nombre-del-sitio.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /
  {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
