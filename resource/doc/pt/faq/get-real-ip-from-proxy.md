## Como obter o IP real do cliente através de um proxy Nginx/Apache?

Quando se utiliza Nginx/Apache como proxy para o Workerman, na verdade Nginx/Apache age como o cliente do Workerman. Portanto, o IP do cliente obtido no Workerman será o IP do servidor Nginx/Apache, e não o IP real do cliente. Para obter o IP real do cliente, pode-se seguir o método abaixo.

**Princípio:**

Nginx/Apache passa o IP real do cliente através do cabeçalho HTTP, por exemplo, adicionando `proxy_set_header X-Real-IP $remote_addr;` na configuração do Nginx. O Workerman lê este cabeçalho e armazena o valor no objeto `$connection` (no GatewayWorker, pode ser armazenado na variável `$_SESSION`), o qual pode ser acessado quando necessário.

**Observação:**

As configurações a seguir são aplicáveis aos protocolos HTTP/HTTPS e WS/WSS. Para outros protocolos, o método para obter o IP do cliente é similar, necessitando que o servidor proxy insira a real IP do cliente nos pacotes de dados.

**Exemplo de configuração do Nginx:**
```nginx
server {
  listen 443;

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
    # Esta seção utiliza o cabeçalho HTTP para transmitir o IP real do cliente
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # outras configurações do site ...
}
```

**Leitura do IP do cliente a partir do cabeçalho configurado no Nginx no Workerman:**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:7272');

// Callback quando o cliente se conecta, após a conclusão do handshake TCP
$worker->onConnect = function(TcpConnection $connection) {
   /**
    * Callback onWebSocketConnect ao realizar o handshake do WebSocket com o cliente
    * Neste callback é possível obter o valor X_REAL_IP do cabeçalho HTTP do Nginx
    */
   $connection->onWebSocketConnect = function(TcpConnection $connection){
       /**
        * O objeto de conexão originalmente não possui a propriedade realIP, então aqui ela é dinamicamente adicionada
        * Lembre-se que em PHP é possível adicionar dinamicamente propriedades ao objeto, ou utilize o nome de sua preferência
        */
       $connection->realIP = $_SERVER['HTTP_X_REAL_IP'];
   };
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Ao utilizar o IP real do cliente, basta acessar $connection->realIP diretamente
    $connection->send($connection->realIP);
};
Worker::runAll();
```

**Obtenção do IP do cliente a partir do cabeçalho configurado no Nginx no GatewayWorker:**

Adicione o seguinte código em Events.php
```php
class Events
{
   public static function onWebsocketConnect($client_id, $data)
   {    
        $_SESSION['realIP'] = $data['server']['HTTP_X_REAL_IP'];
   }
   // .... Código omitido ....
}
```
Após adicionar o código, é necessário reiniciar o GatewayWorker.

Desta forma, é possível obter o IP real do cliente em `onMessage` e `onClose` através de `$_SESSION['realIP']` em Events.php.

> Observação: Em `onWorkerStart`, `onConnect` e `onWorkerStop` de Events.php, `$_SESSION['realIP']` não pode ser acessado diretamente.
