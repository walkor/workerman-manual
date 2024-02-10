# Prefacio

**Workerman, contenedor de aplicaciones PHP de alto rendimiento**

## ¿Qué es Workerman?
Workerman es un contenedor de aplicaciones de PHP de alto rendimiento y código abierto desarrollado puramente en PHP.

Workerman no es un refrito, no es un marco de trabajo MVC, sino que es un marco de servicio más profundo y generalizado. Puedes usarlo para desarrollar un proxy TCP, un proxy de red privada virtual (VPN), servidores de juegos, servidores de correo, servidores FTP e incluso desarrollar una versión de PHP de Redis, una versión de base de datos de PHP, una versión de PHP de Nginx, y una versión de PHP de PHP-FPM, entre otros. Workerman puede considerarse una innovación en el campo de PHP, liberando por completo a los desarrolladores de la limitación de que PHP solo pueda ser utilizado para aplicaciones web.

En realidad, Workerman es similar a una versión en PHP de Nginx, con un núcleo de múltiples procesos + Epoll + E/S no bloqueante. Cada proceso de Workerman puede manejar decenas de miles de conexiones concurrentes. Debido a que reside en la memoria en todo momento y no depende de contenedores como Apache, Nginx o PHP-FPM, ofrece un rendimiento extremadamente alto. Además, admite TCP, UDP, UNIXSOCKET, conexiones persistentes, así como protocolos de comunicación como Websocket, HTTP, WSS, HTTPS y varios protocolos personalizados. También cuenta con temporizadores, clientes de sockets asincrónicos, Redis asincrónico, HTTP asincrónico, colas de mensajes asincrónicas, y muchos otros componentes de alto rendimiento.

## Algunas direcciones de aplicación de Workerman
Diferente de los marcos de trabajo MVC tradicionales, Workerman no solo puede ser utilizado para el desarrollo web, sino que también tiene un campo de aplicación más amplio, como mensajería instantánea, Internet de las cosas, juegos, gobierno de servicios, servidores y middleware, lo que sin duda amplía la visión de los desarrolladores de PHP. Actualmente, hay una gran escasez de desarrolladores de PHP en estos campos. Si deseas tener una ventaja técnica en el campo de PHP, o si no estás satisfecho con el trabajo diario de CRUD, o si quieres avanzar en dirección a la arquitectura o convertirte en un experto técnico, Workerman es un marco que vale la pena aprender. Se recomienda que los desarrolladores no solo lo usen, sino también que desarrollen sus propios proyectos de código abierto basados en Workerman para mejorar sus habilidades y aumentar su influencia. Por ejemplo, [Beanbun, un marco de rastreo de redes de múltiples procesos](https://github.com/kiddyuchina/Beanbun) es un excelente ejemplo, que recibió muchas críticas favorables poco después de su lanzamiento.

Algunas de las direcciones de aplicación de Workerman son las siguientes:

1. Mensajería instantánea
   Por ejemplo, chat en tiempo real en páginas web, envío de mensajes en tiempo real, aplicaciones de mensajería de WeChat, envío de mensajes de aplicaciones móviles, envío de mensajes de software de PC, etc.
   [[Ejemplos de workerman-chat, sala de chat](https://www.workerman.net/workerman-chat), [Envío de mensajes web](https://www.workerman.net/web-sender), [Sala de chat de renacuajos](https://www.workerman.net/workerman-todpole)]

2. Internet de las cosas
   Comunicación con impresoras, microcontroladores, pulseras inteligentes, hogares inteligentes, bicicletas compartidas, etc.
   [Ejemplos de clientes como Yilianyun, Yiboshi era, etc.]

3. Servidores de juegos
   Juegos de mesa, juegos MMORPG, etc.
   [[Ejemplo de browserquest-php](https://www.workerman.net/browserquest)]

4. Servicios HTTP
   Creación de interfaces HTTP de alto rendimiento, sitios web de alto rendimiento. Si deseas ofrecer servicios HTTP o construir un sitio web, se recomienda encarecidamente [webman](https://github.com/walkor/webman).

5. Servicios SOA
   Utilizando Workerman para encapsular diferentes unidades de funcionalidades comerciales existentes en forma de servicios y proporcionar una interfaz unificada hacia afuera. Esto permite que el sistema sea flexible, fácil de mantener, altamente disponible y escalable.
   [[Ejemplo de trabajo workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc), [workerman-thrift](https://github.com/walkor/workerman-thrift)]

6. Otros software de servidor
   [GatewayWorker](https://www.workerman.net/doc/gateway-worker), [PHPSocket.IO](https://www.workerman.net/phpsocket_io), [proxy HTTP](https://github.com/walkor/php-http-proxy), [proxy SOCKS5](https://github.com/walkor/php-socks5), [componente de comunicación distribuida](https://github.com/walkor/Channel), [componente de intercambio de variables distribuidas](https://github.com/walkor/GlobalData), [colas de mensajes](https://github.com/walkor/workerman-queue), servidores DNS, servidores web, servidores CDN, servidores FTP, etc.

7. Componentes
   [Redis asincrónico](components/workerman-redis.md), [cliente HTTP asincrónico](components/workerman-http-client.md), [cliente MQTT para Internet de las cosas](components/workerman-mqtt.md), [cola de mensajes de Redis workerman](components/workerman-redis-queue.md), [colas de mensajes STOMP](components/workerman-stomp.md), [workerman/rabbitmq](components/workerman-rabbitmq.md), [componente de monitoreo de archivos](components/file-monitor.md), y muchos otros marcos de componentes de terceros, etc.

Claramente, es muy difícil para el marco MVC tradicional lograr las funcionalidades mencionadas anteriormente, y es por eso que nace Workerman.

## Filosofía de Workerman
Simple, estable, de alto rendimiento y distribuido.

### **Simple**
La simplicidad es atractiva. El núcleo de Workerman es simple, compuesto por solo unos pocos archivos PHP y expone solo unos pocos interfaces, lo que lo hace muy fácil de aprender. Todas las demás funciones se expanden a través de componentes.

Workerman cuenta con una documentación completa, un sitio web autorizado, una comunidad activa, varios grupos de QQ con miles de personas, una gran cantidad de componentes de alto rendimiento, y muchas ejemplos, todo esto hace que el uso de Workerman sea más intuitivo.

### **Estable**
Workerman ha sido de código abierto durante varios años y ha sido ampliamente utilizado por muchas empresas cotizadas. Es extremadamente estable. Algunos servicios han estado funcionando sin reiniciarse durante más de dos años a alta velocidad. No se producen core dump, fugas de memoria ni errores.

### **Alto rendimiento**
Debido a que Workerman reside en la memoria en todo momento, no depende de Apache/Nginx/PHP-FPM, no hay gastos generales de comunicación entre contenedores de PHP, y no hay gastos de inicialización y eliminación de todo en cada solicitud. Por lo tanto, tiene un rendimiento extremadamente alto, hasta varias veces más alto que el de los marcos MVC tradicionales, e incluso bajo pruebas de presión con ab, el rendimiento de QPS en PHP7 incluso puede superar al de Nginx por sí solo.

### **Distribuido**
En la actualidad, ya no es un mundo de un solo lobo, y aunque el rendimiento de un solo servidor sea impresionante, la verdadera escala se encuentra en las implementaciones distribuidas de múltiples servidores. Workerman proporciona directamente un conjunto de soluciones de comunicación distribuida de larga duración [GatewayWorker framework](https://doc2.workerman.net). Agregar servidores es tan simple como configurarlos y ponerlos en marcha sin cambiar ninguna línea de código comercial, lo que multiplica la capacidad del sistema. Si estás desarrollando una aplicación TCP de larga duración, se recomienda utilizar [GatewayWorker](https://doc2.workerman.net), que es un envoltorio de Workerman que proporciona una interfaz más rica y una poderosa capacidad de procesamiento distribuido centrado en aplicaciones de larga duración.

## Alcance de este manual
Versiones de Workerman 3.x - 4.x

## Usuarios de Windows (lectura obligatoria)
Workerman es compatible con sistemas Linux y Windows. La versión de Windows de Workerman en sí **no depende de ninguna extensión**, solo necesita configurar correctamente la variable de entorno de PHP. **Para obtener información sobre la instalación y consideraciones importantes de la versión de workerman en Windows, consulta [Windows Users Must Read](https://www.workerman.net/windows).**

## Cliente
El protocolo de comunicación de WorkerMan es abierto y personalizable, por lo tanto, en teoría, WorkerMan puede comunicarse con clientes de plataformas y protocolos utilizando cualquier protocolo. Al desarrollar clientes, puedes completar la comunicación con el servidor según el protocolo de comunicación correspondiente.
