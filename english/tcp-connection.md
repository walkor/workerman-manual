# Interfaces Provided by the Connection Class

In Workerman, there are two important classes, Worker and Connection.

Each client connection corresponds to a Connection object, which allows developers to set callbacks such as onMessage and onClose. It also provides interfaces for sending data to clients (send) and closing connections (close), as well as other necessary interfaces.

It can be said that Worker is a listening container responsible for accepting client connections and wrapping the connections into Connection objects for developers to operate.
