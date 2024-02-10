# Text-Protokoll
Workerman definiert ein Textprotokoll namens "text", das das Format "Datapaket + Zeilenumbruch" hat, das heißt, am Ende jedes Datapakets wird ein Zeilenumbruch hinzugefügt, um das Paket zu beenden.

Beispielweise entsprechen die folgenden buffer1- und buffer2-Zeichenfolgen dem Textprotokoll:

```php
// Text mit einem Zeilenumbruch
$buffer1 = 'abcdefghijklmn
';
// In PHP steht \n in doppelten Anführungszeichen für einen Zeilenumbruch, z.B. "\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// Stellt eine Socket-Verbindung zum Server her
$client = stream_socket_client('tcp://127.0.0.1:5678');
// Sendet Daten im Textprotokoll mit buffer1
fwrite($client, $buffer1);
// Sendet Daten im Textprotokoll mit buffer2
fwrite($client, $buffer2);
```

Das Textprotokoll ist sehr einfach und benutzerfreundlich. Wenn Entwickler ein eigenes Protokoll benötigen, z.B. für die Datenübertragung mit einer mobilen App oder die Kommunikation mit Hardware usw., können sie das Textprotokoll in Betracht ziehen, da die Entwicklung und Fehlerbehebung sehr einfach sind.

**Textprotokoll-Debugging**

Das Textprotokoll kann mit einem Telnet-Client debuggt werden, wie im folgenden Beispiel:

Erstellen Sie eine Datei namens test.php

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

Führen Sie ```php test.php start``` aus, um das Folgende anzuzeigen:

```bash
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

Öffnen Sie ein neues Terminal, und führen Sie einen Telnet-Test durch (empfohlen für das telnet-Tool unter Linux).

Angenommen, es handelt sich um einen lokalen Test,
führen Sie im Terminal telnet 127.0.0.1 5678 aus
Geben Sie dann hi ein und drücken Sie die Eingabetaste
Sie werden die Daten "hello world\n" erhalten

```bash
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
