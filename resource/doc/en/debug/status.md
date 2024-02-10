# Viewing Running Status

Running `php start.php status` shows WorkerMan's running status similar to the following:

``` 
----------------------------------------------GLOBAL STATUS----------------------------------------------------
Workerman version: 3.5.13          PHP version: 5.5.9-1ubuntu4.24
start time: 2018-02-03 11:48:20   run 112 days 2 hours   
load average: 0, 0, 0            event-loop: \Workerman\Events\Event
4 workers       11 processes
worker_name        exit_status      exit_count
ChatBusinessWorker 0               0
ChatGateway        0               0
Register           0               0
WebServer          0               0
----------------------------------------------PROCESS STATUS---------------------------------------------------
pid	memory  listening                worker_name        connections send_fail timers  total_request qps    status
18306	2.25M   none                     ChatBusinessWorker 5           0         0       11            0      [idle]
18307	2.25M   none                     ChatBusinessWorker 5           0         0       8             0      [idle]
18308	2.25M   none                     ChatBusinessWorker 5           0         0       3             0      [idle]
18309	2.25M   none                     ChatBusinessWorker 5           0         0       14            0      [idle]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [idle]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [idle]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [idle]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [idle]
18314	1.75M   text://0.0.0.0:1236      Register           8           0         0       8             0      [idle]
18315	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [idle]
18316	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [idle]
----------------------------------------------PROCESS STATUS---------------------------------------------------
Summary	18M     -                        -                  54          0         4       138           0      [Summary]
```
  
## Description   

### GLOBAL STATUS

In this section, we can see:

WorkerMan version `version:3.5.13`

Start time `2018-02-03 11:48:20`, running for `run 112 days 2 hours`

Server load `load average: 0, 0, 0`, representing the system's average load in the last 1, 5, and 15 minutes.

Used IO event library: `event-loop:\Workerman\Events\Event`

`4 workers` (3 types of processes, including ChatGateway, ChatBusinessWorker, Register processes and WebServer process)

`11 processes` (a total of 11 processes)

`worker_name` (worker process name)

`exit_status` (exit status code of worker processes)

`exit_count` (exit count of the status code)

Generally, an `exit_status` of 0 indicates a normal exit. If it has a different value, the process exited unexpectedly, and an error message similar to `WORKER EXIT UNEXPECTED` will be recorded in the file specified in [Worker::logFile](worker/log-file.md).

**Common `exit_status` and their meanings:**

* 0: Represents a normal exit, which can occur after a reload during a smooth restart. It's important to note that calling exit or die in the program will also result in an exit code of 0, and will generate a `WORKER EXIT UNEXPECTED` error message. Workerman does not allow business code to call exit or die statements.
* 9: Indicates that the process was killed by a SIGKILL signal. This exit code mainly occurs during stops and reloads, often due to the sub-process not responding to the main process' reload signal within the specified time (e.g. due to long blocking of MySQL, curl, etc., or business dead loops), and was forced to be killed by the main process using a SIGKILL signal. Note that using the kill command in the Linux command line to send a SIGKILL signal to a sub-process will also result in this exit code.
* 11: Indicates a coredump of PHP, which generally occurs due to unstable extensions. Comment out the corresponding extensions in php.ini. In some cases, it may be a PHP bug, in which case the solution is to upgrade PHP.
* 65280: This exit code is caused by fatal errors in business code, such as calling non-existent functions or syntax errors, and the specific error message will be recorded in the file specified in [Worker::logFile](worker/log-file.md) and, if specified, in the file specified in [error_log](https://php.net/manual/en/errorfunc.configuration.php#ini.error-log) in php.ini.
* 64000: This exit code is caused by business code throwing exceptions without catching them, resulting in the process exiting. If Workerman is running in debug mode, the exception call stack will be printed to the terminal, otherwise, it will be recorded in the file specified in [Worker::stdoutFile](worker/stdout-file.md).

## PROCESS STATUS

pid: Process ID

memory: The current memory usage of the process (excluding the memory usage of the PHP executable)

listening: Transmission layer protocol and listening IP:port. If it is not listening on any port, it shows none. Refer to [Worker class constructor](worker/construct.md).

worker_name: The name of the service the process is running. See [Worker class name attribute](worker/name.md).

connections: The current number of TCP connection instances in the process, including TcpConnection and AsyncTcpConnection instances. This value is real-time, not cumulative. Note: If the connection instance is closed but the corresponding count is not reduced, it may be because the business code has saved the $connection object, preventing the connection instance from being destroyed.

total_request: Represents the total number of requests received by the process from startup to now. This includes not only requests from clients, but also internal communication requests within Workerman, such as GatewayWorker's communication requests between Gateway and BusinessWorker. This value is cumulative.

send_fail: The number of times the process failed to send data to clients. The failure is generally due to the client disconnecting. If this value is not 0, it generally indicates a normal state. See [the reasons for send_fail in status](../faq/about-send-fail.md). This value is cumulative.

timers: The number of active timers in the process (excluding deleted timers and one-time timers that have already run). Note: This feature requires Workerman version >=3.4.7. This value is real-time, not cumulative.

qps: The number of network requests received by the process per second. Note: Only with `-d` added to the status will this option be counted; otherwise, it will display 0. This feature requires Workerman version >=3.5.2. This value is real-time, not cumulative.

status: Process status, showing idle if it is idle, and busy if it is busy. Note: If the process is briefly busy, it is normal. If the process is consistently busy, there may be a business blockage or a business dead loop situation, which needs to be investigated based on the section [Debugging Busy Processes](busy-process.md). Note: This feature requires Workerman version >=3.5.0.

## Principle

After running the status script, the main process sends a `SIGUSR2` signal to all worker processes. Then, the status script enters a brief sleep stage to wait for the status statistics from each worker process. When an idle worker process receives the `SIGUSR2` signal, it immediately writes its own running status (e.g., connection count, request count, etc.) to a specific disk file, while a worker process processing business logic waits to finish processing business logic before writing its status information. After a brief sleep, the status script starts reading the status files from the disk and displays the results on the console.

## Note

When using the status command, you may find that some processes are busy. This is because the process is busy processing business logic (e.g., long-term blocking in curl or database requests, running large loops), and cannot report its status, resulting in a busy display.

When this issue occurs, you need to troubleshoot the business code to find where the business is blocked for a long time and evaluate whether the blocked time is within expectations. If it does not meet expectations, you need to troubleshoot the business code based on the section [Debugging Busy Processes](busy-process.md).
