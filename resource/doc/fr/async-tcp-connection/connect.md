# Méthode de connexion

```php
void AsyncTcpConnection::connect()
```

Effectue une opération de connexion asynchrone. Cette méthode renvoie immédiatement.

Remarque : Si vous avez besoin de définir un rappel onError pour la connexion asynchrone, vous devriez le faire avant d'exécuter la méthode connect, sinon le rappel onError pourrait ne pas être déclenché. Par exemple, dans l'exemple ci-dessous, le rappel onError pourrait ne pas être déclenché, ce qui signifie que l'événement de connexion asynchrone échouée ne pourrait pas être capturé.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// Le rappel onError n'est pas défini lorsque la connexion est établie
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### Paramètres
Aucun paramètre

### Valeur de retour
Aucune valeur de retour

### Exemple : Proxy Mysql

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Adresse réelle de MySQL, supposons que c'est le port 3306 de la machine locale
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// Écouteur de proxy sur le port local 4406
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // Établir de manière asynchrone une connexion au serveur MySQL réel
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // Lorsque la connexion MySQL envoie des données, elles sont transmises à la connexion client correspondante
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer) use ($connection)
    {
        $connection->send($buffer);
    };
    // Lorsque la connexion MySQL est fermée, fermer la connexion proxy correspondante au client
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // En cas d'erreur de connexion MySQL, fermer la connexion proxy correspondante au client
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // Exécuter la connexion asynchrone
    $connection_to_mysql->connect();

    // Lorsque le client envoie des données, elles sont transmises à la connexion MySQL correspondante
    $connection->onMessage = function(TcpConnection $connection, $buffer) use ($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // Lorsque la connexion client est fermée, fermer la connexion MySQL correspondante
    $connection->onClose = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // En cas d'erreur de connexion client, fermer la connexion MySQL correspondante
    $connection->onError = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
};

// Exécuter le worker
Worker::runAll();
```

 **Test**

```bash
mysql -uroot -P4406 -h127.0.0.1 -p
```
