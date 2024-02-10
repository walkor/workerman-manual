# Procesos concentran solicitudes

### Fenómeno
A veces, al ejecutar el comando `php start.php status`, observamos que las solicitudes se concentran en procesos específicos, dejando a los demás procesos completamente inactivos.

### Mecanismo de prelación
La forma en que los múltiples procesos de Workerman obtienen conexiones es **por defecto** **prelativa**, lo que significa que cuando un cliente inicia una conexión, todos los procesos inactivos tienen la oportunidad de obtener esa conexión, y el más rápido será el primero en obtenerla. La rapidez está determinada por la programación del núcleo del sistema operativo. El sistema operativo puede optar por dar prioridad al proceso que se usó más recientemente para otorgarle el uso de la CPU, ya que los registros de la CPU pueden contener información del contexto del proceso anterior, lo que puede reducir el costo de cambiar de contexto. Por lo tanto, cuando la operación es lo suficientemente rápida o durante la prueba de estrés, es más probable que ocurra la concentración de conexiones en ciertos procesos, ya que esta estrategia evita cambios frecuentes de procesos, lo cual suele dar la mejor optimización del rendimiento y no es necesariamente un problema.

### Mecanismo de rotación
Workerman puede cambiar el mecanismo de obtención de conexiones a un método de **rotación** configurando `$worker->reusePort = true;`. En el mecanismo de rotación, el núcleo del sistema opera para distribuir las conexiones de manera equitativa entre todos los procesos, de modo que todos los procesos manejen las solicitudes juntos.

### Malentendido
Muchos desarrolladores creen que cuanto más procesos participen en el procesamiento de las solicitudes, mejor será el rendimiento, pero esto no siempre es cierto. Cuando la operación es lo suficientemente simple, el rendimiento es mejor cuando el número de procesos involucrados en el procesamiento de las solicitudes se acerca al número de núcleos de la CPU del servidor. Por ejemplo, en un servidor de 4 núcleos, el rendimiento suele ser óptimo cuando se configura el número de procesos en 4 para la prueba de rendimiento de "hola mundo”. Si el número de procesos involucrados en el procesamiento supera significativamente la cantidad de núcleos de la CPU, el costo del contexto del proceso será mayor y el rendimiento será peor. Generalmente, para operaciones que involucran bases de datos, el rendimiento puede ser mejor cuando el número de procesos se establece entre 3 y 6 veces la cantidad de núcleos de la CPU.
