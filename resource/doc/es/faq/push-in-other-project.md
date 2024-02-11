# Utilizar Workerman para enviar datos a clientes en otros proyectos

**Pregunta:**

Tengo un proyecto web común y corriente, y quiero llamar a la API de Workerman en este proyecto para enviar datos a los clientes.

**Respuesta:**

**Para un proyecto basado en Workerman, puedes consultar los siguientes enlaces:**

- [Ejemplo de envío con el componente de Canal](../components/channel-examples.md) (compatible con múltiples procesos/cluster de servidores, requiere descargar el componente de Canal)

- [Envío basado en Worker](https://www.workerman.net/q/508) (solo un proceso, el más sencillo)

**Para un proyecto basado en webman, consulta el siguiente enlace:**
- [Plugin de envío de webman](https://www.workerman.net/plugin/2)

**Para un proyecto basado en GatewayWorker, consulta el siguiente enlace:**

- [Envío a través de GatewayWorker en otros proyectos](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html) (compatible con múltiples procesos/cluster de servidores, compatible con grupos, multicast y envío individual)

**Para un proyecto basado en PHPSocket.IO, consulta el siguiente enlace:**

- [Envío de mensajes web](https://www.workerman.net/web-sender) (por defecto un solo proceso, basado en socket.io, mejor compatibilidad con navegadores)
