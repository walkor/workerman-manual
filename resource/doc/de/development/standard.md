# Entwicklungsstandards

## Anwendungsverzeichnis

Das Anwendungsverzeichnis kann an beliebiger Stelle platziert werden.

## Einstiegsdatei

Wie bei PHP-Anwendungen unter nginx+PHP-FPM benötigt auch die Anwendung in WorkerMan eine Einstiegsdatei. Der Name der Einstiegsdatei ist nicht festgelegt, und diese Datei wird im PHP Cli-Modus ausgeführt.

Die Einstiegsdatei enthält den Code zum Erstellen von Überwachungsprozessen, wie zum Beispiel das folgende Code-Snippet, das auf Worker-Entwicklung basiert.

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Erstellen eines Workers, der den Port 2345 überwacht und das HTTP-Protokoll verwendet
$http_worker = new Worker("http://0.0.0.0:2345");

// Starten von 4 Prozessen, um Dienste bereitzustellen
$http_worker->count = 4;

// Antwort "hello world" an den Browser, wenn Daten vom Browser empfangen werden
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // "hello world" an den Browser senden
    $connection->send('hello world');
};

Worker::runAll();

```


## Code-Standards in WorkerMan

1. Klassen werden in CamelCase mit großem Anfangsbuchstaben benannt, und der Dateiname der Klasse muss mit dem internen Klassennamen übereinstimmen, um eine automatische Ladung zu ermöglichen. Beispiel:
```php
class UserInfo
{
...
```

2. Verwenden Sie Namensräume, die dem Verzeichnispfad entsprechen, und verwenden Sie das Wurzelverzeichnis des Projekts des Entwicklers als Grundlage.

Beispiel für ein Projekt namens MyApp/: Die Klasse MyApp/MyClass.php benötigt keinen Namensraum, da sie sich im Projektstammverzeichnis befindet. Die Klasse MyApp/Protocols/MyProtocol.php hingegen befindet sich im Verzeichnis Protocols des MyApp-Projekts, daher muss der Namensraum hinzugefügt werden: ```namespace Protocols;``` wie folgt:
```php
namespace Protocols;
class MyProtocol
{
....
```

3. Allgemeine Funktionen und Variablennamen werden in Kleinschreibung mit Unterstrichen verwendet. Zum Beispiel:
```php
$connection_list = array();
function get_connection_list()
{
....
```

4. Klassenmember und -methoden werden in CamelCase mit kleinem Anfangsbuchstaben verwendet. Zum Beispiel:
```php
public $connectionList;
public function getConnectionList();
```

5. Funktions- und Klassenparameter werden in Kleinschreibung mit Unterstrichen verwendet. Zum Beispiel:
```php
function get_connection_list($one_param, $two_param)
{
....
```
