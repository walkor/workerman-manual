# Verificação de Funções Desativadas

Utilize este script para verificar se algumas funções estão desativadas. Execute o comando no terminal ```curl -Ss https://www.workerman.net/check | php```.

Se aparecer a mensagem ```Function nome_da_funcao pode estar desativada. Por favor, verifique disable_functions no php.ini```, isso significa que as funções necessárias para o funcionamento do Workerman estão desativadas e precisam ser habilitadas no php.ini para que o Workerman funcione corretamente.

Para habilitar as funções, siga um dos métodos abaixo.

## Método Um: Script de Habilitação

Execute o script `curl -Ss https://www.workerman.net/fix | php` para habilitar as funções.

## Método Dois: Habilitação Manual

**Passos a seguir:**

1. Execute `php --ini` para encontrar o local do arquivo php.ini usado pelo php cli.

2. Abra o php.ini e encontre a entrada `disable_functions` para habilitar as funções correspondentes.

**Funções Necessárias**

Para utilizar o Workerman, é necessário habilitar as seguintes funções:
```
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
