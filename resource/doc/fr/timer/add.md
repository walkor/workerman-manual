```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
Exécute périodiquement une fonction ou une méthode de classe.

Remarque : Les temporisateurs s'exécutent dans le processus actuel, workerman ne crée pas de nouveaux processus ou threads pour exécuter les temporisateurs.

### Paramètres
``` time_interval ```

Durée en secondes entre chaque exécution, supporte les décimales, peut être précis à 0,001, c'est-à-dire à un niveau milliseconde.

``` callback ```

Fonction de rappel. Remarque : Si la fonction de rappel est une méthode de classe, la méthode doit être publique.

``` args ```

Arguments de la fonction de rappel, doit être un tableau dont les éléments sont les valeurs des arguments.

``` persistent ```

Détermine si le temporisateur est persistant. Si vous voulez exécuter le temporisateur une seule fois, transmettez false (la tâche qui s'exécute une seule fois est automatiquement détruite après l'exécution, pas besoin d'appeler ```Timer::del()```). Par défaut, c'est vrai, c'est-à-dire qu'il s'exécute périodiquement.

### Valeur de retour
Retourne un entier représentant l'identifiant du temporisateur, qui peut être détruit en appelant ```Timer::del($timerid)```.

### Exemple

#### 1. Fonction anonyme comme temporisateur (fermeture)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "tâche en cours\n";
    });
};

Worker::runAll();
```

### 2. Définir le temporisateur uniquement dans un processus spécifique

Une instance de worker a 4 processus, définir le temporisateur uniquement dans le processus avec l'identifiant 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function(Worker $worker)
{
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 processus worker, seul le processus 0 a un temporisateur\n";
        });
    }
};

Worker::runAll();
```

### 3. Définir la fonction de temporisation comme fonction anonyme en passant des paramètres via une fermeture
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
$ws_worker->onConnect = function(TcpConnection $connection)
{
    $time_interval = 10;
    $connect_time = time();
    $connection->timer_id = Timer::add($time_interval, function()use($connection, $connect_time)
    {
         $connection->send($connect_time);
    });
};

$ws_worker->onClose = function(TcpConnection $connection)
{
    Timer::del($connection->timer_id);
};

Worker::runAll();
```

### 4. Définir la fonction de temporisation comme fonction anonyme en passant des paramètres via l'interface de temporisation
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
$ws_worker->onConnect = function(TcpConnection $connection)
{
    $time_interval = 10;
    $connect_time = time();
    $connection->timer_id = Timer::add($time_interval, function($connection, $connect_time)
    {
         $connection->send($connect_time);
    }, array($connection, $connect_time));
};

$ws_worker->onClose = function(TcpConnection $connection)
{
    Timer::del($connection->timer_id);
};

Worker::runAll();
```

### 5. Définir la fonction de temporisation comme fonction normale
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

function send_mail($to, $content)
{
    echo "envoyer un mail ...\n";
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $to = 'workerman@workerman.net';
    $content = 'salut workerman';
    Timer::add(10, 'send_mail', array($to, $content), false);
};

Worker::runAll();
```

### 6. Définir la fonction de temporisation comme méthode de classe
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    public function send($to, $content)
    {
        echo "envoyer un mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'salut workerman';
    Timer::add(10, array($mail, 'send'), array($to, $content), false);
};

Worker::runAll();
```

### 7. Définir la fonction de temporisation comme méthode de classe (utilisation du temporisateur dans la classe)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    public function send($to, $content)
    {
        echo "envoyer un mail ...\n";
    }

    public function sendLater($to, $content)
    {
        Timer::add(10, array($this, 'send'), array($to, $content), false);
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'salut workerman';
    $mail->sendLater($to, $content);
};

Worker::runAll();
```

### 8. Définir la fonction de temporisation comme méthode statique de classe
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    public static function send($to, $content)
    {
        echo "envoyer un mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $to = 'workerman@workerman.net';
    $content = 'salut workerman';
    Timer::add(10, array('Mail', 'send'), array($to, $content), false);
};

Worker::runAll();
```

### 9. Définir la fonction de temporisation comme méthode statique de classe (avec namespace)
```php
namespace Task;

use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    public static function send($to, $content)
    {
        echo "envoyer un mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $to = 'workerman@workerman.net';
    $content = 'salut workerman';
    Timer::add(10, array('\Task\Mail', 'send'), array($to, $content), false);
};

Worker::runAll();
```

### 10. Détruire le temporisateur en cours dans le temporisateur (en utilisant une fermeture pour passer $timer_id)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $count = 1;
    $timer_id = Timer::add(1, function()use(&$timer_id, &$count)
    {
        echo "Temporisation en cours $count\n";
        if($count++ >= 10)
        {
            echo "Timer::del($timer_id)\n";
            Timer::del($timer_id);
        }
    });
};

Worker::runAll();
```

### 11. Détruire le temporisateur en cours dans le temporisateur (passer de $timer_id via les paramètres)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    public function send($to, $content, $timer_id)
    {
        $this->count = empty($this->count) ? 1 : $this->count;
        echo "envoyer un mail {$this->count}...\n";
        if($this->count++ >= 10)
        {
            echo "Timer::del($timer_id)\n";
            Timer::del($timer_id);
        }
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $mail = new Mail();
    $timer_id = Timer::add(1, array($mail, 'send'), array('to', 'content', &$timer_id));
};

Worker::runAll();
```
