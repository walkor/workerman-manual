# How Many Processes Should Be Started

## How to Set the Number of Processes
The number of processes is determined by the ```count``` property (Windows system does not support process count setting), for example, in the following code:
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// ## Start 4 processes to provide services externally ##
$http_worker->count = 4;

...
```

## Factors to Consider for Setting the Number of Processes
1. Number of CPU cores
2. Amount of memory
3. Business inclination towards IO-intensive or CPU-intensive

## Principles for Setting the Number of Processes
1. The total memory occupied by each process should be less than the total memory (generally, each business process occupies about 40M of memory).
2. If it is IO-intensive, meaning it involves some **blocking** IO (for example, accessing Mysql, Redis, etc., which are blocking accesses), the number of processes can be set larger, such as configuring it to be 3 times the number of CPU cores. If there are a lot of blocking waits in the business, the number of processes can be increased appropriately, such as 8 times the number of CPU cores or even higher. Note that **non-blocking** IO belongs to CPU-intensive, not IO-intensive.
3. If it is CPU-intensive, meaning there is no **blocking** IO overhead in the business, for example, using asynchronous IO to read network resources, and the process will not be blocked by the business code, the number of processes can be set to be the same as the number of CPU cores.

## Reference Values for Setting the Number of Processes
If the business code tends to be IO-intensive, the number of processes can be set based on the degree of IO intensity, for example, 3-8 times the number of CPU cores.

If the business code tends to be CPU-intensive, the number of processes can be set to the number of CPU cores.

## Note
WorkerMan's IO itself is non-blocking, for example, ```Connection->send``` is non-blocking, which belongs to CPU-intensive operations. If it is not clear which type your business belongs to, setting the number of processes to around 3 times the number of CPU cores is recommended.
Additionally, more processes do not necessarily mean better performance. If too many processes are opened, the overhead of process switching will increase, which will have a certain impact on performance.
