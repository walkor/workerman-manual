# Rastreando chamadas de sistema

Quando queremos saber o que um processo está fazendo, podemos rastrear todas as chamadas de sistema de um processo usando o comando ```strace```.

1. Ao executar ```php start.php status```, podemos ver as informações relacionadas aos processos do workerman, como mostrado abaixo:

```plaintext
Olá admin
---------------------------------------STATUS GLOBAL--------------------------------------------
Versão do Workerman: 3.0.1
Hora de início: 2014-08-12 17:42:04   0 dias de execução 1 horas
carga média: 3.34, 3.59, 3.67
1 usuário       8 workers       14 processos
nome_do_worker       status_de_saida     contagem_de_saída
BusinessWorker    0                0
ChatWeb           0                0
FileMonitor       0                0
Gateway           0                0
Monitor           0                0
StatisticProvider 0                0
StatisticWeb      0                0
StatisticWorker   0                0
---------------------------------------STATUS DO PROCESSO-------------------------------------------
pid	memória      ouvindo        carimbo de data/hora  nome_do_worker       requisição_total  erro_de_pacote thunder_herd fechar_cliente falha_de_envio lançar_exceção suc/total
10352	1.5M    tcp://0.0.0.0:55151  1407836524 ChatWeb           12             0          0            2            0         0               100%
10354	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          0            0            0         0               100%
10355	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           0              0          1            0            0         0               100%
10365	1.25M   tcp://0.0.0.0:55757  1407836524 StatisticWeb      0              0          0            0            0         0               100%
10358	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          2            0            0         0               100%
10364	1.25M   tcp://0.0.0.0:55858  1407836524 StatisticProvider 0              0          0            0            0         0               100%
10356	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          2            0            0         0               100%
10366	1.25M   udp://0.0.0.0:55656  1407836524 StatisticWorker   55             0          0            0            0         0               100%
10349	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    5              0          0            0            0         0               100%
10350	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    0              0          0            0            0         0               100%
10351	1.5M    tcp://127.0.0.1:7373 1407836524 BusinessWorker    5              0          0            0            0         0               100%
10348	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    2              0          0            0            0         0               100%
```

2. Por exemplo, se quisermos saber o que o processo gateway com o pid 10354 está fazendo, podemos executar o comando ```strace -p 10354``` (talvez seja necessário permissão de root), semelhante ao exemplo abaixo:

```plaintext
sudo strace -p 10354
Processo 10354 conectado - interrompa para sair
clock_gettime(CLOCK_MONOTONIC, {118627, 242986712}) = 0
gettimeofday({1407840609, 102439}, NULL) = 0
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Chamada de sistema interrompida)
--- SIGUSR2 (Sinal definido pelo usuário 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (máscara agora [])
clock_gettime(CLOCK_MONOTONIC, {118627, 699623319}) = 0
gettimeofday({1407840609, 559092}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118627, 699810499}) = 0
gettimeofday({1407840609, 559277}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Recurso temporariamente indisponível)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Chamada de sistema interrompida)
--- SIGUSR2 (Sinal definido pelo usuário 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (máscara agora [])
clock_gettime(CLOCK_MONOTONIC, {118628, 699497204}) = 0
gettimeofday({1407840610, 558937}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118628, 699588603}) = 0
gettimeofday({1407840610, 559023}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Recurso temporariamente indisponível)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Chamada de sistema interrompida)
--- SIGUSR2 (Sinal definido pelo usuário 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (máscara agora [])
```

3. Cada linha representa uma chamada de sistema. A partir dessas informações, podemos ver facilmente o que o processo está fazendo e identificar onde ele está preso, seja em uma conexão ou na leitura de dados de rede, etc.
