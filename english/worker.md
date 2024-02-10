# Worker Class
In Workerman, there are two important classes: Worker and Connection.

The Worker class is used to listen on ports and can set callback functions for client connection events, message events, and connection close events in order to implement business logic.

The number of processes for a Worker instance can be set using the count property. The Worker's main process will fork out count child processes to listen on the same port, enabling parallel reception of client connections and handling of connection events.
