# Criptografia de transporte-ssl/tls

**Pergunta:**

Como garantir a segurança na comunicação com o Workerman?

**Resposta:**

Uma maneira conveniente é adicionar uma camada de criptografia [SSL](https://baike.baidu.com/item/ssl) sobre o protocolo de comunicação. Por exemplo, os protocolos wss e [https](https://baike.baidu.com/item/https) são baseados na transmissão criptografada por [SSL](https://baike.baidu.com/item/ssl), o que é muito seguro. O próprio Workerman suporta [SSL](https://baike.baidu.com/item/ssl) (```requer Workerman>=3.3.7```), e apenas precisa configurar as propriedades para ativar o SSL.

Os desenvolvedores também podem implementar um mecanismo de criptografia personalizado com base em algum algoritmo de criptografia.

## Método para ativar o ssl no Workerman:

**Pré-requisitos:**

1. Versão do Workerman igual ou superior a 3.3.7

2. Extensão openssl do PHP instalada

3. Certificado já obtido (arquivos pem/crt e arquivo de chave) e colocados em /etc/nginx/conf.d/ssl

**Código:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// É recomendável usar um certificado obtido
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // pode ser um arquivo crt também
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // deve ser ativado se o certificado for autoassinado
    )
);
// Aqui é definido o protocolo websocket, mas também pode ser http ou outro protocolo
$worker = new Worker('websocket://0.0.0.0:443', $context);
// Ativando o transporte com SSL
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Ativar o Indicador de Nome do Servidor [SNI (Server Name Indication)](https://baike.baidu.com/item/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA) no Workerman
Permite vincular vários certificados ao mesmo IP e porta.

**Fundir arquivos.pem e .key do certificado:**

Junte o conteúdo de cada arquivo .pem com o respectivo arquivo .key, adicionando o conteúdo do arquivo .key ao final do arquivo .pem (isso pode ser ignorado se a chave privada já estiver no arquivo .pem).

**Atenção para juntar apenas um certificado por vez, não todos em um único arquivo**

Por exemplo, o conteúdo do arquivo.pem após a fusão do *host1.com.pem* provavelmente será semelhante a isto:

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
        'SNI_enabled' => true, // Ativar SNI
        'SNI_server_certs' => [ // Definir vários certificados
            'host1.com' => '/path/host1.com.pem', // Certificado 1
            'host2.com' => '/path/host2.com.pem', // Certificado 2
        ],
        'local_cert' => '/path/default.com.pem', // Certificado padrão
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
