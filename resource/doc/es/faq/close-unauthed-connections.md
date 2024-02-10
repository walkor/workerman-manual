```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // Añadir temporalmente una propiedad auth_timer_id al objeto $connection para almacenar el id del temporizador
    // Cerrar la conexión después de 30 segundos, si el cliente no ha enviado datos para la autenticación dentro de ese tiempo
    $connection->auth_timer_id = Timer::add(30, function() use ($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
        case 'login':
            ...omitir
            // Autenticación exitosa, eliminar el temporizador para evitar el cierre de la conexión
            Timer::del($connection->auth_timer_id);
            break;
        ...omitir
    }
    ...omitir
}
```
