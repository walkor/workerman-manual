# O Webman suporta quantas conexões simultâneas

O conceito de **concorrência** é muito vago, aqui vamos falar sobre duas métricas quantificáveis: **número de conexões simultâneas** e **número de solicitações simultâneas**.

**Número de conexões simultâneas** refere-se ao total de conexões TCP mantidas pelo servidor em um determinado momento, independentemente de haver comunicação de dados nessas conexões. Por exemplo, um servidor de envio de mensagens pode manter milhões de conexões de dispositivos, mas como há pouca comunicação de dados nessas conexões, a carga no servidor pode ser quase zero, desde que haja memória suficiente para continuar aceitando conexões.

**Número de solicitações simultâneas** geralmente é medido em QPS (quantas solicitações o servidor processa por segundo), sem muita preocupação com o número de conexões TCP atuais no servidor. Por exemplo, um servidor pode ter apenas 10 conexões de clientes, mas cada uma dessas conexões pode gerar 10.000 solicitações por segundo. Nesse caso, o servidor precisaria sustentar pelo menos 10 * 10.000 = 100.000 de throughput por segundo (QPS). Se esse limite de 100.000 de throughput por segundo for o máximo para o servidor, isso significa que o servidor pode suportar 100.000 clientes se cada um enviar uma solicitação por segundo.

O **número de conexões simultâneas** é limitado pela memória do servidor. Em geral, um servidor Workerman com 24 GB de memória pode suportar aproximadamente **1,200,000** conexões simultâneas.

O **número de solicitações simultâneas** é limitado pela capacidade de processamento da CPU do servidor. Um servidor Workerman com 24 núcleos pode atingir um throughput de **450,000** solicitações por segundo (QPS). O valor real pode variar de acordo com a complexidade do negócio e a qualidade do código.

## Observação

Para cenários de alta concorrência, é necessário instalar a extensão "event", consulte a seção de configuração de instalação. Além disso, é necessário otimizar o kernel do Linux, especialmente o limite de arquivos abertos por processo. Consulte a seção de otimização do kernel no apêndice.

## Dados de Teste de Pressão

Os dados são provenientes de um teste de pressão da 20ª rodada de uma agência de teste de terceiros e autoritária - techempower.com https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf

**Configuração do Servidor:**
Total de núcleos 14, Total de threads 28, 32 GB de memória, Switch Cisco Ethernet dedicado de 10 gigabits
**Lógica de negócios:**
Negócio com consulta ao banco de dados, banco de dados pgsql, php8+jit
QPS de 390.000+
![](../images/screenshot_1636522357217.png)
