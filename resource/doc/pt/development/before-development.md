# Leitura obrigatória antes do desenvolvimento

Ao desenvolver um aplicativo usando Workerman, é necessário entender o seguinte:

## 1. Diferenças entre o desenvolvimento com Workerman e o desenvolvimento PHP comum

Além de não ser possível o uso direto de variáveis e funções relacionadas ao protocolo HTTP, o desenvolvimento com Workerman não difere muito do desenvolvimento PHP comum.

### 1.1. Diferença no protocolo de aplicação
- O desenvolvimento PHP comum geralmente é baseado no protocolo de aplicação HTTP, em que o servidor web já completou a análise do protocolo para o desenvolvedor.
- O Workerman suporta vários protocolos e atualmente possui protocolos embutidos, como HTTP e WebSocket. O Workerman recomenda aos desenvolvedores o uso de protocolos de comunicação personalizados mais simples.
- Para o desenvolvimento com protocolo HTTP, consulte a seção de [serviço HTTP](../http/request.md).

### 1.2. Diferenças no ciclo de solicitação
- No PHP em um aplicativo web, após uma solicitação, todas as variáveis e recursos são liberados.
- Os aplicativos desenvolvidos com Workerman permanecem na memória após a primeira análise, tornando a definição de classes, objetos globais e membros estáticos de classe disponíveis para reutilização subsequente.

### 1.3. Evitar a definição duplicada de classes e constantes
- Como o Workerman em cache dos arquivos PHP compilados, evite incluir várias vezes o mesmo arquivo de definição de classe ou constante. É recomendável usar require_once ou include_once para carregar arquivos.

### 1.4. Liberação de recursos de conexão em modo Singleton
- Como o Workerman não libera objetos globais e membros estáticos de classe após cada solicitação, em modelos Singleton, como em bancos de dados, é comum salvar uma instância do banco de dados (que inclui uma conexão de soquete com o banco de dados) em um membro estático da classe do banco de dados. Assim, o Workerman reutiliza essa conexão de soquete do banco de dados durante todo o ciclo de vida do processo. É importante observar que, quando o servidor de banco de dados percebe que uma conexão está inativa por um certo período de tempo, ele pode fechar a conexão do soquete, resultando em um erro ao usar essa instância do banco de dados novamente (um erro semelhante a "mysql gone away"). O Workerman fornece uma [classe de banco de dados](../components/workerman-mysql.md) com reconexão automática, que os desenvolvedores podem usar diretamente.

### 1.5. Evitar o uso de declarações de saída como exit ou die
- O Workerman é executado no modo de linha de comandos do PHP. Ao chamar declarações de saída como exit ou die, o processo atual será encerrado. Embora o processo filho seja imediatamente recriado após encerramento, ainda pode afetar o negócio.

### 1.6. Necessidade de reinicializar o serviço após a alteração do código
Como o Workerman permanece na memória, as definições de classes e funções PHP carregadas uma vez permanecem na memória e não são lidas novamente do disco. Portanto, após alterar o código do aplicativo, é necessário reiniciar o serviço para que as alterações tenham efeito.

## 2. Conceitos básicos a serem compreendidos

### 2.1. Protocolo de transporte TCP
O TCP é um protocolo de transporte baseado em IP, orientado à conexão e confiável. Uma característica importante do TCP é que ele é baseado em fluxo de dados, onde as solicitações do cliente são enviadas continuamente ao servidor, e os dados recebidos pelo servidor podem não ser uma solicitação completa ou até mesmo podem ser várias solicitações em conjunto. Portanto, é necessário definir um conjunto de regras no protocolo de aplicação para distinguir os limites de cada solicitação dentro do fluxo contínuo de dados.

### 2.2. Protocolo de aplicação
O protocolo de aplicação define como os processos de aplicativos em diferentes sistemas finais (cliente, servidor) se comunicam entre si, por exemplo, HTTP e WebSocket são protocolos de aplicação. Um exemplo simples de um protocolo de aplicação é o seguinte: `{"module":"user","action":"getInfo","uid":456}\n`. Este protocolo usa `"\n"` (observe que `"\n"` representa uma quebra de linha) para marcar o final da solicitação, e o corpo da mensagem é uma string.

### 2.3. Conexão de curta duração
Uma conexão de curta duração implica estabelecer uma conexão quando há troca de dados entre as partes e desconectar a conexão após o envio dos dados, ou seja, cada conexão completa apenas um envio de dados. Isso é comum em serviços HTTP de sites da web.

### 2.4. Conexão de longa duração
Uma conexão de longa duração permite o envio de vários pacotes de dados em uma única conexão. Observação: aplicativos de conexão de longa duração devem incluir [testes de pulsação](../faq/heartbeat.md), caso contrário, a conexão pode ser fechada pelo firewall do nó de encaminhamento devido à inatividade por um longo período. Conexões de longa duração são comumente usadas em casos de comunicação ponto a ponto e operações frequentes. Cada conexão TCP requer um aperto de mão de três etapas, o que leva tempo. Se cada operação requer uma conexão e, em seguida, uma operação, a velocidade de processamento será significativamente reduzida. Portanto, com a conexão de longa duração, não há desconexão após cada operação; portanto, na próxima operação, os dados podem ser enviados diretamente, sem a necessidade de estabelecer uma nova conexão. Por exemplo, a conexão com o banco de dados utiliza uma conexão de longa duração. Se a comunicação for frequente usando uma conexão de curta duração, isso resultará em erros de soquete e desperdício de recursos. Quando é necessário enviar dados ativamente para os clientes, como em aplicativos de chat, jogos em tempo real e notificação para dispositivos móveis, é necessária uma conexão de longa duração.

### 2.5. Reinicialização suave
A reinicialização normalmente leva a uma interrupção de todos os processos, seguida pela criação de novos processos de serviço do zero. Durante esse processo, pode haver um breve período de tempo em que nenhum processo fornece serviços, o que pode tornar o serviço temporariamente indisponível, levando a falhas nas solicitações em situações de alto volume de tráfego. Por outro lado, a reinicialização suave não interrompe todos os processos de uma vez, mas para cada processo sendo parado, imediatamente cria um novo processo para substituí-lo. Isso ocorre até que todos os processos antigos tenham sido substituídos por novos. O Workerman implementa a reinicialização suave com o comando `php your_file.php reload`, que permite atualizar o programa sem afetar a qualidade do serviço.

**Observação:** Somente os arquivos carregados nas funções de retorno `on{...}` serão atualizados automaticamente após a reinicialização suave. Os arquivos carregados diretamente no script de inicialização ou o código rígido não serão atualizados automaticamente.

## 3. Distinção entre processo principal e processo secundário
É importante estar ciente de se o código está sendo executado no processo principal ou no secundário. Em geral, o código que é executado antes da chamada `Worker::runAll();` é executado no processo principal, e o código executado nas chamadas de retorno `onXXX` é executado no processo secundário. Observe que o código escrito após `Worker::runAll();` nunca será executado.

Por exemplo, o código a seguir:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Executado no processo principal
$tcp_worker = new Worker("tcp://0.0.0.0:2347");
// Atribuições feitas no processo principal
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Este trecho é executado no processo secundário
    $connection->send('hello ' . $data);
};

Worker::runAll();
```

**Observação:** Não inicialize recursos de conexão, como banco de dados, memcache ou conexões Redis no processo principal, pois as conexões inicializadas no processo principal podem ser herdadas automaticamente pelos processos secundários (especialmente ao usar o padrão Singleton), resultando em todos os processos mantendo a mesma conexão. Isso pode levar a dados incorretos quando o servidor retorna dados que podem ser lidos por vários processos, resultando em confusão de dados. Da mesma forma, se qualquer processo fechar a conexão (por exemplo, ao executar o modo deamon, o processo principal sai, resultando no fechamento da conexão), isso também fechará todas as conexões dos processos secundários, causando erros imprevisíveis, como o erro "mysql gone away".

É recomendado inicializar os recursos de conexão em `onWorkerStart`.
