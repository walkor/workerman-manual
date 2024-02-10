# Features of Workerman

### 1. Pure PHP Development
Applications developed using Workerman can run independently without relying on containers such as php-fpm, apache, or nginx. This makes it very convenient for PHP developers to develop, deploy, and debug applications.

### 2. Support for PHP Multi-Process
To fully utilize the performance of servers with multiple CPUs, Workerman supports multi-process multi-tasking by default. Workerman starts a main process and multiple child processes to provide services. The main process is responsible for monitoring the child processes, while the child processes independently listen to network connections, receive and process data. Due to the simple process model, Workerman is more stable and efficient.

### 3. Support for TCP and UDP
Workerman supports both TCP and UDP transport layer protocols, and switching between them only requires changing a single attribute, without the need to modify the business code.

### 4. Support for Long Connections
Often, PHP applications need to maintain long connections with clients, such as in chat rooms or games. However, traditional PHP containers (apache, nginx, php-fpm) find it challenging to achieve this. With Workerman, as long as the server's business does not actively call the close connection interface, PHP long connections can be used. A single Workerman process can support tens of thousands of concurrent connections, while multiple processes can support hundreds of thousands or even millions of concurrent connections.

### 5. Support for Various Application Layer Protocols
Workerman's interface supports various application layer protocols, including custom protocols. Changing protocols in Workerman is equally straightforward, requiring just a configuration field, and the protocol switches automatically without any changes to the business code. It is even possible to enable multiple ports with different protocols to meet the needs of different clients.

### 6. Support for High Concurrency
Workerman supports the Libevent event polling library (requires installation of the event extension). When using Event for high-concurrency long connections, the performance is outstanding. If the Event extension is not installed, Workerman uses the PHP built-in Select related system calls, which also provide strong performance.

### 7. Support for Smooth Service Restart
When it is necessary to restart the service (e.g., for deployment of a new version), it is essential that the processes handling user requests are not immediately terminated and that the restart does not lead to communication failure with clients. Workerman provides smooth restart functionality to ensure seamless service upgrades without affecting client usage.

### 8. Support for File Update Detection and Automatic Loading
During development, it is important that code changes take effect immediately for result verification. Workerman provides the FileMonitor file monitoring component, which automatically triggers a reload when a file is updated, enabling the loading of new files to take effect.

### 9. Support for Running Child Processes as a Specified User
For security reasons, as the child processes are the ones handling user requests, Workerman supports setting the user under which the child processes run to enhance server security.

### 10. Support for Permanent Storage of Objects or Resources
In Workerman, PHP files are parsed and loaded into memory just once, and class and function declarations, PHP execution environment, symbol tables, etc., are not repeatedly created and destroyed. Within the lifespan of a process, static members or global variables are permanently stored if not actively destroyed. This means that objects or connections can be placed in global variables or class static members, and all requests within the process's lifecycle can reuse them. For example, initializing a database connection once within a single process allows all subsequent requests within that process to reuse the same database connection, avoiding the overhead of frequent database connections, such as TCP three-way handshake, database permission verification, and TCP four-way handshake, significantly improving application efficiency.

### 11. High Performance
Because PHP files are loaded into memory after being read and parsed from the disk, and then directly use the opcodes in memory on subsequent uses, it greatly reduces disk IO and the time-consuming processes involved in PHP request initialization, creating execution environments, lexical analysis, syntax analysis, opcode compilation, and request closure, while also eliminating the overhead of communicating with containers like nginx and apache. Most importantly, resources can be kept permanently, eliminating the need to initialize database connections every time, etc. Therefore, developing applications with Workerman ensures very high performance.

### 12. Support for HHVM
Support running on the HHVM virtual machine, which can significantly improve PHP performance. Specifically, in CPU-intensive computational businesses, the performance is outstanding. Through actual stress testing comparisons, in the absence of loaded businesses, Workerman running under HHVM achieves a 30-80% increase in network throughput compared to running under Zend PHP 5.6.

### 13. Support for Distributed Deployment
### 14. Support for Daemonization
### 15. Support for Multi-Port Listening
### 16. Support for Standard Input and Output Redirection
