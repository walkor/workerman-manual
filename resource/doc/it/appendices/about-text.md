# Protocollo text
Workerman definisce un protocollo di testo chiamato "text", il formato del protocollo è ```pacchetto di dati + carattere di nuova riga```, cioè aggiungere un carattere di nuova riga alla fine di ogni pacchetto di dati per indicarne la fine.

Ad esempio, le stringhe buffer1 e buffer2 seguenti sono conformi al protocollo di testo:

```php
// Testo più un ritorno a capo
$buffer1 = 'abcdefghijklmn
';
// In PHP, "\n" tra virgolette doppie rappresenta un carattere di nuova riga, ad esempio "\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// Connettersi al server tramite socket
$client = stream_socket_client('tcp://127.0.0.1:5678');
// Invia i dati del buffer1 utilizzando il protocollo di testo
fwrite($client, $buffer1);
// Invia i dati del buffer2 utilizzando il protocollo di testo
fwrite($client, $buffer2);
```

Il protocollo di testo è molto semplice da usare. Se gli sviluppatori hanno bisogno di un protocollo personalizzato, ad esempio per trasferire dati con un'applicazione mobile o comunicare con l'hardware, possono considerare l'utilizzo del protocollo di testo, che è molto comodo sia per lo sviluppo che per il debug.

**Debug del protocollo di testo**

> Il protocollo di testo può essere debuggato utilizzando un client telnet, ad esempio:

Creare un file test.php

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

Eseguire ```php test.php start``` mostrerà quanto segue

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

Aprire una nuova console e provare con telnet (si consiglia di utilizzare telnet di un sistema Linux)

Supponendo che il test sia locale,
Eseguire telnet 127.0.0.1 5678
Quindi inserire "hi" e premere Invio
Si riceverà "hello world\n" come risposta.

``` 
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
