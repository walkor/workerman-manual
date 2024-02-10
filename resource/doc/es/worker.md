# Clase Worker
En WorkerMan, hay dos clases importantes: Worker y Connection.

La clase Worker se utiliza para escuchar puertos y se puede configurar funciones de devolución de llamada para eventos como la conexión de clientes, los mensajes recibidos y la desconexión. Esto permite implementar el procesamiento de negocios.

Se puede configurar el número de procesos de una instancia de Worker (propiedad count). El proceso principal de Worker bifurcará count subprocesos para escuchar el mismo puerto y recibir conexiones de clientes de forma paralela, y procesar los eventos de conexión.
