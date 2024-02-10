# Crear un servicio wss

**Pregunta:**

¿Cómo se crea un servicio wss en Workerman para que los clientes puedan comunicarse a través de wss, por ejemplo, conectarse desde una aplicación de mini programa de WeChat?

**Respuesta:**

El protocolo wss en realidad es [websocket](https://baike.baidu.com/item/WebSocket)+[SSL](https://baike.baidu.com/item/ssl), es decir, agrega una capa [SSL](https://baike.baidu.com/item/ssl) al protocolo websocket, similar a [https](https://baike.baidu.com/item/https)([http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl)). Por lo tanto, solo es necesario habilitar [SSL](https://baike.baidu.com/item/ssl) sobre el protocolo [websocket](https://baike.baidu.com/item/WebSocket) para admitir el protocolo wss.

## Método 1: Utilizar un proxy SSL a través de nginx/apache (recomendado)

**Principio y flujo de comunicación:**

1. El cliente establece una conexión wss con nginx/apache.
2. Nginx/apache convierte los datos del protocolo wss a datos del protocolo ws y los reenvía al puerto del protocolo websocket de Workerman.
3. Workerman recibe los datos y realiza el procesamiento lógico.
4. Cuando Workerman envía un mensaje al cliente, el proceso es inverso: los datos pasan por nginx/apache, se convierten en protocolo wss y se envían al cliente.

## Ejemplo de configuración de nginx

**Requisitos previos y preparativos:**

1. Tener instalado nginx, con una versión no inferior a 1.3.
2. Suponiendo que Workerman está escuchando en el puerto 8282 (protocolo websocket).
3. Se ha obtenido un certificado (archivos pem/crt y key) y se asume que se han guardado en /etc/nginx/conf.d/ssl.
4. Planea utilizar nginx para habilitar el puerto 443 como servicio proxy wss (el puerto puede modificarse según sea necesario).
5. En general, nginx también se ejecuta como servidor web para otros servicios. Para no afectar el uso de los sitios web existentes, se utilizará la dirección "dominio.com/wss" como la entrada de proxy wss. Es decir, la dirección de conexión para el cliente será wss://dominio.com/wss.

**Configuración de nginx:**

```nginx
server {
  listen 443;
  # Resto de la configuración del dominio...

  ssl on;
  ssl_certificate /etc/ssl/server.pem;
  ssl_certificate_key /etc/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /wss
  {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} Resto de la configuración del sitio web...
}
```

**Prueba:**

```javascript
// El certificado verifica el nombre de dominio, por lo que debe usar el nombre de dominio para la conexión. No incluir el puerto aquí.
ws = new WebSocket("wss://dominio.com/wss");

ws.onopen = function() {
    alert("Conexión exitosa");
    ws.send('tom');
    alert("Enviando una cadena al servidor: tom");
};
ws.onmessage = function(e) {
    alert("Mensaje recibido del servidor: " + e.data);
};
```

## Utilizar Apache como proxy wss

También se puede utilizar Apache como proxy wss para hacer reenvío a Workerman.

**Preparativos:**

1. GatewayWorker escucha en el puerto 8282 (protocolo websocket).
2. Se ha obtenido un certificado SSL, guardado en /server/httpd/cert/.
3. Configurar Apache para reenviar el puerto 443 al puerto 8282.

**Habilitar el módulo proxy_wstunnel_module:**
```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**Configuración SSL y proxy:**
```apache
#extra/httpd-ssl.conf
DocumentRoot "/ruta/del/sitio"
ServerName dominio

# Configuración del proxy
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# Agregar soporte para protocolos SSL, eliminar los protocolos inseguros
SSLProtocol all -SSLv2 -SSLv3
# Modificar suite de cifrado como se indica a continuación
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# Configuración de la clave pública del certificado
SSLCertificateFile /server/httpd/cert/your.pem
# Configuración de la clave privada del certificado
SSLCertificateKeyFile /server/httpd/cert/your.key
# Configuración de la cadena del certificado
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**Prueba:**

```javascript
// El certificado verifica el nombre de dominio, por lo que debe usar el nombre de dominio para la conexión. No incluir el puerto aquí.
ws = new WebSocket("wss://dominio.com/wss");

ws.onopen = function() {
    alert("Conexión exitosa");
    ws.send('tom');
    alert("Enviando una cadena al servidor: tom");
};
ws.onmessage = function(e) {
    alert("Mensaje recibido del servidor: " + e.data);
};
```

## Método 2: Habilitar SSL directamente en Workerman (no recomendado)

> **Nota:**
> No se pueden habilitar simultáneamente el proxy SSL desde nginx/Apache y la configuración SSL en Workerman.

**Preparativos:**

1. Versión de Workerman >= 3.3.7.
2. PHP tiene instalada la extensión openssl.
3. Se ha obtenido un certificado (archivos pem/crt y key) y se han guardado en el disco en cualquier directorio.

**Código:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Se recomienda usar un certificado válido
$context = array(
    // Para más opciones de SSL, consulta el manual en http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // Utilizar rutas absolutas
        'local_cert'        => 'ruta/al/dispositivo/server.pem', // También puede ser un archivo crt
        'local_pk'          => 'ruta/al/dispositivo/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Habilitar si es un certificado autofirmado
    )
);
// Establecer websocket como protocolo (el puerto es arbitrario pero no debe estar ocupado por otro programa)
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// Habilitar el transporte SSL, websocket + SSL equivale a wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Con este código, Workerman estará escuchando el protocolo wss, permitiendo que los clientes se conecten a través de wss para lograr una comunicación segura en tiempo real.

**Prueba:**

Abrir el navegador Chrome, presionar F12 para abrir la consola de depuración y en la pestaña de Consola, ingresar (o colocar el siguiente código en una página HTML y ejecutarlo con JavaScript):

```javascript
// El certificado verifica el nombre de dominio, por lo que debe usar el nombre de dominio para la conexión. Incluir el puerto aquí.
ws = new WebSocket("wss://dominio.com:8282");
ws.onopen = function() {
    alert("Conexión exitosa");
    ws.send('tom');
    alert("Enviando una cadena al servidor: tom");
};
ws.onmessage = function(e) {
    alert("Mensaje recibido del servidor: " + e.data);
};
```

**Observaciones:**

1. Si es necesario usar el puerto 443, utilice el primer método (proxy wss a través de nginx/Apache) para habilitar wss.
2. El puerto wss solo puede accederse a través del protocolo wss; no se puede acceder a un puerto wss mediante el protocolo ws.
3. Los certificados generalmente están vinculados a un dominio, por lo que al realizar pruebas, los clientes deben usar el nombre de dominio para la conexión en lugar de una dirección IP.
4. Si la conexión no funciona, verifique las reglas del firewall del servidor.
5. Este método requiere PHP versión >= 5.6, ya que los mini programas de WeChat requieren TLS 1.2, y las versiones de PHP por debajo de 5.6 no admiten TLS 1.2.

Artículos relacionados:
- [Obtener la IP real del cliente a través de un proxy](get-real-ip-from-proxy.md)
- [Referencia de opciones de contexto SSL en Workerman](https://php.net/manual/zh/context.ssl.php)
