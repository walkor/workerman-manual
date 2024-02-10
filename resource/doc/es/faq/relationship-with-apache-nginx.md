# Relación con Apache y Nginx
**Pregunta:**

¿Cuál es la relación entre Workerman y Apache/nginx/php-fpm? ¿Hay conflictos entre Workerman y Apache/nginx/php-fpm?

**Respuesta:**

Workerman no tiene ninguna relación con Apache/nginx/php-fpm y su funcionamiento no depende de ellos. Ambos son contenedores independientes que no interfieren entre sí ni entran en conflicto (si no están escuchando en el mismo puerto).

Workerman es un marco de servidor de socket general que soporta conexiones persistentes y varios protocolos como HTTP, WebSocket y protocolos personalizados. Por otro lado, Apache/nginx/php-fpm generalmente se utiliza para desarrollar proyectos web que utilizan el protocolo HTTP.

Si un servidor ya tiene desplegados Apache/nginx/php-fpm, desplegar Workerman no afectará su funcionamiento.
