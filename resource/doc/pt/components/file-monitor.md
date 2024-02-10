# Componente de Monitoramento de Arquivos

**Contexto:**

O Workerman é executado em memória persistente, o que evita a leitura repetitiva de disco e a interpretação repetida e compilação do PHP, a fim de atingir o melhor desempenho. Portanto, as alterações no código do negócio precisam ser recarregadas ou reiniciadas manualmente para entrar em vigor.

Ao mesmo tempo, o Workerman fornece um serviço de monitoramento de atualização de arquivos que, ao detectar uma atualização de arquivo, automaticamente executa o recarregamento, recarregando os arquivos PHP. Os desenvolvedores podem incluí-lo no projeto e iniciá-lo junto com o projeto.

**Links para download do serviço de monitoramento de arquivos:**

1. Versão sem dependências: https://github.com/walkor/workerman-filemonitor

2. Versão dependente do inotify: https://github.com/walkor/workerman-filemonitor-inotify (requer a instalação da extensão [inotify](https://php.net/manual/pt_BR/book.inotify.php))

**Diferenças entre as duas versões:**

A versão do link 1 usa o método de consulta do tempo de atualização do arquivo a cada segundo para determinar se o arquivo foi atualizado.

A versão do link 2 utiliza o mecanismo do kernel Linux [inotify](https://baike.baidu.com/view/2645027.htm), que notifica ativamente o Workerman quando o arquivo é atualizado no sistema.

Geralmente, a primeira versão sem dependências é suficiente.

**Instruções de uso:**

Copie o diretório Applications/FileMonitor para o diretório Applications do seu projeto.

Se o seu projeto não tiver um diretório Applications, você pode copiar o arquivo Applications/FileMonitor/start.php para qualquer local do seu projeto e requerer no script de inicialização.

Por padrão, o componente de monitoramento monitora o diretório Applications. Se for necessário alterar, é possível modificar a variável ```$monitor_dir``` em Applications/FileMonitor/start.php, sendo recomendável que o valor de ```$monitor_dir``` seja um caminho absoluto.

**Observações:**

- O sistema Windows não suporta recarregamento, portanto, este serviço de monitoramento não pode ser utilizado.
- Só funciona em modo de depuração; não será executado em modo daemon (veja a explicação abaixo sobre por que não é suportado em modo daemon).
- Apenas os arquivos carregados após a execução de Worker::runAll podem ser atualizados dinamicamente, ou seja, apenas os arquivos carregados nos callbacks onXXX podem ser atualizados dinamicamente.

**Por que não é suportado em modo daemon?**

O modo daemon geralmente é usado para a execução em ambientes de produção. Quando uma versão de produção é publicada, geralmente vários arquivos são publicados de uma vez, e esses arquivos podem ter dependências entre si. Como a sincronização de vários arquivos para o disco leva um certo tempo, pode haver um momento em que os arquivos no disco não estão completos. Se o monitoramento detectar uma atualização de arquivo e executar um recarregamento nesse momento, existe o risco de erros fatais devido à falta de arquivos.

Além disso, em ambientes de produção, às vezes é necessário depurar online. Se o código for editado e salvo diretamente, terá efeito imediato, podendo resultar em erros de sintaxe que tornam o serviço online indisponível. O método correto seria salvar o código, verificar se há erros de sintaxe usando ```php -l yourfile.php```, e então recarregar dinamicamente o código.

Se o desenvolvedor realmente precisar habilitar o monitoramento e a atualização automática de arquivos no modo daemon, é possível alterar o código em Applications/FileMonitor/start.php, removendo a verificação de Worker::$daemonize.
