# Must-read before development

When developing applications using Workerman, you need to understand the following content:

## I. Differences between Workerman development and regular PHP development

Apart from the fact that variables and functions related to the HTTP protocol cannot be used directly, there are not many differences between Workerman development and regular PHP development.

### 1. Different application layer protocols
- Regular PHP development is generally based on the HTTP application layer protocol, where the WebServer has already completed the protocol parsing for developers.
- Workerman supports various protocols, and currently has built-in support for HTTP, WebSocket, and other protocols. Workerman recommends developers to use simpler custom protocols for communication.

  - For HTTP protocol development, please refer to the [HTTP Service section](../http/request.md).

### 2. Difference in request lifecycle
- In a Web application, PHP releases all variables and resources after each request.
- Application programs developed in Workerman reside in memory after the initial loading and parsing, so the definitions of classes, global objects, and static class members do not get released, making them available for reuse in subsequent requests.

### 3. Avoiding duplicate definitions of classes and constants
- Because Workerman caches compiled PHP files, it is important to avoid multiple require/include of the same class or constant definition files. It's recommended to use require_once/include_once to load files.

### 4. Release of singleton mode connection resources
- Since Workerman does not release global objects and class static members after each request, in singleton mode for resources like databases, the database instance (which includes a database socket connection) is often saved in a static member of the database, allowing Workerman to reuse this database socket connection throughout the process lifecycle. It is important to note that if the database server detects inactivity in a connection for a certain period, it may actively close the socket connection, leading to errors when using this database instance again (similar to "mysql gone away" error). Workerman provides a [database class](../components/workerman-mysql.md) with a reconnect feature, which developers can use directly.

### 5. Avoiding the use of exit and die statements
- Workerman runs in PHP command-line mode, and calling exit or die statements will cause the current process to exit. While the child process will be immediately recreated to continue serving, it may still have an impact on the business.

### 6. Restart the service to apply code changes
Since Workerman resides in memory, PHP class and function definitions are loaded once and remain in memory without disk reads. Therefore, any changes to business code require a restart to take effect.

## II. Basic concepts to understand

### 1. TCP transport layer protocol
TCP is a connection-oriented, reliable, IP-based transport layer protocol. An important feature of the TCP transport layer protocol is that it is based on a data stream. The client's requests are continuously sent to the server, and the server may receive incomplete requests or multiple requests concatenated together. This requires a set of rules to differentiate each request within the continuous data stream, and the application layer protocol mainly defines rules to avoid request data confusion.

### 2. Application layer protocol
The application layer protocol defines how application processes running on different end systems (client, server) exchange messages with each other. Examples of application layer protocols include HTTP and WebSocket. For example, a simple application layer protocol may look like this: ```{"module":"user","action":"getInfo","uid":456}\n```. In this protocol, the request is marked with ```"\n"``` (note that ```"\n"``` represents a line break) to signify the end of the request, and the message body is a string.

### 3. Short connection
A short connection refers to establishing a connection when the two communicating parties have data to exchange. Once the data transmission is complete, the connection is terminated, and each connection completes only one business transaction. This is commonly used in HTTP services on the web.

*The development of short connection applications can refer to the basic development process chapter.*

### 4. Long connection
A long connection allows for continuous transmission of multiple data packets over a single connection.

Note: Long connection applications must include a [heartbeat](../faq/heartbeat.md), as otherwise the connection may be disconnected by the router’s firewall due to long periods of inactivity.

Long connections are often used in scenarios where operations are frequent and involve point-to-point communication. Each TCP connection requires a three-way handshake, which takes time. If each operation involves establishing a connection before the operation, the processing speed would be significantly reduced. Therefore, with long connections, the connection is not terminated after each operation, and the next operation can simply send the data packet without establishing a new TCP connection. For example, long connections are used in database connections, as frequent communication using short connections can cause socket errors and waste resources.

*When there is a need to actively push data to clients, such as in chat applications, real-time gaming, and mobile push applications, long connections are required.*

*The development of long connection applications can refer to the Gateway/Worker development process.*

### 5. Smooth restart
The typical process of restarting involves stopping all processes and then creating completely new service processes. During this process, there is a short period of time when no processes are available to provide services, which can lead to temporary unavailability of the service and a possibility of request failure in high-concurrency scenarios.

On the other hand, a smooth restart does not stop all processes at once, but rather one process at a time. After stopping each process, a new process is immediately created to take its place, and this process is repeated until all old processes have been replaced.

Workerman can use the command ```php your_file.php reload``` for a smooth restart, which allows for updating the application program without affecting service quality.

**Note: Only files loaded in the on{...} callback will be automatically updated after a smooth restart. Files directly loaded in the startup script or hardcoded code will not be automatically updated.**

## III. Distinguishing between main process and child process
It is important to be aware of whether the code is running in the main process or the child process. In general, any code running before the call to ```Worker::runAll();``` runs in the main process, while the code running in onXXX callbacks belongs to the child process. Code written after ```Worker::runAll();``` will never be executed.

For example, in the following code:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Runs in the main process
$tcp_worker = new Worker("tcp://0.0.0.0:2347");
// The assignment process runs in the main process
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // This part runs in the child process
    $connection->send('hello ' . $data);
};

Worker::runAll();
```

**Note: Do not initialize database, memcache, or redis connections in the main process, as connections initialized in the main process may be automatically inherited by child processes, especially when using singletons. All processes will thus hold the same connection, and data returned through this connection is readable by multiple processes, leading to data confusion. Similarly, if any process closes a connection (e.g., when running in daemon mode, the main process exits causing the connection to close), all child processes' connections will be closed together, leading to unpredictable errors such as "mysql gone away" errors. It is recommended to initialize connection resources in onWorkerStart.**
