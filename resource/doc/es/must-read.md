# Algunas preguntas que los desarrolladores deben conocer sobre Workerman

**1. Limitaciones del entorno de Windows**

En el sistema Windows, un solo proceso de Workerman solo admite 200+ conexiones.
En el sistema Windows, no es posible utilizar el parámetro "count" para configurar múltiples procesos.
En el sistema Windows, no es posible utilizar comandos como "status", "stop", "reload", "restart", etc.
En el sistema Windows, no es posible ejecutar el proceso en segundo plano, ya que el servicio se detendrá cuando se cierre la ventana del cmd.
En el sistema Windows, no es posible inicializar múltiples escuchas en un solo archivo.
En el sistema Linux, no hay limitaciones como las mencionadas anteriormente. Se recomienda utilizar el sistema Linux para entornos de producción, mientras que el entorno de desarrollo puede optar por utilizar el sistema Windows.

**2. Workerman no depende de Apache o Nginx**

Workerman en sí mismo es un contenedor similar a Apache/Nginx. Si el entorno de PHP está bien configurado, Workerman puede funcionar.

**3. Workerman se inicia desde la línea de comandos**

El método de inicio es similar al de Apache, ya que se utiliza un comando para iniciarlo (generalmente no se puede usar en un espacio web). La interfaz de inicio aparece de la siguiente forma:
![Captura de pantalla](image/screenshot_1495622774534.png)

**4. Las conexiones largas deben incluir un mecanismo de latido**

Es imprescindible incluir un mecanismo de latido para las conexiones largas. Es crucial repetir tres veces que las conexiones largas deben incluir un mecanismo de latido. Si no hay comunicación durante mucho tiempo, la conexión se cerrará debido a la limpieza realizada por el nodo de enrutamiento. Se puede obtener más información sobre el latido en [Workerman Latido](faq/heartbeat.md) y [GatewayWorker Latido](https://www.workerman.net/doc/gateway-worker/heartbeat.html).

**5. El protocolo del cliente y del servidor debe coincidir para establecer una comunicación**

Este es un problema común entre los desarrolladores. Por ejemplo, si el cliente está utilizando el protocolo WebSocket, el servidor también debe utilizar el protocolo WebSocket (se puede hacer mediante `new Worker('websocket://0.0.0.0...')`) para establecer una conexión y comunicarse. No se debe intentar acceder al puerto del protocolo WebSocket desde la barra de direcciones del navegador ni utilizar el protocolo WebSocket para acceder al puerto del protocolo TCP sin formato. Es fundamental que los protocolos coincidan.

Esto es similar a la idea de que si quieres comunicarte con alguien que habla inglés, tú también debes hablar inglés. Si quieres comunicarte con alguien que habla japonés, debes hablar japonés. En este caso, el lenguaje se asemeja al protocolo de comunicación; ambas partes (cliente y servidor) deben utilizar el mismo lenguaje para comunicarse. De lo contrario, la comunicación no será posible.

**6. Posibles causas de fallo en la conexión**

Es común tener problemas al iniciar Workerman, especialmente, la conexión del cliente con el servidor falla. Las causas suelen ser las siguientes:
1. El cortafuegos del servidor (incluyendo el grupo de seguridad del servidor en la nube) bloquea la conexión (aproximadamente un 50% de probabilidad).
2. El cliente y el servidor utilizan protocolos diferentes (aproximadamente un 30% de probabilidad).
3. La dirección IP o el puerto están mal escritos (aproximadamente un 15% de probabilidad).
4. El servidor no ha sido iniciado.

**7. No utilizar las declaraciones exit, die o sleep**

Si el código del negocio ejecuta las declaraciones exit o die, provocará la terminación del proceso y mostrará un error de "WORKER EXIT UNEXPECTED". Sin embargo, el proceso se reiniciará inmediatamente para continuar el servicio. Si es necesario realizar un retorno, se debe utilizar la declaración return. La declaración sleep hace que el proceso entre en modo de suspensión, durante el cual no se ejecutará ningún negocio y el marco de trabajo se detendrá, resultando en la incapacidad para manejar las solicitudes de los clientes en dicho proceso.

**8. No utilizar la función pcntl_fork**

El uso de `pcntl_fork` para crear dinámicamente nuevos procesos en el código del negocio puede conducir a la generación de procesos huérfanos que no se pueden recuperar, lo que ocasiona anomalías en el negocio. La utilización de `pcntl_fork` en el negocio también afectará el manejo de eventos como conexiones, mensajes, cierre de conexiones y temporizadores, lo que puede resultar en excepciones imprevistas.

**9. Evitar bucles infinitos en el código del negocio**

No es recomendable tener bucles infinitos en el código del negocio, ya que esto impedirá que el control se entregue al marco de trabajo de Workerman, lo que a su vez resultará en la incapacidad para recibir y manejar mensajes de otros clientes.

**10. Reiniciar Workerman después de realizar cambios en el código**

Workerman es un marco de trabajo que reside en memoria. Por lo tanto, es necesario reiniciar Workerman después de realizar cambios en el código para que los cambios surtan efecto.

**11. Recomendación de utilizar el marco de trabajo GatewayWorker para aplicaciones de conexión larga**

Muchos desarrolladores utilizan Workerman para desarrollar aplicaciones de **conexión larga**, como la mensajería instantánea y la Internet de las cosas. Para este tipo de aplicaciones, se recomienda utilizar directamente el marco de trabajo GatewayWorker, que ofrece una capa adicional de abstracción sobre Workerman y facilita el desarrollo de aplicaciones de conexión larga.

**12. Soporte para una mayor concurrencia**

Si el número de conexiones simultáneas en el negocio supera las 1000, es fundamental realizar [optimizaciones en el núcleo de Linux](appendices/kernel-optimization.md) y [instalar la extensión event](appendices/install-extension.md).
