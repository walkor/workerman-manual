# Principles

### Worker Description
Worker is the most basic container in Workerman, which can start multiple processes to listen on ports and communicate using specific protocols, similar to nginx listening on a port. Each Worker process operates independently, using Epoll (requires the installation of the event extension) and non-blocking IO. Each Worker process can handle tens of thousands of client connections and process data sent over these connections. The main process is responsible for monitoring the child processes to maintain stability, without receiving data or performing any business logic.

### Relationship between Client and Worker Processes
![workerman master worker model](images/Worker.png)

### Relationship between Main Process and Worker Child Processes
![workerman master worker model](images/Worker2.png)

**Features:**

From the diagrams, we can see that each Worker maintains its own client connections, making it easy to achieve real-time communication between clients and servers. Based on this model, we can easily implement some basic development needs, such as HTTP servers, Rpc servers, real-time data reporting for some smart hardware, server-side data push, game servers, WeChat Mini Program backends, and so on.
