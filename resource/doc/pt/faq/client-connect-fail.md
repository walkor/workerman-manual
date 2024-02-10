# Razões para falha na conexão do cliente

Geralmente, a falha na conexão do cliente pode resultar em dois tipos de erros, `connection refuse` (recusa de conexão) e `connection timeout` (tempo de conexão esgotado).

## Connection refuse (recusa de conexão)

Geralmente é causado por:
1. A porta à qual o cliente está se conectando está errada
2. O nome de domínio ou IP ao qual o cliente está se conectando está errado
3. Se o cliente estiver usando um nome de domínio para se conectar, o domínio pode estar apontando para o endereço IP errado
4. O servidor está utilizando um CDN ou outro proxy de aceleração, o que faz com que o IP de conexão real seja diferente do esperado
5. O servidor não está em execução ou a porta não está sendo ouvida
6. Está sendo utilizado software de proxy de rede
7. O IP de escuta do servidor e o endereço de acesso não estão no mesmo intervalo de endereços. Por exemplo, se o servidor estiver ouvindo em 127.0.0.1, o cliente só poderá se conectar através de 127.0.0.1 e não através do IP da LAN ou da Internet. É recomendável configurar o endereço de escuta como 0.0.0.0, desta forma a máquina local, a LAN e a Internet podem se conectar.

## Connection timeout (tempo de conexão esgotado)

Geralmente é causado por:
1. O firewall do servidor está bloqueando a conexão; pode-se tentar temporariamente desativar o firewall
2. Se for um servidor em nuvem, o grupo de segurança pode estar bloqueando a conexão e é necessário abrir a porta correspondente no painel de administração
3. Se estiver utilizando um painel como o Baota, é necessário abrir a porta correspondente no Baota
4. O servidor não existe ou não foi inicializado
5. Se o cliente estiver usando um nome de domínio para se conectar, o domínio pode estar apontando para o endereço IP errado
6. O IP acessado pelo cliente é um IP interno do servidor e o cliente e o servidor não estão na mesma rede local

## Cannot assign requested address (Não é possível atribuir o endereço solicitado)

**Como cliente**, ao iniciar uma conexão, é necessário reservar uma porta temporária local. Por padrão, um servidor tem aproximadamente 20-30 mil portas temporárias disponíveis. Se o número de conexões iniciadas para um servidor específico exceder esse valor, não será possível atribuir uma porta disponível, resultando neste erro.
Pode-se aumentar o número de portas temporárias localmente alterando o parâmetro do kernel `/etc/sysctl.conf` `net.ipv4.ip_local_port_range`, por exemplo, configurando-o como `10000 65535` (a faixa de portas locais é configurada como 10000 a 65535, aumentando o número de portas locais para 55.535), e então executando `sysctl -p` para que tenha efeito.
Além disso, as conexões que são encerradas passam para o estado TIME_WAIT e ainda ocupam a porta local correspondente por um certo período de tempo. Portanto, iniciar uma grande quantidade de conexões curtas em um curto período de tempo (mais de 20-30 mil) também resultará em `Cannot assign requested address`. Se este for o caso, pode-se resolver o problema configurando o kernel para descartar rapidamente as conexões TIME_WAIT, consulte [Otimização do kernel](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html).

> **Nota**
> A limitação do número de portas locais se aplica apenas ao cliente. O servidor não tem restrição de portas locais; contanto que os recursos sejam suficientes, o servidor pode manter uma quantidade ilimitada de conexões.

## Outros erros
Se o erro não for `connection refuse` ou `connection timeout`, geralmente é devido a:
**1. O protocolo de comunicação utilizado pelo cliente não é compatível com o do servidor.**
Por exemplo, se o servidor estiver usando o protocolo de comunicação HTTP, o cliente não poderá se conectar usando o protocolo de comunicação WebSocket. Caso o cliente esteja se conectando utilizando o protocolo WebSocket, o servidor também deve estar utilizando o protocolo WebSocket. Se o servidor estiver utilizando o protocolo HTTP, o cliente deve utilizar o protocolo HTTP.

O princípio é semelhante a precisar falar inglês para se comunicar com alguém do Reino Unido ou falar japonês para se comunicar com alguém do Japão. Neste contexto, a linguagem é comparável ao protocolo de comunicação, onde ambas as partes (cliente e servidor) precisam utilizar a mesma linguagem para se comunicar; caso contrário, a comunicação será impossível.

**Erros comuns causados pela incompatibilidade do protocolo de comunicação:**
> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: Unexpected response code: xxx

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: net::ERR_INVALID_HTTP_RESPONSE

**Solução:**
Como pode ser observado nos dois erros acima, o cliente está utilizando a conexão ws (protocolo WebSocket). Portanto, o servidor também deve estar utilizando o protocolo WebSocket para que a comunicação seja possível. Ao ouvir no servidor, o código deve especificar o protocolo WebSocket, como mostrado nos exemplos a seguir.

Se for um gatewayWorker, o código para ouvir será semelhante a:
```php
// Protocolo WebSocket, permitindo que o cliente se conecte através de ws://...; xxxx é a porta e não precisa ser modificada
$gateway = new Gateway('websocket://0.0.0.0:xxxx');
```
Se for um Workerman, então o código será:
```php
// Protocolo WebSocket, permitindo que o cliente se conecte através de ws://...; xxxx é a porta e não precisa ser modificada
$worker = new Worker('websocket://0.0.0.0:xxxx');
```
