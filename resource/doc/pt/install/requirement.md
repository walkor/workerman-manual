# Requisitos de Ambiente

## Usuários do Windows
A partir da versão 3.5.3, o Workerman pode ser usado tanto em sistemas Linux quanto em sistemas Windows.

1. PHP>=5.4 é necessário e a variável de ambiente do PHP deve estar configurada.
2. A versão do Workerman para Windows não depende de nenhuma extensão.
3. Consulte as instruções de instalação e restrições de uso [**aqui**](https://www.workerman.net/windows).
4. Devido às limitações de uso do Workerman no Windows, é recomendado o uso do sistema Linux para ambientes de produção, com o Windows sendo recomendado apenas para ambientes de desenvolvimento.

``` === For Linux users only, please ignore if you are a Windows user. === ```

## Usuários do Linux (incluindo macOS)
Usuários do Linux devem utilizar a versão do Workerman para Linux.

1. Instale o PHP>=5.4 e as extensões pcntl e posix.
2. É recomendado instalar a extensão event, mas não é obrigatório (observe que a extensão event requer PHP>=5.4).

### Script de Verificação de Ambiente para Linux
Os usuários do Linux podem executar o seguinte script para verificar se o ambiente local atende aos requisitos do Workerman:

```curl -Ss https://www.workerman.net/check | php```

Se todas as mensagens exibidas pelo script estiverem "ok", significa que o ambiente de execução do Workerman está adequado.

(Observação: o script de verificação não inclui a verificação da extensão event; se o número de conexões simultâneas for superior a 1024, é recomendável instalar a extensão event, cujo método de instalação está detalhado na próxima seção.)

## Explicação Detalhada

### Sobre o PHP-CLI

O Workerman é executado no modo de linha de comando do PHP ([PHP-CLI](https://php.net/manual/zh/features.commandline.php)). O PHP-CLI é um programa executável independente do PHP-FPM ou do MOD-PHP do Apache. Não há conflito ou dependência entre eles, sendo completamente independentes.

### Extensões Necessárias para o Workerman

1. [Extensão pcntl](https://cn2.php.net/manual/zh/book.pcntl.php)

A extensão pcntl é essencial para o controle de processos em ambientes Linux. O Workerman depende de recursos como [criação de processo](https://cn2.php.net/manual/zh/function.pcntl-fork.php), [controle de sinais](https://cn2.php.net/manual/zh/function.pcntl-signal.php), [temporizadores](https://cn2.php.net/manual/zh/function.pcntl-alarm.php) e [monitoramento do estado do processo](https://cn2.php.net/manual/zh/function.pcntl-waitpid.php). Esta extensão não é compatível com a plataforma Windows.

2. [Extensão posix](https://cn2.php.net/manual/zh/book.posix.php)

A extensão posix permite que o PHP, em ambiente Linux, acesse interfaces fornecidas pelo sistema de acordo com os padrões [POSIX](https://baike.baidu.com/view/209573.htm). O Workerman utiliza principalmente essas interfaces para implementar recursos como a demonização de processos e o controle de grupos de usuários. Esta extensão não é compatível com a plataforma Windows.

3. [Extensão Event](https://php.net/manual/zh/book.event.php) ou [Extensão libevent](https://cn2.php.net/manual/en/book.libevent.php)

A extensão event permite que o PHP utilize mecanismos avançados de tratamento de eventos, como [Epoll](https://baike.baidu.com/view/1385104.htm) e Kqueue, melhorando significativamente a utilização da CPU pelo Workerman em cenários de conexões concorrentes. Essa extensão é crucial em aplicativos que envolvem conexões persistentes e de alta concorrência. A extensão libevent (ou event) não é obrigatória; sem ela, o Workerman usará o mecanismo de tratamento de eventos nativo do PHP (Select).

## Como Instalar as Extensões

Consulte a seção de [Instalação de Extensões](../appendices/install-extension.md) para obter mais detalhes.
