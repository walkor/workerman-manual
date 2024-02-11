# Workerman soporta cuántas conexiones concurrentes

El concepto de **conexiones concurrentes** es bastante vago, así que aquí se explicarán dos métricas cuantificables: **número de conexiones concurrentes** y **número de solicitudes concurrentes**.

El **número de conexiones concurrentes** se refiere a cuántas conexiones TCP mantiene el servidor en un momento dado, independientemente de si hay comunicación de datos en esas conexiones. Por ejemplo, un servidor de envío de mensajes puede mantener millones de conexiones de dispositivos, pero como hay muy poca comunicación de datos en esas conexiones, la carga en este servidor podría ser casi nula. Mientras la memoria sea suficiente, el servidor puede seguir recibiendo conexiones.

El **número de solicitudes concurrentes** generalmente se mide en QPS (peticiones por segundo que el servidor puede procesar). En este caso, no importa tanto el número actual de conexiones TCP en el servidor. Por ejemplo, un servidor puede tener solo 10 conexiones de clientes, pero si cada conexión recibe 10,000 solicitudes por segundo, el servidor debe poder manejar un rendimiento mínimo de 10 * 10,000 = 100,000 solicitudes por segundo (QPS). Suponiendo que 100,000 QPS es el límite de este servidor, si cada cliente envía solo una solicitud por segundo, el servidor puede manejar 100,000 clientes.

El **número de conexiones concurrentes** está limitado por la memoria del servidor, y generalmente un servidor Workerman con 24 GB de memoria puede admitir aproximadamente **1.2 millones** de conexiones concurrentes.

El **número de solicitudes concurrentes** está limitado por la capacidad de procesamiento de la CPU del servidor, y un servidor Workerman de 24 núcleos puede alcanzar un rendimiento de hasta **450,000** solicitudes por segundo (QPS). El valor real variará según la complejidad del negocio y la calidad del código.

## Nota

En un escenario de alta concurrencia, es imprescindible instalar la extensión event, consulte la sección de instalación y configuración. Además, se necesita optimizar el kernel de Linux, especialmente el límite de número de archivos abiertos por proceso, consulte el apéndice sobre la optimización del kernel.

## Datos de pruebas de rendimiento
> Los datos proceden de la 20ª ronda de pruebas de rendimiento de una institución de pruebas de autoridad de terceros, techempower.com
https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf

**Configuración del servidor:**
Núcleos totales 14, hilos totales 28, 32 GB de memoria, switch Ethernet Cisco dedicado de 10 gigabits.
**Lógica empresarial:**
Con negocios de consultas a bases de datos, base de datos pgsql, php8+jit
QPS es de más de 390,000+
![](../images/screenshot_1636522357217.png)
