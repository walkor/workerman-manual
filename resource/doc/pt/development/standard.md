# Normas de Desenvolvimento

## Diretório da Aplicação

O diretório da aplicação pode ser colocado em qualquer local.

## Arquivo de Entrada

Assim como nas aplicações PHP sob nginx+PHP-FPM, as aplicações do Workerman também necessitam de um arquivo de entrada, cujo nome não é obrigatório, e esse arquivo de entrada é executado como PHP Cli.

O arquivo de entrada contém o código relacionado à criação do processo de escuta, como o seguinte trecho de código baseado no desenvolvimento do Worker:

test.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Cria um Worker para escutar a porta 2345, utilizando o protocolo http
$http_worker = new Worker("http://0.0.0.0:2345");

// Inicia 4 processos para fornecer serviços externos
$http_worker->count = 4;

// Ao receber dados enviados pelo navegador, responde "hello world" para o navegador
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Envia "hello world" para o navegador
    $connection->send('hello world');
};

Worker::runAll();

```

## Normas de Código em Workerman

1. Os nomes das classes seguem a convenção camel case com a primeira letra em maiúsculo, e o nome do arquivo da classe deve ser o mesmo que o nome interno da classe, para permitir o carregamento automático. Por exemplo:

```php
class UserInfo
{
...
```

2. Utilize namespaces, onde o nome do namespace corresponde ao caminho do diretório, com base no diretório raiz do projeto.

Por exemplo, em um projeto MyApp/, o arquivo da classe MyClass.php está no diretório raiz do projeto, então o namespace é omitido. Já o arquivo da classe MyProtocol.php está no diretório Protocols/ do projeto MyApp, por isso o namespace deve ser adicionado como ```namespace Protocols;```, conforme o exemplo a seguir:

```php
namespace Protocols;
class MyProtocol
{
....
```

3. Os nomes de funções e variáveis comuns seguem a convenção de letras minúsculas com sublinhado, por exemplo:

```php
$connection_list = array();
function get_connection_list()
{
....
```

4. Os membros da classe e métodos da classe seguem a convenção de letras minúsculas com a primeira letra em maiúsculo, no formato camel case, por exemplo:

```php
public $connectionList;
public function getConnectionList();
```

5. Os parâmetros de funções e métodos de classes seguem a convenção de letras minúsculas com sublinhado, por exemplo:

```php
function get_connection_list($um_parametro, $outro_parametro)
{
....
```
