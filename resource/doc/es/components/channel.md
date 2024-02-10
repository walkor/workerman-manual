# Componente de comunicación distribuida Channel

**``` (Requiere Workerman versión>=3.3.0) ```**

Dirección del código fuente: https://github.com/walkor/Channel

Channel es un componente de comunicación distribuida utilizado para la comunicación entre procesos o servidores.

## Características
1. Basado en el modelo de publicación/suscripción
2. IO no bloqueante

## Principio
Channel incluye el servidor Channel/Server y el cliente Channel/Client.

El cliente Channel/Client se conecta al servidor Channel/Server mediante la interfaz connect y mantiene una conexión de larga duración.

El cliente Channel/Client utiliza la interfaz on para informar al servidor Channel/Server sobre los eventos que le interesan y registra funciones de devolución de llamada de eventos (la devolución ocurre en el proceso del cliente Channel/Client).

El cliente Channel/Client utiliza la interfaz publish para publicar un evento y los datos relacionados al servidor Channel/Server.

El servidor Channel/Server recibe el evento y los datos y los distribuye a los clientes Channel/Client que están interesados en ese evento.

El cliente Channel/Client recibe el evento y los datos, y activa las devoluciones de llamada establecidas con la interfaz on.

El cliente Channel/Client solo recibirá eventos en los que esté interesado y activará las devoluciones de llamada.

## Instalación
`composer require workerman/channel`

## Nota
Channel solo puede utilizarse en un entorno de workerman. No se puede utilizar en un entorno de php-fpm.
