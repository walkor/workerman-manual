# Falha na Atualização do Código

**Razão:**

O Workerman é executado em memória residencial, o que evita a leitura repetida do disco e a interpretação repetida do PHP, proporcionando assim o melhor desempenho. Portanto, após a alteração do código de negócio, é necessário recarregar manualmente ou reiniciar para que as alterações tenham efeito.

Além disso, o Workerman fornece um serviço de monitoramento de atualização de arquivos que detecta automaticamente as atualizações nos arquivos e executa o recarregamento, re-carregando os arquivos PHP. Os desenvolvedores podem colocá-lo no projeto para que ele seja iniciado junto com o projeto.

Nota: O sistema Windows não suporta recarregamento e não pode usar o serviço de monitoramento.

**Endereços para Download do Serviço de Monitoramento de Arquivos:**

1. Versão sem dependências: https://github.com/walkor/workerman-filemonitor

2. Versão com dependência inotify: https://github.com/walkor/workerman-filemonitor-inotify

**Diferenças entre as duas versões:**

A versão no endereço 1 utiliza o método de verificar o tempo de atualização do arquivo a cada segundo para determinar se o arquivo foi atualizado.

A versão no endereço 2 utiliza o mecanismo inotify do kernel Linux, no qual o sistema notifica o Workerman quando ocorre uma atualização no arquivo.

Normalmente, a versão sem dependências do endereço 1 é suficiente.
