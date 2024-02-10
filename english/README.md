# Preface

**Workerman, a high-performance PHP application container**

## What is Workerman?
Workerman is an open-source high-performance PHP application container developed purely in PHP.

Workerman is not reinventing the wheel. It is not an MVC framework but a lower-level, more general-purpose service framework. You can use it to develop TCP proxies, VPN proxies, game servers, mail servers, FTP servers, and even develop a PHP version of Redis, a PHP version of a database, a PHP version of Nginx, a PHP version of PHP-FPM, and much more. Workerman can be considered as an innovation in the PHP field, freeing developers from the constraints of only being able to develop web applications in PHP.

In reality, Workerman is similar to a PHP version of Nginx. Its core also consists of multiple processes, Epoll, and non-blocking IO. Each Workerman process can maintain tens of thousands of concurrent connections. Because it resides in memory and does not depend on containers such as Apache, Nginx, or PHP-FPM, it has extremely high performance. It also supports TCP, UDP, UNIXSOCKET, long connections, WebSockets, HTTP, WSS, HTTPS, and various custom protocols. It includes timers, asynchronous socket clients, asynchronous Redis, asynchronous HTTP, asynchronous message queues, and numerous other high-performance components.

## Some Application Directions of Workerman
Unlike traditional MVC frameworks, Workerman can be used not only for web development but also for a broader range of applications such as real-time communication, IoT, games, service governance, and other servers or middleware, significantly expanding the horizons of PHP developers. Currently, PHP developers in these domains are in short supply. For those who want to have their own technical advantage in the PHP field, are not satisfied with everyday CRUD work, or want to move towards becoming an architect or a tech expert, Workerman is a framework worth studying. It is recommended that developers not only use it but also develop their own open-source projects based on Workerman to enhance their skills and increase their impact. For example, [Beanbun, a multi-process web crawler framework](https://github.com/kiddyuchina/Beanbun), has received numerous accolades shortly after its launch.

Some application directions of Workerman are as follows:

1. Real-time communication:
   Examples include web-based instant messaging, real-time message pushing, WeChat mini-programs, mobile app message pushing, PC software message pushing, etc.
   [[Examples: workerman-chat chat room](https://www.workerman.net/workerman-chat), [web message push](https://www.workerman.net/web-sender), [Todpole chat room](https://www.workerman.net/workerman-todpole)]

2. Internet of Things:
   Examples include communication with printers, microcontrollers, smart bracelets, smart homes, shared bicycles, etc.
   [Customer cases such as Yilianyun, Yiposhi Dai]

3. Game servers:
   Examples include card games, MMORPG games, etc.
   [[Example: browserquest-php](https://www.workerman.net/browserquest)]

4. HTTP services:
   For building high-performance HTTP interfaces and websites. If you want to work on HTTP-related services or sites, [webman](https://github.com/walkor/webman) is highly recommended.

5. SOA service:
   Utilize Workerman to encapsulate different functional units of existing businesses as services to provide a unified interface to the outside world, achieving loose coupling, easy maintenance, high availability, and easy scalability.
   [[Examples: workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc), [workerman-thrift](https://github.com/walkor/workerman-thrift)]

6. Other server software:
   Examples include [GatewayWorker](https://www.workerman.net/doc/gateway-worker), [PHPSocket.IO](https://www.workerman.net/phpsocket_io), [HTTP proxy](https://github.com/walkor/php-http-proxy), [socks5 proxy](https://github.com/walkor/php-socks5), distributed communication components, distributed variable sharing components, message queues, DNS servers, WebServers, CDN servers, FTP servers, etc.

7. Components:
   Examples include asynchronous Redis, asynchronous HTTP clients, IoT MQTT clients, message queues [workerman/redis-queue](components/workerman-redis-queue.md), [workerman/stomp](components/workerman-stomp.md), [workerman/rabbitmq](components/workerman-rabbitmq.md), file monitoring components, and many third-party developed component frameworks.

It's clear that traditional MVC frameworks have difficulty achieving the above functionalities, and that's the reason for the birth of Workerman.

## Workerman Philosophy
Minimalist, stable, high-performance, distributed.

### **Minimalist**
Small is beautiful. Workerman's core is minimalist, comprising only a few PHP files and exposing only a few interfaces, making it very easy to learn. All other functionalities are expanded through components.

Workerman has comprehensive documentation, an authoritative home page, an active community, several large QQ groups, numerous high-performance components, and countless examples, making it easier for developers to use.

### **Stable**
Workerman has been open-source for several years and is widely used by many publicly-listed companies, proven to be extremely stable. Some services have been running for over two years without restarting, with no core dumps, memory leaks, or bugs.

### **High Performance**
Because Workerman resides in memory and does not depend on Apache/Nginx/PHP-FPM, it has no communication overhead between containers and PHP, nor the overhead of initializing and destroying everything for each request. It delivers exceptionally high performance, surpassing traditional MVC frameworks by tens of times. Under PHP7, it achieves a QPS in ab stress tests that may even exceed that of standalone Nginx.

### **Distributed**
In the present era, even the most powerful single server has its limits, and it's the era of distributed multi-server deployment. Workerman directly provides a long-term distributed communication solution [GatewayWorker framework](https://doc2.workerman.net). Adding servers only requires simple configuration and startup, with no changes to the business code, greatly increasing the system's capacity. If you are developing TCP long connections, it is recommended to use [GatewayWorker](https://doc2.workerman.net), which is a wrapper for Workerman, providing richer interfaces and robust distributed processing capabilities specifically for long connection applications.

## Scope of This Handbook
Workerman 3.x - 4.x versions

## For Windows Users (Must Read)
Workerman supports both Linux and Windows systems. The Windows version of Workerman **does not rely on any extensions** and only requires the PHP environment to be properly configured. For details on installing and important considerations for the Windows version of Workerman, please see [Windows Users Must Read](https://www.workerman.net/windows).

## Client
WorkerMan's communication protocol is open and customizable. Therefore, in theory, WorkerMan can communicate with clients on any platform using any protocol. When developing clients, users can complete communication with the server based on the corresponding communication protocol.
