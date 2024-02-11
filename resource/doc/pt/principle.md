# Princípio

### Descrição do Worker
O Worker é o contêiner mais básico no Workerman e pode abrir vários processos para escutar portas e se comunicar usando um protocolo específico, semelhante ao nginx que escuta uma porta específica. Cada processo do Worker opera independentemente, utilizando Epoll (requer extensão event) e IO não bloqueante. Cada processo do Worker pode lidar com dezenas de milhares de conexões de clientes e processar os dados enviados por essas conexões. O processo principal é responsável apenas por monitorar os processos filhos para manter a estabilidade, sem receber dados ou realizar qualquer lógica de negócios.

### Relação entre o cliente e o processo do Worker
![workerman master woker模型](images/Worker.png)

### Relação entre o processo principal e os processos filhos
![workerman master woker模型](images/Worker2.png)

**Características:**

Através dos gráficos, podemos observar que cada Worker mantém suas próprias conexões de clientes, o que facilita a implementação de comunicação em tempo real entre clientes e servidores. Com base nesse modelo, podemos facilmente atender a algumas necessidades básicas de desenvolvimento, como servidores HTTP, servidores Rpc, relatórios em tempo real de dispositivos inteligentes, envio de dados do servidor, servidores de jogos, backends para aplicativos WeChat mini-program, entre outros.
