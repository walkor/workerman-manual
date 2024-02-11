# Principio

### Descripción del Trabajador
El Trabajador (Worker) es el contenedor más básico en Workerman, que puede abrir múltiples procesos para escuchar puertos y comunicarse usando un protocolo específico, similar a nginx escuchando en un puerto. Cada proceso Worker opera de forma independiente, utilizando Epoll (requiere la extensión event) + E/S no bloqueante. Cada proceso Worker puede manejar decenas de miles de conexiones de clientes y procesar los datos enviados por estas conexiones. El proceso principal se encarga de vigilar a los procesos secundarios para mantener la estabilidad, sin recibir datos ni realizar ninguna lógica de negocio.

### Relación entre el cliente y el proceso Worker
![Modelo maestro y trabajador de Workerman](images/Worker.png)

### Relación entre el proceso principal y los subprocesos Worker
![Modelo maestro y trabajador de Workerman](images/Worker2.png)

**Características:**

En el gráfico, podemos ver que cada Trabajador mantiene sus propias conexiones de clientes, lo que facilita la implementación de la comunicación en tiempo real entre el cliente y el servidor. Basándose en este modelo, podemos satisfacer fácilmente algunas necesidades básicas de desarrollo, como servidores HTTP, servidores RPC, informes en tiempo real de datos de dispositivos inteligentes, envío de datos desde el servidor, servidores de juegos, backend para aplicaciones de WeChat, entre otros.
