# Falha ao iniciar o workerman

## Sintoma 1
Após a inicialização, aparece um erro semelhante ao seguinte:
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx

```
**Palavras-chave**: ```Address already in use```

**Causa raiz**: A porta está sendo utilizada por outra aplicação, impossibilitando a inicialização.

#### Solução 1

Pode-se encontrar qual programa está usando a porta através do comando ```netstat -anp | grep port_number```, em seguida, interromper o programa correspondente para liberar a porta.

#### Solução 2
Se não for possível interromper o programa que está usando a porta, pode-se resolver o problema alterando a porta do workerman.

#### Solução 3
Se a porta está sendo utilizada pelo Workerman e não é possível interrompê-lo através do comando "stop" (geralmente devido à perda do arquivo pid ou por ter sido encerrado pelo desenvolvedor), pode-se encerrar o processo do Workerman executando os seguintes dois comandos.

```shell
killall php
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

#### Solução 4
Se de fato não há nenhum programa escutando essa porta, pode ser que o desenvolvedor tenha configurado dois ou mais listeners no workerman, ambos ou todos escutando na mesma porta. Nesse caso, o desenvolvedor precisa verificar o script de inicialização para garantir que não há portas duplicadas.

#### Solução 5
Verifique se o programa está utilizando o reusePort e tente desativá-lo.

## Sintoma 2
Após a inicialização, aparece um erro semelhante ao seguinte:
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
ou
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (在其上下文中，该请求的地址无效) in ...workerman/Worker.php on line xxxx
```
**Palavras-chave**: `Cannot assign requested address`, `该请求的地址无效`

**Causa da falha**:

O endereço de IP fornecido no script de inicialização está incorreto, não é um endereço de IP local. É necessário fornecer o endereço de IP local ou usar ```0.0.0.0``` (para escutar em todos os endereços de IP locais).

**Dica**: No sistema Linux, pode-se usar o comando ```ifconfig``` para verificar todos os endereços de IP das interfaces de rede. Se estiver usando um servidor em nuvem (como Alibaba Cloud ou Tencent Cloud), observe que o endereço de IP público pode ser na realidade um endereço de IP proxy (por exemplo, uma rede privada na Alibaba Cloud), e que o endereço de IP público não pertence ao servidor atual. Nesse caso, não é possível escutar no endereço de IP público, mas ainda é possível vincular para ```0.0.0.0```.

## Sintoma 3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**Causa da falha**:

A função stream_socket_server foi desativada no arquivo php.ini.

**Solução**

1. Execute o comando ```php --ini``` para encontrar o arquivo php.ini.
2. Abra o arquivo php.ini, encontre a diretiva disable_functions e remova a função stream_socket_server da lista de funções desativadas.

## Sintoma 4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**Causa da falha**:

No Linux, para escutar em portas com números inferiores a 1024, é necessário ter permissões de root.

**Solução**

Use uma porta com um número maior que 1024 ou execute o serviço como usuário root.
