# Usando WorkerMan para enviar dados para o cliente em outros projetos

**Pergunta:**

Eu tenho um projeto web comum e gostaria de chamar a interface do WorkerMan neste projeto para enviar dados para o cliente.


**Resposta:**

**Com base no Workerman, você pode consultar o seguinte link:**

- [Exemplo de envio com o componente Channel](../components/channel-examples.md) (suporta múltiplos processos/servidores em cluster, é necessário baixar o componente Channel)

- [Envio baseado em Worker](https://www.workerman.net/q/508) (apenas um processo, o mais simples)

**Com base no webman, consulte o seguinte link:**
  
- [Plugin de envio do webman](https://www.workerman.net/plugin/2)


**Com base no GatewayWorker, consulte o seguinte link:**

- [Envio através do GatewayWorker em outros projetos](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html) (suporta múltiplos processos/servidores em cluster, suporta grupos, multicast, envio individual)


**Com base no PHPSocket.IO, consulte o seguinte link:**

- [Envio de mensagens web](https://www.workerman.net/web-sender) (por padrão, apenas um processo, baseado em socket.io, a melhor compatibilidade com navegadores)
