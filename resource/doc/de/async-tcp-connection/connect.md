# Die connect-Methode
```php
void AsyncTcpConnection::connect()
```
Führt eine asynchrone Verbindung durch. Diese Methode gibt sofort zurück.

Hinweis: Wenn das onError-Rückruf für die asynchrone Verbindung festgelegt werden soll, sollte dies vor dem Aufruf von connect erfolgen. Andernfalls wird der onError-Rückruf möglicherweise nicht ausgelöst. Im folgenden Beispiel wird der onError-Rückruf möglicherweise nicht ausgelöst, da er nicht festgelegt wurde, bevor die Verbindung hergestellt wurde.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// Der onError-Rückruf wurde noch nicht festgelegt, als die Verbindung hergestellt wurde
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### Parameter
Keine Parameter

### Rückgabewert
Kein Rückgabewert

### Beispiel für MySQL-Proxy

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Die tatsächliche MySQL-Adresse, angenommen, sie befindet sich auf Port 3306 des lokalen Computers
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// Der Proxy lauscht auf Port 4406 des lokalen Computers
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // Asynchrone Verbindung zum tatsächlichen MySQL-Server herstellen
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // Wenn Daten von der MySQL-Verbindung empfangen werden, leite sie an die entsprechende Client-Verbindung weiter
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer) use($connection)
    {
        $connection->send($buffer);
    };
    // Wenn die MySQL-Verbindung geschlossen wird, wird auch die entsprechende Verbindung zum Client geschlossen
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql) use($connection)
    {
        $connection->close();
    };
    // Wenn bei der MySQL-Verbindung ein Fehler auftritt, wird auch die entsprechende Verbindung zum Client geschlossen
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql) use($connection)
    {
        $connection->close();
    };
    // Asynchrone Verbindung herstellen
    $connection_to_mysql->connect();

    // Wenn Daten vom Client empfangen werden, leite sie an die entsprechende MySQL-Verbindung weiter
    $connection->onMessage = function(TcpConnection $connection, $buffer) use($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // Wenn die Client-Verbindung getrennt wird, wird auch die entsprechende MySQL-Verbindung geschlossen
    $connection->onClose = function(TcpConnection $connection) use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // Wenn bei der Client-Verbindung ein Fehler auftritt, wird auch die entsprechende MySQL-Verbindung geschlossen
    $connection->onError = function(TcpConnection $connection) use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
};
// Worker ausführen
Worker::runAll();
```

**Test**

```bash
mysql -uroot -P4406 -h127.0.0.1 -p

Willkommen zur MySQL-Monitorschnittstelle.  Befehle enden mit ; oder \g.
Ihre MySQL-Server-Version ist 5.5.31-1~dotdeb.0 (Debian).

Urheberrecht © 2000, 2013, Oracle und/oder Tochtergesellschaften. Alle Rechte vorbehalten.

Oracle ist ein eingetragenes Warenzeichen der Oracle Corporation und/oder ihrer
Tochtergesellschaften. Andere Namen können Warenzeichen ihrer jeweiligen
Eigentümer sein.

Geben Sie 'help;' oder '\h' für Hilfe ein. Geben Sie '\c' ein, um die aktuelle Eingabeanweisung zu löschen.

mysql>
```
