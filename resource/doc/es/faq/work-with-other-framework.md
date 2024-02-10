# Cómo integrarse con otros frameworks

**Pregunta:**

¿Cómo se integra con otros frameworks MVC (como thinkPHP, Yii, etc.)?

**Respuesta:**

![workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

Se **recomienda** integrar con otros frameworks MVC de la siguiente manera (usando ThinkPHP como ejemplo):

1. ThinkPHP y Workerman son sistemas independientes, desplegables de forma independiente (pueden ser desplegados en diferentes servidores) y no interfieren entre sí.

2. ThinkPHP proporciona páginas web que se renderizan y muestran en el navegador a través del protocolo HTTP.

3. El JavaScript proporcionado por ThinkPHP inicia una conexión websocket para conectarse con Workerman.

4. Después de establecer la conexión, se envía un paquete de datos a Workerman (que incluye un nombre de usuario, contraseña o algún tipo de token) para validar la conexión websocket del usuario.

5. Solo cuando ThinkPHP necesite enviar datos al navegador, se llama a la interfaz de socket de Workerman para enviar los datos.

6. El resto de las solicitudes siguen siendo manejadas de acuerdo con el método HTTP original de ThinkPHP.

**En resumen:**

Se utiliza Workerman como un canal para enviar datos al navegador, y solo se llama a la interfaz de Workerman para enviar datos cuando sea necesario enviar datos al navegador. La lógica del negocio se completa completamente en ThinkPHP.

Para obtener información sobre cómo ThinkPHP llama a la interfaz de socket de Workerman para enviar datos, consulte la sección [Preguntas frecuentes - Enviar en otros proyectos](push-in-other-project.md).

**ThinkPHP ya es compatible con Workerman, consulte el [manual de ThinkPHP5](https://www.kancloud.cn/manual/thinkphp5/235128) para más información.**
