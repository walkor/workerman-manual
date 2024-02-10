# Request Concentration in Certain Processes

### Phenomenon
Sometimes when we use the command `php start.php status`, we see that requests are concentrated in specific processes, while other processes remain completely idle.

### Preemptive Mechanism
By default, workerman uses a **preemptive** approach for multiple processes to obtain connections. This means that when a client initiates a connection, all idle processes have the opportunity to obtain this connection, and the fastest one gets it. The decision of which process is the fastest is determined by the operating system kernel. The operating system may prioritize the most recently used process to obtain CPU usage rights, because the CPU registers may still contain the context information of the previous process, which can reduce context switching overhead. Therefore, when the business logic is fast enough or during stress testing, it is more likely to occur that connections are concentrated in certain processes, as this strategy can avoid frequent process switches and often results in optimal performance, which is not necessarily a problem.

### Polling Mechanism
Workerman can change the way connections are obtained to a **polling** mode by setting `$worker->reusePort = true;`. In the polling mode, the kernel distributes connections to all processes in an approximately equal manner, so all processes will handle requests together.

### Misconception
Many developers believe that the more processes participating in request processing, the better the performance. In reality, this is not always the case. When the business logic is simple enough, the closer the number of processes participating in request processing is to the number of CPU cores, the higher the server throughput. For example, on a 4-core server, the QPS for a helloworld stress test is usually highest when the number of processes is set to 4. If the number of processes participating in processing exceeds the number of CPU cores by too much, the context switching overhead increases, resulting in worse performance. However, when dealing with general database-related businesses, setting the number of processes to 3-6 times the number of CPU cores may lead to better performance.
