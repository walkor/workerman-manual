# Solicitações concentradas em determinados processos

### Fenômeno
Às vezes, ao usar o comando `php start.php status`, percebemos que as solicitações estão sendo concentradas em processos específicos para processamento, enquanto outros processos estão completamente ociosos.

### Mecanismo de preempção
A forma como vários processos no Workerman obtêm conexões é **por padrão** **preemptiva**, ou seja, quando um cliente inicia uma conexão, todos os processos ociosos têm a oportunidade de capturar essa conexão, e o mais rápido ganha. Quem será o mais rápido é decidido pelo escalonamento do kernel do sistema operacional. O sistema operacional pode dar preferência ao processo que foi usado mais recentemente para ter direitos de uso da CPU, porque as informações de contexto do processo anterior ainda podem estar nos registradores da CPU, o que pode reduzir o custo da troca de contexto. Portanto, quando o desempenho do aplicativo é rápido o suficiente ou durante testes de carga, é mais provável que as conexões sejam concentradas no processamento de alguns processos, já que essa estratégia evita trocas de processos frequentes e geralmente oferece o melhor desempenho, não sendo necessariamente um problema.

### Mecanismo de poluição
O Workerman pode mudar para um mecanismo de obtenção de conexões em um **modo de poluição**, definindo `$worker->reusePort = true;`. Nesse modo, o kernel distribuirá as conexões de maneira mais ou menos uniforme entre todos os processos, permitindo que todos os processos lidem com as solicitações juntos.

### Equívoco
Muitos desenvolvedores acreditam que quanto mais processos participarem do processamento das solicitações, melhor será o desempenho. Na verdade, nem sempre é assim. Quando o aplicativo é simples o suficiente, o número de processos envolvidos no processamento das solicitações se aproximando do número de núcleos da CPU permite uma maior taxa de transferência do servidor. Por exemplo, em um servidor de 4 núcleos, definir o número de processos como 4 geralmente resultará na maior taxa de QPS ao testar com "helloworld". Se o número de processos envolvidos no processamento exceder significativamente o número de núcleos da CPU, o custo do contexto do processo será maior, levando a um desempenho pior. No entanto, em operações de negócios gerais que envolvem bancos de dados, definir o número de processos como 3 a 6 vezes o número de núcleos da CPU pode resultar em um melhor desempenho.
