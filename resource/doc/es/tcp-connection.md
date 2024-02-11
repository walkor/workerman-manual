# Interfaz proporcionada por la clase Connection

En Workerman, hay dos clases importantes: Worker y Connection.

Cada conexión de cliente corresponde a un objeto Connection, que puede configurar devoluciones de llamada como onMessage, onClose, etc. También proporciona interfaces para enviar datos al cliente (send) y cerrar la conexión (close), así como otras interfaces necesarias.

Se puede decir que Worker es un contenedor de escucha responsable de aceptar las conexiones de los clientes y proporcionar objetos de conexión para que los desarrolladores los manipulen.
