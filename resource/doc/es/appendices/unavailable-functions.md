# Funciones no compatibles

Función/Declaración no compatible | Alternativa | Descripción
----|------|----
pcntl_fork | Configurar el número de procesos de antemano | 
php://input | [`$request->rawBody()`](http/request.md) | Utilizado para que las aplicaciones bajo el protocolo HTTP obtengan los datos originales POST
exit | return | Usar exit provocará la terminación del proceso, si se desea retornar, usar directamente la declaración return
die | return | Usar die provocará la terminación del proceso, si se desea retornar, usar directamente la declaración return
Funciones relacionadas con header cookie session | Consultar la clase [`$request`](http/request.md) y [`$response`](http/response.md) | 
set_time_limit | No se recomienda | Solo puede ser fijado en 0, de lo contrario, provocará la terminación del proceso de Workerman después de cierto tiempo
