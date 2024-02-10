# Criar um serviço https

**Pergunta:**

Como posso criar um serviço [https](https://baike.baidu.com/item/https) com o Workerman, para que os clientes possam se comunicar utilizando o protocolo [https](https://baike.baidu.com/item/https).

**Resposta:**

O protocolo [https](https://baike.baidu.com/item/https) é, na verdade, uma combinação do [http](https://baike.baidu.com/item/http) com [SSL](https://baike.baidu.com/item/ssl), ou seja, é a camada [SSL](https://baike.baidu.com/item/ssl) sobre o protocolo [http](https://baike.baidu.com/item/http). O Workerman suporta o protocolo [http](https://baike.baidu.com/item/http) e também suporta o [SSL](https://baike.baidu.com/item/ssl) (```requer Workerman versão >= 3.3.7```), portanto, é possível suportar o protocolo [https](https://baike.baidu.com/item/https) simplesmente abrindo o [SSL](https://baike.baidu.com/item/ssl) sobre o protocolo [http](https://baike.baidu.com/item/http).

Há duas maneiras comuns de fazer o Workerman suportar [https](https://baike.baidu.com/item/https), uma delas é iniciar o [SSL](https://baike.baidu.com/item/ssl) diretamente no Workerman e a outra é usar o nginx como um proxy [SSL](https://baike.baidu.com/item/ssl). Apenas uma das duas opções pode ser escolhida, não é possível configurá-las simultaneamente.

## Iniciando o [SSL](https://baike.baidu.com/item/ssl) no Workerman

**Pré-requisitos:**

1. Workerman versão >= 3.3.7
2. Extensão openssl instalada no PHP
3. Certificado já foi adquirido (arquivos pem/crt e chave) e está localizado em /etc/nginx/conf.d/ssl

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// É recomendável utilizar um certificado adquirido
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // Pode ser também um arquivo crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Se for um certificado autoassinado, é necessário ativar esta opção
    )
);
// Aqui está sendo configurado o protocolo http
$worker = new Worker('http://0.0.0.0:443', $context);
// Configurando o transporte para abrir o [SSL](https://baike.baidu.com/item/ssl), transformando em http+SSL, ou seja, https
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Com o código acima do Workerman, um serviço https é criado e os clientes poderão realizar a comunicação de forma segura e criptografada através do protocolo https.

**Teste:**

Digite "https://domínio:443" na barra de endereços do navegador para acessar.

**Observações:**

1. A porta https deve ser acessada apenas com o protocolo https; o protocolo http não funcionará.
2. O certificado geralmente está vinculado a um domínio. Portanto, ao realizar o teste, utilize o domínio em vez do IP.
3. Se o acesso via https não estiver funcionando, verifique o firewall do servidor.

## Utilizando o nginx como proxy [SSL](https://baike.baidu.com/item/ssl)

Além de usar o [SSL](https://baike.baidu.com/item/ssl) do Workerman, também é possível usar o nginx como um proxy [SSL](https://baike.baidu.com/item/ssl) para implementar o https.

> **Observação**
> A utilização do nginx como proxy [SSL](https://baike.baidu.com/item/ssl) e a configuração do [SSL](https://baike.baidu.com/item/ssl) no Workerman são duas opções exclusivas, não podem ser ativadas simultaneamente.

O princípio e o fluxo de comunicação são os seguintes:

1. O cliente inicia a conexão [https](https://baike.baidu.com/item/https) com o nginx.
2. O nginx converte os dados do protocolo [https](https://baike.baidu.com/item/https) para o protocolo [http](https://baike.baidu.com/item/http) e encaminha para a porta http do Workerman.
3. O Workerman recebe os dados e realiza o processamento lógico, retornando os dados no protocolo [http](https://baike.baidu.com/item/http) para o nginx.
4. O nginx converte novamente os dados no protocolo [http](https://baike.baidu.com/item/http) para o protocolo [https](https://baike.baidu.com/item/https) e encaminha para o cliente.

### Referência de configuração do nginx

**Pré-requisitos e preparação:**

1. Supondo que o Workerman está ouvindo na porta 8181 (protocolo http).
2. Certificado já foi adquirido (arquivos pem/crt e chave) e está localizado em /etc/nginx/conf.d/ssl.
3. Pretende-se usar o nginx para abrir a porta 443 e fornecer serviços de proxy wss (a porta pode ser modificada conforme necessário).

**A configuração do nginx é semelhante ao seguinte exemplo:**

```nginx
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}

server {
  listen 443;
  server_name nomedosite.com;
  access_log off;

  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location / {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
