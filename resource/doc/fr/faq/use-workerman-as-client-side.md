workerman peut-il recevoir et traiter des données provenant d'un serveur distant en tant que client?

Il est possible d'utiliser AsyncTcpConnection pour établir une connexion asynchrone, permettant à workerman d'interagir en tant que client avec le serveur.

Par exemple, ci-dessous sont des exemples :

1. [workerman agissant en tant que client websocket](as-wss-client.md)

2. [workerman agissant en tant qu'agent mysql](../async-tcp-connection/connect.md)

3. [workerman agissant en tant que client http](../async-tcp-connection/construct.md)

4. [workerman agissant en tant qu'agent http](https://github.com/walkor/php-http-proxy)

5. [workerman agissant en tant qu'agent socks5](https://github.com/walkor/php-socks5)
