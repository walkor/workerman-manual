# Características do WorkerMan

### 1. Desenvolvimento puro em PHP
Aplicativos desenvolvidos com WorkerMan podem ser executados de forma independente, sem depender de contêineres como php-fpm, apache ou nginx. Isso torna conveniente para os desenvolvedores de PHP desenvolver, implantar e depurar aplicativos.

### 2. Suporte a multiprocessamento em PHP
Para aproveitar ao máximo o desempenho de vários CPUs do servidor, o WorkerMan suporta por padrão múltiplos processos e tarefas. O WorkerMan inicia um processo principal e vários processos filhos para fornecer serviços, onde o processo principal supervisiona os processos filhos. Cada processo filho é responsável por ouvir conexões de rede, receber, enviar e processar dados. Devido ao modelo de processos simples, o WorkerMan é mais estável e eficiente.

### 3. Suporte a TCP e UDP
O WorkerMan suporta dois protocolos de transporte, TCP e UDP, e basta modificar um atributo para alternar entre eles, sem a necessidade de alterar o código de negócio.

### 4. Suporte a conexões persistentes
Muitas vezes, aplicativos PHP precisam manter conexões persistentes com o cliente, como em salas de bate-papo, jogos, entre outros. No entanto, os contêineres tradicionais de PHP (apache, nginx, php-fpm) têm dificuldade em fazer isso. Usando o WorkerMan, desde que o negócio do servidor não encerre ativamente a conexão, é possível utilizar conexões persistentes em PHP. Um único processo do WorkerMan pode suportar dezenas de milhares de conexões simultâneas, e múltiplos processos podem suportar dezenas de milhões ou até mesmo centenas de milhões de conexões simultâneas.

### 5. Suporte a vários protocolos de aplicação
A interface do WorkerMan suporta vários protocolos de aplicação, incluindo protocolos personalizados. Alterar o protocolo no WorkerMan é muito simples, por configuração, sem alterar o código de negócio. É até possível abrir várias portas com diferentes protocolos para atender às necessidades de diferentes clientes.

### 6. Suporte a alta concorrência
O WorkerMan suporta a biblioteca de eventos Libevent (requer a extensão event instalada). O desempenho é excelente com o Event em conexões persistentes de alta concorrência. Se a extensão Event não estiver instalada, o WorkerMan usará chamadas de sistema relacionadas ao Select internas do PHP, mantendo um desempenho igualmente forte.

### 7. Suporte a reinício suave do serviço
Quando há necessidade de reiniciar o serviço (por exemplo, durante a publicação de uma nova versão), não queremos que os processos que estão lidando com as solicitações dos usuários sejam encerrados imediatamente, nem queremos que a comunicação com o cliente falhe no momento do reinício. O WorkerMan oferece a capacidade de reinício suave, garantindo uma atualização tranquila do serviço sem afetar o uso do cliente.

### 8. Suporte à detecção e recarga automática de arquivos
Durante o desenvolvimento, esperamos que as alterações no código entrem em vigor imediatamente para poder visualizar os resultados. O WorkerMan oferece o [componente FileMonitor](../components/file-monitor.md), que recarrega automaticamente quando um arquivo é atualizado, para carregar os novos arquivos e torná-los efetivos.

### 9. Suporte à execução de processos filhos por usuário especificado
Como os processos filhos são os responsáveis reais por lidar com as solicitações dos usuários, por questões de segurança, os processos filhos não devem ter permissões muito altas. Portanto, o WorkerMan suporta a configuração do usuário sob o qual os processos filhos são executados, aumentando a segurança do servidor.

### 10. Suporte à manutenção permanente de objetos ou recursos
Durante a execução do WorkerMan, um arquivo PHP só é lido e interpretado uma vez, permanecendo em memória. Isso garante que a declaração de classes e funções, o ambiente de execução do PHP e a tabela de símbolos não sejam criados e destruídos repetidamente, o que é totalmente diferente do mecanismo do PHP em execução nos contêineres da Web.

No WorkerMan, membros estáticos ou variáveis globais mantidos sem serem destruídos ativamente em um processo são permanentes durante todo o ciclo de vida do processo, permitindo que todos os pedidos desse processo reutilizem objetos ou conexões de recursos, por exemplo. Ao inicializar a conexão com o banco de dados apenas uma vez em um único processo, todas as solicitações desse processo podem reutilizar essa conexão, evitando o processo de conexão frequente ao banco de dados, incluindo o processo de estabelecimento de conexão TCP, autenticação de banco de dados, e desconexão seguindo o processo de encerramento de conexão TCP, o que melhora significativamente a eficiência do aplicativo.

### 11. Alto desempenho
Devido ao fato de que o arquivo PHP é lido do disco, interpretado e permanece na memória após a primeira execução, as operações de E/S de disco e muitos processos demorados, como inicialização de solicitação e criação de ambiente de execução, análise léxica, análise sintática, compilação de opcode, encerramento de solicitação, são consideravelmente reduzidos. Além disso, o WorkerMan não depende de contêineres como nginx, apache, reduzindo as despesas de comunicação com o PHP nos contêineres. O aspecto mais importante é que os recursos podem ser mantidos permanentemente, sem a necessidade de inicializar a conexão com o banco de dados sempre que necessário, o que torna o desenvolvimento de aplicativos com o WorkerMan muito eficiente.

### 12. Suporte ao HHVM
O WorkerMan suporta a execução no HHVM (HipHop Virtual Machine), proporcionando um grande aumento de desempenho do PHP. Especialmente em operações intensivas em CPU, o desempenho é excepcional. Em testes de estresse reais e comparações, em condições de baixa carga de trabalho, o WorkerMan em execução no HHVM aumentou o throughput de rede em cerca de 30-80% em comparação com a execução no Zend PHP 5.6.

### 13. Suporte a implantação distribuída

### 14. Suporte à demonização

### 15. Suporte a múltiplas portas de escuta

### 16. Suporte à redirecionamento de entrada e saída padrão
