# ¿Puede Workerman recibir y procesar datos de un servidor remoto como cliente?

Sí, puede usar AsyncTcpConnection para iniciar una conexión asincrónica y permitir que Workerman actúe como cliente para interactuar con el servidor.

Por ejemplo, aquí se muestran algunos ejemplos:

1. [Workerman como cliente WebSocket](as-wss-client.md)
2. [Workerman como agente MySQL](../async-tcp-connection/connect.md)
3. [Workerman como cliente HTTP](../async-tcp-connection/construct.md)
4. [Workerman como agente HTTP](https://github.com/walkor/php-http-proxy)
5. [Workerman como agente SOCKS5](https://github.com/walkor/php-socks5)
