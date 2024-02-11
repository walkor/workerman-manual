# Verificar o estado de execução

Executar ```php start.php status```
pode verificar o estado de execução do Workerman, semelhante ao seguinte:

```plaintext
----------------------------------------------ESTADO GLOBAL----------------------------------------------------
Versão do Workerman: 3.5.13          Versão do PHP: 5.5.9-1ubuntu4.24
hora de início: 2018-02-03 11:48:20   execução de 112 dias 2 horas   
carga média: 0, 0, 0            loop de eventos:\Workerman\Events\Event
4 trabalhadores       11 processos
nome_trabalhador        status_saída      contagem_saída
ChatBusinessWorker 0                0
ChatGateway        0                0
Register           0                0
WebServer          0                0
----------------------------------------------ESTADO DO PROCESSO---------------------------------------------------
pid	memória  escutando                nome_trabalhador        conexões falha_envio timers  total_requisições qps    status
18306	2.25M   nenhum                     ChatBusinessWorker 5           0         0       11            0      [ocioso]
18307	2.25M   nenhum                     ChatBusinessWorker 5           0         0       8             0      [ocioso]
18308	2.25M   nenhum                     ChatBusinessWorker 5           0         0       3             0      [ocioso]
18309	2.25M   nenhum                     ChatBusinessWorker 5           0         0       14            0      [ocioso]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [ocioso]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [ocioso]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [ocioso]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [ocioso]
18314	1.75M   text://0.0.0.0:1236      Register           8           0         0       8             0      [ocioso]
18315	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [ocioso]
18316	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [ocioso]
----------------------------------------------ESTADO DO PROCESSO---------------------------------------------------
Resumo	18M     -                        -                  54          0         4       138           0      [Resumo]
```

## Explicação

### ESTADO GLOBAL

Aqui, podemos ver

A versão do Workerman```version:3.5.13```

Tempo de início ```2018-02-03 11:48:20```, com execução de ```112 dias 2 horas ```

Carga do servidor ```load average: 0, 0, 0```, representando a carga média do sistema nos últimos 1, 5 e 15 minutos, respectivamente.

Utilização da biblioteca de eventos de E/S, ```event-loop:\Workerman\Events\Event```

 ```4 trabalhadores``` (3 tipos de processos, incluindo processos ChatGateway, ChatBusinessWorker, Register e WebServer)

 ```11 processos``` (um total de 11 processos)

 ```nome_trabalhador``` (nome do processo de trabalhador)

 ```status_saída``` (código de saída do processo de trabalhador)

 ```contagem_saída``` (contagem de vezes em que o código de saída foi registrado)

Normalmente, um status_saída de 0 indica uma saída normal. Se for outro valor, o processo saiu de forma anormal, e uma mensagem de erro como ```WORKER EXIT UNEXPECTED``` será registrada no arquivo especificado por [Worker::logFile](worker/log-file.md). 

**Códigos de status_saída comuns e seus significados são os seguintes:**

* 0: Representa uma saída normal, ocorrendo após um reload suave. É importante observar que chamadas de exit ou die no código de negócios também gerarão um código de saída 0 e uma mensagem de erro ```WORKER EXIT UNEXPECTED```, pois o Workerman não permite que o código de negócios use essas declarações.
* 9: Indica que o processo foi morto pelo sinal SIGKILL. Este código de saída geralmente ocorre durante uma parada ou reload suave, devido à falha do processo filho em responder ao sinal de reload do processo pai dentro do tempo especificado (por exemplo, devido a bloqueios prolongados, como em chamadas longas para mysql, curl, ou loops de negócios infinitos, entre outros). É importante observar que o uso do comando kill em um terminal Linux para enviar o sinal SIGKILL para um processo filho também gerará este código de saída.
* 11: Refere-se a um despejo de núcleo do PHP, geralmente causado pelo uso de extensões instáveis. O recomendado é comentar a extensão correspondente no arquivo php.ini; em casos raros, pode se tratar de um bug no PHP, requerendo então a atualização do PHP.
* 65280: Ocorre quando o código de negócios tem um erro fatal, como a chamada para uma função inexistente ou erro de sintaxe. As informações específicas do erro são registradas no arquivo especificado por [Worker::logFile](worker/log-file.md) e, se especificado, no arquivo [error_log](https://php.net/manual/pt_BR/errorfunc.configuration.php#ini.error-log) do php.ini.
* 64000: Ocorre quando o código de negócios lança uma exceção que não é tratada, resultando no encerramento do processo. Se o Workerman estiver sendo executado no modo de depuração, a pilha de chamadas da exceção será impressa no terminal. No modo daemon, a pilha de chamadas da exceção será registrada no arquivo especificado por [Worker::stdoutFile](worker/stdout-file.md).

### ESTADO DO PROCESSO

pid: PID do processo

memória: A quantidade atual de memória usada pelo processo (não incluindo a memória usada pelo próprio arquivo executável do PHP)

escutando: Protocolo de transporte e enfileiramento IP. Se nenhum porto estiver sendo escutado, será exibido como nenhum. Consulte [Construtor da Classe Worker](worker/construct.md).

nome_trabalhador: O nome do serviço executado pelo processo. Ver [propriedade de nome da classe Worker](worker/name.md).

conexões: O número atual de instâncias de conexão TCP que o processo tem. As instâncias de conexão incluem instâncias de TcpConnection e AsyncTcpConnection. Este valor é uma contagem em tempo real e não cumulativo. Observação: Se a contagem ainda não diminuiu após o fechamento de uma instância de conexão, pode ser devido ao código de negócios mantendo o objeto $connection vivo, impedindo a destruição da instância de conexão.

total_requisições: O total de requisições que o processo recebeu desde o seu início. Essas requisições incluem tanto as recebidas do cliente quanto as solicitações de comunicação interna do Workerman, como no caso da arquitetura GatewayWorker, em que Gateway e BusinessWorker trocam solicitações de comunicação.

falha_envio: O número de falhas no envio de dados do processo para o cliente. Se este valor for diferente de zero, geralmente significa que a conexão do cliente foi encerrada. Consulte [falhas de envio de status](../faq/about-send-fail.md). Este valor é cumulativo.

timers: O número de temporizadores ativos (excluindo temporizadores excluídos e temporizadores de execução única) no processo. Observação: Este recurso está disponível apenas na versão do Workerman 3.4.7 ou superior.

qps: A quantidade de solicitações de rede que o processo recebe por segundo. Observação: Este recurso está disponível apenas na versão do Workerman 3.5.2 ou superior. Este valor é em tempo real, não cumulativo.

status: O atual estado do processo. Se for ocioso, significa que está inativo; se estiver ocupado, significa que está ocupado. Observação: A presença prolongada de um processo ocupado pode indicar bloqueios de negócios ou loops infinitos de negócios. Consulte a seção [depuração de processos ocupados](busy-process.md). Observação: Este recurso está disponível apenas na versão do Workerman 3.5.0 ou superior.

## Princípio

Após a execução do script status, o processo principal envia um sinal SIGUSR2 a todos os processos de trabalhador. Em seguida, o script de status entra em um curto período de suspensão para aguardar que todos os processos de trabalhador relatem seus estados. Durante este tempo, os processos de trabalhador ociosos imediatamente escrevem seus estados (como número de conexões, número de requisições, etc.) em arquivos específicos no disco, enquanto os processos ocupados aguardam a conclusão do processamento da lógica de negócios antes de atualizar seus estados. Após a breve suspensão, o script de status lê os arquivos de estado no disco e exibe os resultados no console.

## Observação

Ao executar o status, é possível que alguns processos apareçam como ocupados, devido ao bloqueio de processamento de negócios (como bloqueios prolongados em chamadas para curl ou bancos de dados, ou execução de loops de longa duração), impossibilitando a atualização do estado e, consequentemente, sendo exibido como ocupado. Nesse caso, é necessário verificar o código de negócios, identificando onde ocorre o bloqueio e avaliando se o tempo de bloqueio está dentro do esperado. Caso contrário, é necessário seguir o procedimento de [depuração de processos ocupados](busy-process.md) para investigar o código de negócios.
