# Falha ao Parar

## Fenômeno:
Executando ```php start.php stop``` exibe a mensagem ```stop fail```.

### Primeira Possibilidade
Assumindo que o Workerman foi iniciado em modo de depuração, o desenvolvedor pressionou ```ctrl z``` no terminal, enviando o sinal ```SIGSTOP``` para o Workerman, fazendo com que ele entre em segundo plano e pausado, tornando-se incapaz de responder ao comando de parada (sinal ```SIGINT```).

**Solução:**
No terminal onde o Workerman está em execução, digite ```fg``` (para enviar o sinal ```SIGCONT```) e pressione Enter, trazendo o Workerman de volta para o primeiro plano. Em seguida, pressione ```ctrl c``` (para enviar o sinal ```SIGINT```) e parar o Workerman.

Se não for possível parar, tente executar os dois comandos a seguir:
```sh
killall -9 php
```
```sh
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

### Segunda Possibilidade
O usuário que está tentando parar o Workerman é diferente do usuário que o iniciou, ou seja, o usuário que tenta parar não tem permissão para fazê-lo.

**Solução:**
Mude para o usuário que iniciou o Workerman ou use um usuário com permissões mais elevadas para parar o Workerman.

### Terceira Possibilidade
O arquivo de PID do processo principal do Workerman foi excluído, resultando no script incapaz de encontrar o processo de PID e falhando ao parar.

**Solução:**
Mova o arquivo de PID para um local seguro, consulte o manual [Worker::$pidFile](../worker/pid-file.md).

### Quarta Possibilidade
O arquivo de PID do processo principal do Workerman não corresponde ao processo do Workerman.

**Solução:**
Abra o arquivo de PID do processo principal do Workerman para visualizar o PID principal. O arquivo de PID fica por padrão no diretório paralelo ao Workerman. Execute o comando ```ps aux | grep pid_do_processo_principal``` para verificar se o processo correspondente é o processo do Workerman. Se não for, pode ser que o servidor tenha reiniciado, resultando no PID salvo pelo Workerman ser um PID expirado, o qual está sendo utilizado por outro processo, ocasionando a falha ao parar. Se for esse o caso, exclua o arquivo de PID.

### Quinta Possibilidade
A extensão grpc está instalada, mas as variáveis de ambiente correspondentes à extensão grpc não estão configuradas. Ao iniciar, será criado um processo adicional, o que causa falha ao parar.

**Solução:**
Alocar as variáveis de ambiente adequadas para a extensão grpc após a instalação, a fim de evitar a geração desse processo adicional que impede a parada adequada.
