# Protocolos suportados pelo WorkerMan

O WorkerMan oferece suporte a uma variedade de protocolos por meio de interfaces que seguem o `ConnectionInterface` (consulte a seção de Protocolos de Comunicação Personalizados).

Para maior comodidade dos desenvolvedores, o WorkerMan fornece suporte para os protocolos HTTP, WebSocket, bem como um protocolo de texto muito simples que pode ser usado para transmissão de dados binários. Os desenvolvedores podem usar esses protocolos diretamente, sem a necessidade de desenvolvimento adicional. Se nenhum desses protocolos atender às necessidades, os desenvolvedores podem implementar seu próprio protocolo conforme a seção de Protocolos Personalizados.

Os desenvolvedores também podem optar por usar diretamente os protocolos TCP ou UDP.

Exemplos de uso de protocolos:
```php
// protocolo HTTP
$worker1 = new Worker('http://0.0.0.0:1221');
// protocolo WebSocket
$worker2 = new Worker('websocket://0.0.0.0:1222');
// protocolo de texto (protocolo Telnet)
$worker3 = new Worker('text://0.0.0.0:1223');
// protocolo de quadro (usado para transmissão de dados binários)
$worker3 = new Worker('frame://0.0.0.0:1223');
// protocolo de transmissão direta TCP
$worker4 = new Worker('tcp://0.0.0.0:1224');
// protocolo de transmissão direta UDP
$worker5 = new Worker('udp://0.0.0.0:1225');
```
