# Chiudere le connessioni non autenticate

**Domanda:**

Come posso chiudere automaticamente la connessione di un client che non ha inviato dati entro un certo periodo di tempo, ad esempio se non ricevo alcun dato da un client entro 30 secondi, voglio chiudere automaticamente la connessione, allo scopo di garantire che le connessioni non autenticate debbano autenticarsi entro un certo periodo di tempo.

**Risposta:**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // Aggiungere temporaneamente alla proprietà $connection un attributo auth_timer_id per memorizzare l'ID del timer.
    // Chiudere la connessione dopo 30 secondi. Se il client non invia una richiesta di autenticazione entro 30 secondi, il timer scatta e chiude la connessione.
    $connection->auth_timer_id = Timer::add(30, function() use ($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function (TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
        case 'login':
            // Verifica dell'autenticazione riuscita, cancella il timer per evitare la chiusura della connessione
            Timer::del($connection->auth_timer_id);
            break;
    }
}
```
