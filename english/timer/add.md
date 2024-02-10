# add
```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
Execute a function or a class method at regular intervals.

Note: The timer runs in the current process, and Workerman does not create new processes or threads to run the timer.

### Parameters
``` time_interval ```

How long between each execution, in seconds, supports decimals and can be accurate to 0.001, i.e., accurate to the millisecond level. 

``` callback ```

The callback function. ```Note: If the callback function is a class method, the method must be public```

``` args ```

The parameters for the callback function, must be an array, with each array element being a parameter value.

``` persistent ```

Whether it is persistent. If you want the timer to only execute once, pass false (a task that only executes once will be automatically destroyed after execution, there is no need to call ```Timer::del()```). The default is true, which means it executes continuously.

### Return
Returns an integer representing the timer's timerid, which can be used to destroy the timer by calling ```Timer::del($timerid)```.

### Example

#### 1. Timing function is an anonymous function (closure)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Start the specified number of processes to run the timed task. Pay attention to whether there will be concurrent issues in multiple processes for the business logic
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Execute every 2.5 seconds
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// Run the worker
Worker::runAll();
```

#### 2. Set the timer only in the specified process

Suppose a worker instance has 4 processes, the timer is only set in process number 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function(Worker $worker)
{
    // Set the timer only in process with id number 0, no timer will be set in process 1, 2, and 3
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "There are 4 worker processes, and the timer is only set in the process with id 0\n";
        });
    }
};
// Run the worker
Worker::runAll();
```

#### 3. Timing function is an anonymous function, using closure to pass parameters
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
// Set a timer for the corresponding connection when the connection is established
$ws_worker->onConnect = function(TcpConnection $connection)
{
    // Execute every 10 seconds
    $time_interval = 10;
    $connect_time = time();
    // Temporarily add a timer_id attribute to the connection object to save the timer id
    $connection->timer_id = Timer::add($time_interval, function()use($connection, $connect_time)
    {
         $connection->send($connect_time);
    });
};
// When the connection is closed, delete the corresponding connection's timer
$ws_worker->onClose = function(TcpConnection $connection)
{
    // Delete the timer
    Timer::del($connection->timer_id);
};

// Run the worker
Worker::runAll();
```

#### 4. Timing function is an anonymous function, using the timer interface to pass parameters
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
// Set a timer for the corresponding connection when the connection is established
$ws_worker->onConnect = function(TcpConnection $connection)
{
    // Execute every 10 seconds
    $time_interval = 10;
    $connect_time = time();
    // Temporarily add a timer_id attribute to the connection object to save the timer id
    $connection->timer_id = Timer::add($time_interval, function($connection, $connect_time)
    {
         $connection->send($connect_time);
    }, array($connection, $connect_time));
};
// When the connection is closed, delete the corresponding connection's timer
$ws_worker->onClose = function(TcpConnection $connection)
{
    // Delete the timer
    Timer::del($connection->timer_id);
};

// Run the worker
Worker::runAll();
```

#### 5. Timing function is a normal function
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

// Normal function
function send_mail($to, $content)
{
    echo "send mail ...\n";
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // Execute the task of sending emails after 10 seconds. Pass false as the last parameter to indicate running only once
    Timer::add(10, 'send_mail', array($to, $content), false);
};

// Run the worker
Worker::runAll();
```

#### 6. Timing function is a class method
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // Note that the callback function property must be public
    public function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // Send an email after 10 seconds
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    Timer::add(10, array($mail, 'send'), array($to, $content), false);
};

// Run the worker
Worker::runAll();
```

#### 7. Timing function is a class method (using timer within the class)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // Note that the callback function property must be public
    public function send($to, $content)
    {
        echo "send mail ...\n";
    }

    public function sendLater($to, $content)
    {
        // If the callback method belongs to the current class, then the first element in the callback array must be $this
        Timer::add(10, array($this, 'send'), array($to, $content), false);
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Send an email after 10 seconds
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    $mail->sendLater($to, $content);
};

// Run the worker
Worker::runAll();
```

#### 8. Timing function is a static method of a class
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // Note that this is a static method, and the callback function property must be public
    public static function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Send an email after 10 seconds
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // Schedule a call to the class's static method
    Timer::add(10, array('Mail', 'send'), array($to, $content), false);
};

// Run the worker
Worker::runAll();
```

#### 9. Timing function is a static method of a class (with namespace)
```php
namespace Task;

use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // Note that this is a static method, and the callback function property must be public
    public static function send($to, $content)
    {
        echo "send mail ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Send an email after 10 seconds
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    // Schedule a call to the static method of the class with namespace
    Timer::add(10, array('\Task\Mail', 'send'), array($to, $content), false);
};

// Run the worker
Worker::runAll();
```

#### 10. Destroy the current timer within the timer (using closure to pass $timer_id)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Counter
    $count = 1;
    // To pass $timer_id correctly into the callback function, & must be placed before $timer_id
    $timer_id = Timer::add(1, function()use(&$timer_id, &$count)
    {
        echo "Timer run $count\n";
        // Destroy the current timer after running 10 times
        if($count++ >= 10)
        {
            echo "Timer::del($timer_id)\n";
            Timer::del($timer_id);
        }
    });
};

// Run the worker
Worker::runAll();
```

#### 11. Destroy the current timer within the timer (pass $timer_id as a parameter)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    public function send($to, $content, $timer_id)
    {
        // Temporarily add a count attribute to the current object to record the number of times the timer runs
        $this->count = empty($this->count) ? 1 : $this->count;
        // Destroy the current timer after running 10 times
        echo "send mail {$this->count}...\n";
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
    // To pass $timer_id correctly into the callback function, & must be placed before $timer_id
    $timer_id = Timer::add(1, array($mail, 'send'), array('to', 'content', &$timer_id));
};

// Run the worker
Worker::runAll();
```
