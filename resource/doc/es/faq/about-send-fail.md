# Razones de send_fail en status

**Síntomas:**

Al ejecutar el comando "status", se observa que hay casos de send_fail. ¿Cuál es la causa?

**Respuesta:**

En general, el send_fail no suele ser un problema grave. Por lo general, se debe a la desconexión activa del cliente o a la imposibilidad del cliente de recibir datos, lo que provoca el fallo en el envío de datos.

Hay dos razones para send_fail:

1. Al llamar a la interfaz send para enviar datos al cliente, se descubre que el cliente se ha desconectado, por lo que se suma 1 al contador send_fail. Dado que la desconexión es iniciada por el cliente, esto es un fenómeno normal y generalmente se puede ignorar.

2. La velocidad de envío de datos del servidor es mayor que la velocidad de recepción del cliente, lo que provoca que los datos se acumulen constantemente en el búfer de envío del servidor (workerman crea un búfer de envío para cada cliente). Si el tamaño del búfer supera el límite (TcpConnection::$maxSendBufferSize por defecto es 1 MB), los datos se descartarán, lo que activará el evento onError (si está presente) y aumentará el contador send_fail.

Por ejemplo, cuando se minimiza el navegador, es posible que JavaScript se detenga, lo que provoca que el navegador deje de recibir datos del servidor, lo que lleva a que los datos se acumulen en el búfer, y si supera el límite, cada llamada a send aumentará el contador send_fail.

**Conclusión:**

Por lo general, no es necesario preocuparse por el send_fail causado por la desconexión del cliente.

Si el send_fail es causado por la interrupción en la recepción de datos por parte del cliente, es necesario verificar si el cliente está funcionando correctamente.

Si la velocidad de recepción de datos del cliente es **continuamente** inferior a la velocidad de envío del servidor, es necesario considerar la optimización del flujo de trabajo o mejorar el rendimiento del cliente. Si la causa es la congestión de ancho de banda, se puede considerar aumentar el ancho de banda del servidor.
