# Criar um serviço wss

**Pergunta:**

Como criar um serviço wss no Workerman, de modo que os clientes possam se conectar e se comunicar usando o protocolo wss, por exemplo, em um mini-programa WeChat?

**Resposta:**

O protocolo wss é na verdade uma combinação do [websocket](https://baike.baidu.com/item/WebSocket) com [SSL](https://baike.baidu.com/item/ssl), onde o SSL é adicionado como camada sobre o protocolo websocket, semelhante ao [https](https://baike.baidu.com/item/https) ([http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl)).
Portanto, é necessário apenas habilitar o [SSL](https://baike.baidu.com/item/ssl) sobre o protocolo [websocket](https://baike.baidu.com/item/WebSocket) para suportar o protocolo wss.

## Método 1: Utilizando Nginx/Apache como proxy SSL (recomendado)

**Princípio e Fluxo de Comunicação:**

1. O cliente inicia a conexão wss com o Nginx/Apache.
2. O Nginx/Apache converte os dados do protocolo wss para dados do protocolo ws e encaminha para a porta do protocolo websocket do Workerman.
3. O Workerman recebe os dados e os processa logicamente.
4. Quando o Workerman envia mensagens para o cliente, o processo é reverso: os dados são convertidos em protocolo wss pelo Nginx/Apache e, em seguida, enviados para o cliente.

## Referência de Configuração do Nginx

**Pré-requisitos e Preparação:**

1. Nginx instalado, versão não inferior a 1.3.
2. Supondo que o Workerman esteja ouvindo na porta 8282 (protocolo websocket).
3. Certificado SSL (arquivos pem/crt e chave) foi adquirido e está localizado em /etc/nginx/conf.d/ssl.
4. Pretende-se utilizar o Nginx para oferecer o serviço de proxy wss na porta 443 (a porta pode ser modificada conforme necessário).
5. Como o Nginx geralmente executa outros serviços como servidor de sites, para não afetar o uso do site original, será utilizado o endereço `domain.com/wss` como entrada de proxy para wss. Portanto, o cliente se conecta ao endereço wss://domain.com/wss.

**Exemplo de Configuração do Nginx:**

```nginx
server {
  listen 443;
  # Configurações de domínio omitidas...

  ssl on;
  ssl_certificate /etc/ssl/server.pem;
  ssl_certificate_key /etc/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /wss {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} - outras configurações do site...
}
```

**Teste:**

```javascript
// O certificado verifica o nome de domínio; por favor, use o nome de domínio para se conectar. Observe que a porta não é especificada.
ws = new WebSocket("wss://domain.com/wss");

ws.onopen = function() {
    alert("Conexão bem-sucedida");
    ws.send('tom');
    alert("Enviando uma string para o servidor: tom");
};
ws.onmessage = function(e) {
    alert("Mensagem recebida do servidor: " + e.data);
};
```

## Uso do Apache como Proxy WSS

Tambem é possível usar o Apache como proxy para encaminhar o WSS para o Workerman.

## Método 2: Usar o Workerman para Abrir o SSL (não recomendado)

> **Nota:**
> O proxy SSL do Nginx/Apache e a configuração SSL no Workerman são mutuamente exclusivos; não devem ser habilitados simultaneamente.

**Preparação:**

1. Workerman versão >= 3.3.7.
2. PHP com a extensão openssl instalada.
3. Um certificado (arquivos pem/crt e chave) armazenado em qualquer diretório no disco.

**Código:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// É preferível usar um certificado adquirido
$context = array(
    // Mais opções SSL estão disponíveis na documentação: http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // Use caminhos absolutos
        'local_cert'        => 'caminho_no_disco/server.pem', // Também pode ser um arquivo crt
        'local_pk'          => 'caminho_no_disco/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Se for um certificado autoassinado, esta opção deve ser habilitada
    )
);
// Aqui está configurado o protocolo websocket (a porta é arbitrária, desde que não esteja em uso por outro programa)
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// Define o transporte para SSL, para usar websocket com SSL (wss)
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Com esse código, o Workerman estará ouvindo o protocolo wss, permitindo que os clientes se conectem usando o protocolo wss e comuniquem-se com o Workerman de forma segura e em tempo real.

**Teste:**

Abra o console no navegador Chrome usando F12 e insira o seguinte código:

```javascript
// O certificado verifica o nome de domínio; por favor, use o nome de domínio para se conectar, observando que a porta está especificada.
ws = new WebSocket("wss://domain.com:8282");
ws.onopen = function() {
    alert("Conexão bem-sucedida");
    ws.send('tom');
    alert("Enviando uma string para o servidor: tom");
};
ws.onmessage = function(e) {
    alert("Mensagem recebida do servidor: " + e.data);
};
```

**Observações:**

1. Se for necessário utilizar a porta 443, utilize a primeira abordagem com o Nginx/Apache como proxy wss.
2. A porta wss só pode ser acessada pelo protocolo wss; o protocolo ws não pode acessar a porta wss.
3. Os certificados geralmente estão vinculados a um nome de domínio; portanto, ao testar, os clientes devem usar um nome de domínio para se conectar, não um IP.
4. Se houver problemas de acesso, verifique o firewall do servidor.
5. Este método requer a versão do PHP >= 5.6, pois o mini-programa WeChat requer o tls1.2, e as versões do PHP inferiores a 5.6 não oferecem suporte ao tls1.2.

Artigos relacionados:  
[*Obter o IP Real do Cliente via Proxy*](get-real-ip-from-proxy.md)  
[*Referência de Opções de Contexto SSL no Workerman*](https://php.net/manual/zh/context.ssl.php)
