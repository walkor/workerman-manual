# Norme di sviluppo

## Directory dell'applicazione

La directory dell'applicazione può essere posizionata ovunque.

## File di ingresso

Come le applicazioni PHP con nginx+PHP-FPM, le applicazioni in Workerman richiedono anche un file di ingresso, che non ha un nome specifico e viene eseguito come script PHP CLI.

Il file di ingresso contiene il codice relativo alla creazione del processo di ascolto, ad esempio il seguente frammento di codice basato su Worker:

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Crea un Worker in ascolto sulla porta 2345, utilizzando il protocollo http
$http_worker = new Worker("http://0.0.0.0:2345");

// Avvia 4 processi per fornire il servizio
$http_worker->count = 4;

// Quando riceve dati inviati dal browser, risponde con "hello world"
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Invia "hello world" al browser
    $connection->send('hello world');
};

Worker::runAll();

```

## Norme del codice in Workerman

1. I nomi delle classi seguono la convenzione CamelCase iniziale maiuscola, e il nome del file della classe deve corrispondere al nome interno della classe, per consentire il caricamento automatico. Ad esempio:
```php
class UserInfo
{
...
```

2. Utilizzare gli spazi dei nomi, con il nome dello spazio dei nomi che corrisponde al percorso della directory, basato sulla directory radice del progetto dello sviluppatore.

Ad esempio, se il progetto è MyApp/, il file di classe MyApp/MyClass.php, non avendo una sottodirectory, può omettere il nome dello spazio dei nomi. Ma il file di classe MyApp/Protocols/MyProtocol.php, trovandosi nella sottodirectory Protocols di MyApp, deve includere il nome dello spazio dei nomi `namespace Protocols;`, come segue:
```php
namespace Protocols;
class MyProtocol
{
....
```

3. I nomi delle variabili e delle funzioni comuni seguono la convenzione di intestazione minuscola con underscore. Ad esempio:
```php
$connection_list = array();
function get_connection_list()
{
....
```

4. I membri delle classi e i metodi delle classi seguono la convenzione di intestazione minuscola con camelCase. Ad esempio:
```php
public $connectionList;
public function getConnectionList();
```

5. I parametri delle funzioni e delle classi seguono la convenzione di intestazione minuscola con underscore. Ad esempio:
```php
function get_connection_list($one_param, $tow_param)
{
....

```
