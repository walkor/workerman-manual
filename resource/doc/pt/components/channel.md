# Componente de Comunicação Distribuída Channel

**``` (Requer Workerman versão >= 3.3.0) ```**

Repositório: https://github.com/walkor/Channel

Channel é um componente de comunicação distribuída usado para comunicação entre processos ou entre servidores.

## Características
1. Baseado no modelo de publicação e assinatura
2. E/S não bloqueante

## Princípio
O Channel inclui o servidor Channel/Server e o cliente Channel/Client.
O Channel/Client conecta-se ao Channel/Server através da interface connect e mantém uma conexão de longa duração.
O Channel/Client informa ao Channel/Server quais eventos está interessado através da interface on e registra funções de retorno de eventos (callback ocorre no processo Channel/Client).
O Channel/Client publica eventos e seus dados no Channel/Server através da interface publish.
O Channel/Server recebe o evento e os dados e os distribui para o Channel/Client interessado nesse evento.
O Channel/Client recebe o evento e os dados e aciona o retorno de eventos configurado na interface on.
O Channel/Client só receberá eventos pelos quais está interessado e acionará os callbacks.

## Instalação
`composer require workerman/channel`

## Observação
O Channel só pode ser usado no ambiente do Workerman e não pode ser utilizado no ambiente php-fpm.
