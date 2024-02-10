# Protocolo de texto
> Workerman define um protocolo de texto chamado "text", no qual o formato do protocolo é ```pacote de dados + quebra de linha```, ou seja, um caractere de quebra de linha é adicionado ao final de cada pacote de dados para indicar o término do pacote.

Por exemplo, as strings buffer1 e buffer2 a seguir estão de acordo com o protocolo de texto:

```php
// Texto mais uma quebra de linha
$buffer1 = 'abcdefghijklmn
';
// Em PHP, \n dentro de aspas duplas representa um caractere de quebra de linha, por exemplo, "\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// Estabelecer conexão de soquete com o servidor
$client = stream_socket_client('tcp://127.0.0.1:5678');
// Enviar dados buffer1 usando o protocolo de texto
fwrite($client, $buffer1);
// Enviar dados buffer2 usando o protocolo de texto
fwrite($client, $buffer2);
```

O protocolo de texto é muito simples e fácil de usar. Se os desenvolvedores precisarem de um protocolo próprio, como transferir dados para um aplicativo móvel ou se comunicar com hardware, por exemplo, eles podem considerar usar o protocolo de texto, pois é muito conveniente para desenvolvimento e depuração. 

**Depuração do protocolo de texto**

> O protocolo de texto pode ser depurado usando um cliente telnet, como no exemplo a seguir:

Crie um novo arquivo test.php

```php
require_once __DIR__ . '/Workerman/Autoloader.php';
use Workerman\Worker;

$text_worker = new Worker("text://0.0.0.0:5678");

$text_worker->onMessage =  function($connection, $data)
{
    var_dump($data);
    $connection->send("hello world");
};

Worker::runAll();
```

Execute `php test.php start` e será exibido como abaixo:

```
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

Abra um novo terminal e teste usando o telnet (recomendado usar o telnet do sistema Linux)

Supondo que seja um teste local,
No terminal, execute telnet 127.0.0.1 5678
Em seguida, digite hi e pressione Enter
O resultado recebido será hello world\n
```
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world

```
