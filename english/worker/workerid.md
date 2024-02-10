# id

Requirement: (workerman >= 3.2.1)

## Description:
```php
int Worker::$id
```

The id number of the current worker process, ranging from ```0``` to ```$worker->count-1```.

This property is very useful for distinguishing worker processes. For example, if one worker instance has multiple processes and the developer only wants to set a timer in one of the processes, they can do so by identifying the process number by id. For example, setting the timer only in the process with id number 0 of the worker instance (see example).

## Note:

The id number value remains unchanged after process restart.

The distribution of process id numbers is based on each worker instance. Each worker instance starts numbering its processes from 0, so there may be duplicate process id numbers between worker instances, but the process id numbers within a worker instance will not be duplicated. For example:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Worker instance 1 has 4 processes, and the process id numbers will be 0, 1, 2, 3 respectively
$worker1 = new Worker('tcp://0.0.0.0:8585');
// Set to start 4 processes
$worker1->count = 4;
// Print the current process id number ($worker1->id) after each process starts
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// Worker instance 2 has 2 processes, and the process id numbers will be 0, 1 respectively
$worker2 = new Worker('tcp://0.0.0.0:8686');
// Set to start 2 processes
$worker2->count = 2;
// Print the current process id number ($worker2->id) after each process starts
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// Run workers
Worker::runAll();
```
Output similar to:
```
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

Note: In Windows systems, due to the lack of support for setting the number of processes (count), there is only one process with id number 0. In Windows systems, it is not possible to initialize two Worker listeners for the same file, so this example cannot run on Windows systems.

## Example
One worker instance has 4 processes, and a timer is only set in the process with id number 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // Set the timer only in the process with id number 0, the other processes with numbers 1, 2, 3 do not set the timer
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 worker processes, timer set only in process with id number 0\n";
        });
    }
};
// Run worker
Worker::runAll();
```
