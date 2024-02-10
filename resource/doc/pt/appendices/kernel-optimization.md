# Otimização do kernel do Linux

Para poder suportar um maior número de conexões simultâneas, além da instalação obrigatória da [extensão de evento](../install/install.md), a otimização do kernel do Linux também é de extrema importância. Cada uma das otimizações a seguir é extremamente crucial, portanto, é essencial concluí-las uma a uma.

**Explicações dos parâmetros:**

> **max-file**: Representa a quantidade de identificadores de arquivo que o sistema pode abrir em nível de sistema. Este valor é para o sistema operacional como um todo e não para usuários.
> 
> **ulimit -n**: Controla a quantidade de identificadores de arquivo que um processo pode abrir em nível de processo. Isso controla o número de identificadores de arquivo disponíveis para o usuário atual do shell e os processos iniciados por ele.

Verifique a quantidade de identificadores de arquivo que o sistema pode abrir em nível de sistema: `cat /proc/sys/fs/file-max`

Abra o arquivo /etc/sysctl.conf e adicione as seguintes configurações
```conf
# Este parâmetro define a quantidade de TIME_WAIT do sistema. Se exceder o valor padrão, será imediatamente limpo
net.ipv4.tcp_max_tw_buckets = 20000
# Define o comprimento máximo da fila de escuta para cada porta no sistema
net.core.somaxconn = 65535
# Número máximo de solicitações de conexão TCP pendentes que ainda não receberam a confirmação do outro lado
net.ipv4.tcp_max_syn_backlog = 262144
# Quando a taxa de recebimento de pacotes em uma interface de rede é maior do que a taxa na qual o kernel está processando esses pacotes, o número máximo de pacotes que podem ser colocados na fila
net.core.netdev_max_backlog = 30000
# Isso fará com que os clientes em uma rede NAT expirem. Recomenda-se definir como 0. O Linux removeu a configuração tcp_tw_recycle a partir do kernel 4.12. Se ocorrer o erro "No such file or directory", ignore-o
net.ipv4.tcp_tw_recycle = 0
# O número total de arquivos que todos os processos do sistema podem abrir
fs.file-max = 6815744
# Tamanho da tabela de rastreamento de firewall. Observação: se o firewall estiver desativado, poderá ocorrer o erro "net.netfilter.nf_conntrack_max" is an unknown key, que pode ser ignorado
net.netfilter.nf_conntrack_max = 2621440
net.ipv4.ip_local_port_range = 10240 65000
```
Execute `sysctl -p` para que entre em vigor imediatamente.

**Nota:**

/etc/sysctl.conf possui muitas opções configuráveis, e outras opções podem ser configuradas de acordo com as necessidades do ambiente.

## Número de arquivos abertos

Ajuste as configurações do sistema para o número de arquivos abertos, a fim de resolver o problema de ```too many open files``` em situações de alta concorrência. Esta opção afeta diretamente o número máximo de conexões de clientes que um único processo pode gerenciar.

O número de arquivos abertos soft é um parâmetro do sistema Linux que afeta o número máximo de identificadores de arquivo que um processo pode abrir. Esse valor afeta aplicativos de conexões de longa duração, como o número máximo de conexões de usuários que um único processo pode manter. Você pode verificar esse valor executando `ulimit -n`. Se for 1024, isso significa que um único processo pode manter no máximo 1024 ou até menos conexões, devido a outros identificadores de arquivos abertos. Se você iniciar 4 processos para manter as conexões do usuário, então o aplicativo como um todo pode manter no máximo 4x1024 usuários, ou seja, suportar no máximo 4x1024 usuários online. Você pode aumentar essa configuração para que o serviço possa manter mais conexões TCP.

**Três métodos para modificar o número de arquivos abertos soft:**

Primeiro método: execute `ulimit -HSn 102400` no terminal e depois reinicie o webman.
Esta opção é válida apenas para o terminal atual. Quando você sair do terminal, o número de arquivos abertos voltará ao valor padrão.

Segundo método: adicione `ulimit -HSn 102400` no final do arquivo `/etc/profile`. Isso executará automaticamente toda vez que você fizer login no terminal. Depois de fazer o ajuste, você precisará reiniciar o webman.

Terceiro método: para tornar a alteração do número de arquivos abertos permanente, é necessário modificar o arquivo de configuração: `/etc/security/limits.conf`. Adicione o seguinte ao final deste arquivo:

``` 
* soft nofile 1024000
* hard nofile 1024000
root soft nofile 1024000
root hard nofile 1024000
```

Este método requer a reinicialização do servidor para entrar em vigor.
