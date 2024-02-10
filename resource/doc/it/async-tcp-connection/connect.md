# Metodo di connessione
```php
void AsyncTcpConnection::connect()
```
Esegue l'operazione di connessione asincrona. Questo metodo restituirà immediatamente.

Nota: se è necessario impostare il callback onError per la connessione asincrona, dovrebbe essere impostato prima di eseguire la connessione, altrimenti il callback onError potrebbe non essere attivato. Ad esempio, nel seguente esempio il callback onError potrebbe non essere attivato e non catturare l'evento di fallimento della connessione asincrona.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// onError non è ancora stato impostato al momento della connessione
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### Parametri
Nessun parametro

### Valore restituito
Nessun valore restituito

### Esempio di proxy Mysql

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Indirizzo reale di mysql, supponendo che sia sulla porta 3306 di questo computer
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// Proxy in ascolto sulla porta locale 4406
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // Stabilisce in modo asincrono una connessione al server mysql effettivo
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // Quando la connessione a mysql invia dati, li inoltra alla connessione del cliente corrispondente
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer) use ($connection)
    {
        $connection->send($buffer);
    };
    // Quando la connessione a mysql viene chiusa, chiude la connessione proxy corrispondente al cliente
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // Se si verifica un errore nella connessione a mysql, chiude la connessione proxy corrispondente al cliente
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // Esegue la connessione asincrona
    $connection_to_mysql->connect();

    // Quando il cliente invia dati, li inoltra alla connessione mysql corrispondente
    $connection->onMessage = function(TcpConnection $connection, $buffer) use ($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // Quando la connessione del cliente viene chiusa, chiude la connessione mysql corrispondente
    $connection->onClose = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // Se si verifica un errore nella connessione del cliente, chiude la connessione mysql corrispondente
    $connection->onError = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// Avvia il worker
Worker::runAll();
```

**Test**

```sh
mysql -uroot -P4406 -h127.0.0.1 -p

Benvenuto al monitor MySQL. I comandi terminano con ; o \g.
L'id della connessione a MySQL è 25004
Versione del server: 5.5.31-1~dotdeb.0 (Debian)

Copyright (c) 2000, 2013, Oracle e/o dalle sue affiliates. Tutti i diritti riservati.

Oracle è un marchio registrato di Oracle Corporation e/o delle sue affiliates.
Altri nomi potrebbero essere marchi dei rispettivi proprietari.

Digita 'help;' o '\h' per ottenere aiuto. Digita '\c' per cancellare l'affermazione di input corrente.

mysql>
```
