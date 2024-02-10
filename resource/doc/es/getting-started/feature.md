# Características de WorkerMan

### 1. Desarrollo en PHP puro
Las aplicaciones desarrolladas con WorkerMan pueden funcionar de forma independiente sin depender de contenedores como php-fpm, apache o nginx. Esto facilita a los desarrolladores de PHP la creación, implementación y depuración de aplicaciones.

### 2. Soporte para múltiples procesos en PHP
Para aprovechar al máximo el rendimiento de varios CPU del servidor, WorkerMan admite de forma predeterminada múltiples procesos y tareas. WorkerMan inicia un proceso principal y varios subprocesos para brindar servicios, donde el proceso principal supervisa los subprocesos y estos últimos son responsables de escuchar las conexiones de red, así como de recibir, enviar y procesar datos. Gracias a su modelo de procesos simple, WorkerMan es más estable y eficiente.

### 3. Soporte para TCP y UDP
WorkerMan es compatible con los protocolos de capa de transporte TCP y UDP, y solo se necesita cambiar un atributo para cambiar el protocolo de capa de transporte sin necesidad de modificar el código de la aplicación.

### 4. Soporte para conexiones persistentes
En muchas ocasiones, las aplicaciones en PHP necesitan mantener conexiones persistentes con los clientes, como en salas de chat, juegos, entre otros. Sin embargo, los contenedores tradicionales de PHP (apache, nginx, php-fpm) tienen dificultades para lograr esto. Con WorkerMan, siempre que el negocio del servidor no cierre activamente la conexión, se pueden utilizar conexiones persistentes en PHP. Un solo proceso de WorkerMan puede admitir decenas de miles de conexiones concurrentes, y múltiples procesos pueden admitir hasta cientos de miles o incluso millones de conexiones concurrentes.

### 5. Soporte para varios protocolos de capa de aplicación
La interfaz de WorkerMan es compatible con varios protocolos de capa de aplicación, incluidos los protocolos personalizados. Cambiar el protocolo en WorkerMan es extremadamente sencillo, solo requiere configurar un campo, lo que permite el cambio automático a un nuevo protocolo sin modificar el código de la aplicación, e incluso es posible habilitar múltiples puertos con diferentes protocolos para satisfacer distintas necesidades de los clientes.

### 6. Soporte para alta concurrencia
WorkerMan es compatible con la biblioteca de eventos Libevent (requiere la instalación de la extensión "event"). Utilizando Event, se logra un rendimiento excepcional en casos de alta concurrencia; en caso de no tener instalada la extensión Event, se utilizan las llamadas al sistema relacionadas con Select incluidas en PHP, también ofreciendo un rendimiento sorprendente.

### 7. Soporte para reinicio suave del servicio
Cuando es necesario reiniciar el servicio (por ejemplo, para implementar una nueva versión), es deseable que los procesos que están atendiendo las solicitudes de los usuarios no se detengan de inmediato, ni que el reinicio provoque fallas en la comunicación con los clientes. WorkerMan proporciona la función de reinicio suave que garantiza una actualización sin inconvenientes del servicio, sin afectar el uso por parte de los clientes.

### 8. Soporte para la detección y recarga automáticas de archivos
En el proceso de desarrollo, es conveniente que los cambios en el código surtan efecto de inmediato para permitir la visualización de los resultados. WorkerMan ofrece el componente de monitoreo de archivos [FileMonitor](../components/file-monitor.md), que, al detectar una actualización en los archivos, permite la recarga automática de WorkerMan para cargar los nuevos archivos y hacer que surtan efecto.

### 9. Soporte para la ejecución de subprocesos por usuarios específicos
Dado que los subprocesos son los encargados de procesar las solicitudes de los usuarios, por razones de seguridad, estos no deben tener permisos demasiado elevados. Por esta razón, WorkerMan permite configurar el usuario que ejecutará los subprocesos, lo que mejora la seguridad del servidor.

### 10. Soporte para mantener permanentemente objetos o recursos
WorkerMan solo analiza y carga los archivos PHP una vez durante su ejecución, y los mantiene en la memoria en forma permanente. Esto significa que las declaraciones de clases y funciones, el entorno de ejecución de PHP, la tabla de símbolos, entre otros, no se crean y destruyen repetidamente, lo que difiere completamente del mecanismo de ejecución de PHP en los contenedores web. En WorkerMan, los miembros estáticos o variables globales permanecen permanentes durante el ciclo de vida de un proceso, lo que permite reutilizar objetos o conexiones y otros recursos a lo largo de todas las solicitudes dentro de un proceso. Por ejemplo, una vez que se inicializa la conexión a la base de datos en un solo proceso, todas las solicitudes de ese proceso pueden reutilizar esa misma conexión, evitando así la necesidad de establecer frecuentemente la conexión, el proceso de tres vías de TCP, la verificación de permisos de la base de datos, el proceso de cuatro vías de TCP al finalizar la conexión, lo que mejora considerablemente la eficiencia del programa de aplicación.

### 11. Alto rendimiento
Dado que el archivo PHP se lee desde el disco, se analiza una vez y luego se mantiene en memoria, la próxima vez que se utiliza, se utiliza directamente el opcode en memoria, lo que reduce significativamente las operaciones de E/S en disco, como la inicialización de la solicitud en PHP, la creación del entorno de ejecución, el análisis léxico, el análisis sintáctico, la compilación de opcode, el cierre de la solicitud, entre otros procesos que conllevan tiempo. Además, al no depender de contenedores como nginx, apache, se reducen los costos de comunicación con PHP a través de nginx y otros contenedores. Lo más importante es que los recursos pueden mantenerse permanentemente, evitando la necesidad de inicializar la conexión a la base de datos en cada solicitud, lo que permite lograr un alto rendimiento al desarrollar aplicaciones con WorkerMan.

### 12. Soporte para HHVM
WorkerMan es compatible con la máquina virtual HHVM, lo que permite mejorar el rendimiento de PHP de forma significativa. Especialmente en situaciones de cálculos intensivos en la CPU, el rendimiento es excelente. Según pruebas de rendimiento comparativas reales, en situaciones de negocio sin carga, WorkerMan mejora el rendimiento de red en un 30-80% cuando se ejecuta en HHVM en comparación con PHP 5.6 de Zend.

### 13. Soporte para despliegue distribuido

### 14. Soporte para procesos en modo daemon

### 15. Soporte para la escucha en múltiples puertos

### 16. Soporte para redirección de entrada y salida estándar
