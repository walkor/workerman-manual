# ipv6

Pergunta: Como permitir que os clientes acessem tanto através de endereços ipv4 quanto de endereços ipv6?

Resposta: Ao inicializar o contêiner, basta configurar o endereço de escuta como ```[::]```.

Por exemplo:
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```
