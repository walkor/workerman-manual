# Algumas questões importantes que os desenvolvedores do Workerman precisam saber

**1. Restrições no ambiente Windows**

No sistema Windows, um único processo do Workerman suporta apenas 200+ conexões.
No sistema Windows, não é possível usar o parâmetro de contagem para configurar vários processos.
No sistema Windows, comandos como status, stop, reload, restart não podem ser utilizados.
No sistema Windows, não é possível utilizar processos em segundo plano; o serviço para quando a janela do cmd é fechada.
No sistema Windows, não é possível inicializar várias escutas em um único arquivo.
O Linux não tem essas restrições; portanto, é recomendável usar o sistema Linux em ambientes de produção e o sistema Windows em ambientes de desenvolvimento.

**2. O Workerman não depende do Apache ou Nginx**

O Workerman é em si um container semelhante ao Apache/nginx e, desde que o ambiente PHP esteja correto, o Workerman pode ser executado.

**3. O Workerman é iniciado via linha de comando**

A forma de inicialização é semelhante ao Apache, utilizando um comando de inicialização (normalmente, não é possível usar o Workerman em hospedagens de sites). A interface de inicialização é semelhante à abaixo:
![](image/screenshot_1495622774534.png)

**4. Conexões longas devem incluir batimento cardíaco**

É imprescindível incluir batimento cardíaco em conexões longas. É fundamental repetir três vezes: conexões longas devem incluir batimento cardíaco. Se não houver comunicação por um longo período, o nó roteador irá limpar a conexão, resultando no encerramento da mesma.
[Explicação do batimento cardíaco do Workerman](faq/heartbeat.md), [Explicação do batimento cardíaco do GatewayWorker](https://www.workerman.net/doc/gateway-worker/heartbeat.html)

**5. O protocolo do cliente e do servidor deve corresponder para a comunicação**

Este é um problema muito comum para os desenvolvedores. Por exemplo, se o cliente está usando o protocolo WebSocket, o servidor também deve usar o protocolo WebSocket (o servidor `new Worker('websocket://0.0.0.0...')`) para que a conexão ocorra e a comunicação seja possível. 
Não tente acessar a porta do protocolo WebSocket na barra de endereços do navegador, nem tente usar o protocolo WebSocket para acessar a porta do protocolo TCP nu; os protocolos devem corresponder.

Esta lógica é similar à necessidade de usar inglês para se comunicar com pessoas britânicas. Se deseja se comunicar com japoneses, é necessário usar japonês. Os protocolos aqui são similares às línguas; ambas as partes (cliente e servidor) devem utilizar a mesma língua para se comunicar, caso contrário, a comunicação não será possível.

**6. Possíveis motivos para falha na conexão**

É muito comum que, ao começar a usar o Workerman, ocorram falhas na conexão do cliente com o servidor. As razões geralmente são as seguintes:
1. O firewall do servidor (incluindo o grupo de segurança do servidor em nuvem) bloqueia a conexão (probabilidade de 50%).
2. O cliente e o servidor usam protocolos diferentes (probabilidade de 30%).
3. O endereço IP ou a porta estão incorretos (probabilidade de 15%).
4. O servidor não está iniciado.

**7. Não utilize declarações exit die sleep**

A execução de declarações exit die levará à saída do processo e exibirá um erro "WORKER EXIT UNEXPECTED". Obviamente, quando um processo é encerrado, imediatamente será iniciado um novo processo para continuar o serviço. Se for necessário sair, pode-se utilizar o comando return. A declaração sleep fará com que o processo entre em um estado de repouso, durante o qual não será realizada nenhuma operação de negócio, e o framework também deixará de funcionar, impossibilitando o processamento de todas as solicitações de clientes nesse processo.

**8. Não utilize a função pcntl_fork**

`pcntl_fork` é utilizada para criar dinamicamente novos processos. Se for utilizada dentro do código de negócios, pode criar processos órfãos não recuperáveis, resultando em exceções nos negócios. A utilização de `pcntl_fork` nos negócios também afetará o processamento de eventos como conexões, mensagens, fechamento de conexões e temporizadores, resultando em exceções imprevistas.

**9. Não inclua loops infinitos no código de negócios**

O código de negócios não deve incluir loops infinitos, pois isso impedirá que o controle seja devolvido ao framework do Workerman, impossibilitando o recebimento e processamento de outras mensagens de clientes.

**10. É necessário reiniciar o Workerman após a atualização do código**

O Workerman é um framework com memória residencial, portanto, é necessário reiniciar o Workerman para ver os efeitos do novo código.

**11. Aplicações com conexão longa devem usar o framework GatewayWorker**

Muitos desenvolvedores utilizam o Workerman para desenvolver aplicações com conexão longa, como comunicação instantânea e Internet das Coisas. Para essas aplicações com conexão longa, é recomendável utilizar diretamente o framework GatewayWorker, que é uma camada adicional de encapsulamento sobre o Workerman, tornando o backend das aplicações com conexão longa mais simples e fácil de usar.

**12. Suporta maior concorrência**
Se o número de conexões simultâneas ultrapassar 1000, é imperativo [otimizar o kernel do Linux](appendices/kernel-optimization.md) e [instalar a extensão event](appendices/install-extension.md).
