# Classe Worker
Em Workerman, há duas classes importantes: Worker e Connection.

A classe Worker é usada para realizar a escuta de portas e pode definir funções de retorno de chamada para eventos de conexão do cliente, eventos de recebimento de mensagens na conexão e eventos de desconexão, a fim de realizar o processamento de negócios.

Você pode definir o número de processos da instância Worker (atributo count). O processo principal do Worker irá bifurcar (fork) count sub-processos para ouvir a mesma porta simultaneamente, lidando com a conexão e seus eventos de forma paralela.
