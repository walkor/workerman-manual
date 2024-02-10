# Prefácio

**Workerman, um container de aplicativos PHP de alta performance**

## O que é o Workerman?
Workerman é um container de aplicativos PHP de código aberto de alta performance desenvolvido puramente em PHP.

Workerman não é apenas mais uma repetição de código, ele não é um framework MVC, mas sim um framework de serviço mais geral e de nível mais baixo. Com ele, você pode desenvolver proxies TCP, proxies VPN, servidores de jogos, servidores de e-mail, servidores FTP e até mesmo uma versão em PHP do Redis, uma versão em PHP de um banco de dados, uma versão em PHP do Nginx, uma versão em PHP do PHP-FPM, entre outros. Workerman pode ser considerado uma inovação no campo do PHP, permitindo que os desenvolvedores se libertem completamente das restrições de que o PHP só pode ser usado para desenvolvimento web.

Na prática, o Workerman é semelhante a uma versão em PHP do Nginx, com o núcleo consistindo em processos múltiplos, EventPoll e E/S não bloqueante. Cada processo do Workerman pode manter dezenas de milhares de conexões concorrentes. Por residir na memória, sem depender de contêineres como Apache, Nginx, PHP-FPM, ele possui um desempenho extremamente alto. Além disso, suporta TCP, UDP, UNIX SOCKET, conexões de longo prazo, Websocket, HTTP, WSS, HTTPS e vários outros protocolos de comunicação personalizados. Ele também possui muitos componentes de alta performance, como temporizadores, clientes de soquete assíncrono, Redis assíncrono, HTTP assíncrono, filas de mensagens assíncronas, entre outros.

## Algumas direções de aplicação do Workerman
Diferente dos frameworks MVC tradicionais, o Workerman não é apenas usado para desenvolvimento web, mas também possui uma ampla gama de aplicações, como comunicação instantânea, Internet das Coisas, jogos, governança de serviços e outros servidores ou middleware, o que sem dúvida amplia consideravelmente os horizontes dos desenvolvedores PHP. Atualmente, há uma escassez de desenvolvedores PHP nesses campos, então, se você deseja ter uma vantagem técnica no campo do PHP, não está satisfeito com o trabalho diário de CRUD ou deseja seguir em direção a arquitetura ou se tornar um especialista técnico, o Workerman é uma estrutura muito merecedora de estudo. Recomenda-se que os desenvolvedores não apenas saibam como usá-lo, mas também desenvolvam projetos de código aberto baseados no Workerman, aprimorem suas habilidades e aumentem sua influência. Por exemplo, o [Beanbun, um framework de web crawler multi-processo](https://github.com/kiddyuchina/Beanbun), é um excelente exemplo, que recebeu muitos elogios pouco tempo após seu lançamento.

Algumas direções de aplicação do Workerman incluem:

1. Comunicação instantânea
   Como bate-papo instantâneo na web, envio de mensagens instantâneas, aplicativos de mensagens para celulares, envio de mensagens para aplicativos de PC, entre outros.
   [Exemplos: sala de bate-papo do Workerman](https://www.workerman.net/workerman-chat), [envio de mensagens web](https://www.workerman.net/web-sender), [sala de bate-papo Tadpole](https://www.workerman.net/workerman-todpole)

2. Internet das Coisas
   Como comunicação com impressoras, comunicação com microcontroladores, pulseiras inteligentes, automação residencial, bicicletas compartilhadas, entre outros.
   [Exemplos de clientes: Yilianyun, Yiboshi Daili]

3. Servidores de jogos
   Como jogos de tabuleiro, jogos MMORPG, entre outros.
   [Exemplo: browserquest-php](https://www.workerman.net/browserquest)

4. Serviços HTTP
   Como a criação de interfaces HTTP de alto desempenho e websites de alto desempenho. Se você deseja oferecer serviços ou sites HTTP, é altamente recomendado o uso do [webman](https://github.com/walkor/webman).

5. SOA Service-Oriented Architecture
   Usando o Workerman para encapsular diferentes unidades de funcionalidade de negócios existentes como serviços para fornecer uma interface unificada, alcançando desacoplamento flexível, fácil manutenção, alta disponibilidade e escalabilidade fácil.
   [Exemplos: workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc), [workerman-thrift](https://github.com/walkor/workerman-thrift)

6. Outros softwares de servidor
   Como [GatewayWorker](https://www.workerman.net/doc/gateway-worker), [PHPSocket.IO](https://www.workerman.net/phpsocket_io), [proxy HTTP](https://github.com/walkor/php-http-proxy), [proxy SOCKS5](https://github.com/walkor/php-socks5), [componente de comunicação distribuída](https://github.com/walkor/Channel), [componente de compartilhamento de variáveis distribuídas](https://github.com/walkor/GlobalData), [fila de mensagens](https://github.com/walkor/workerman-queue), servidores DNS, servidores web, servidores CDN, servidores FTP, entre outros.

7. Componentes
   Como [Redis assíncrono](components/workerman-redis.md), [cliente HTTP assíncrono](components/workerman-http-client.md), [Cliente MQTT para IoT](components/workerman-mqtt.md), [filas de mensagens Redis-queue](components/workerman-redis-queue.md), [stomp](components/workerman-stomp.md), [rabbitmq](components/workerman-rabbitmq.md), [monitor de arquivos](components/file-monitor.md), e muitos outros frameworks e componentes de terceiros.

Claramente, é difícil para os frameworks MVC tradicionais alcançar as funcionalidades mencionadas acima, o que explica o surgimento do Workerman.

## Princípios do Workerman
Simplicidade, estabilidade, alta performance, e distribuição.

### **Simplicidade**
O núcleo do Workerman é simples, consistindo apenas de alguns arquivos PHP e expondo apenas algumas interfaces, o que facilita muito a aprendizagem. Todos os outros recursos são estendidos por meio de componentes.

O Workerman possui uma documentação completa, um site oficial autorizado, uma comunidade ativa, vários grupos no QQ com milhares de usuários, e muitos componentes de alta performance, além de inúmeros exemplos, tudo isso torna mais fácil para os desenvolvedores utilizarem o Workerman.

### **Estabilidade**
O Workerman é de código aberto há vários anos e é amplamente utilizado por grandes empresas listadas, sendo extremamente estável. Alguns serviços operam em alta velocidade sem reinicialização por mais de dois anos. Sem coredump, vazamentos de memória ou bugs.

### **Alta performance**
Devido à residência na memória, sem depender de Apache/Nginx/PHP-FPM, sem custo de comunicação entre contêineres e sem custo de inicialização e desativação de recursos a cada requisição, o Workerman tem desempenho extremamente alto. Comparado aos frameworks MVC tradicionais, o desempenho é várias dezenas de vezes maior. Sob o PHP 7, os testes de pressão com AB mostraram um QPS até superior ao do Nginx sozinho.

### **Distribuição**
Atualmente, não se trata mais de um mundo de ação isolada. Mesmo com o desempenho de um único servidor aumentando, a distribuição em vários servidores é fundamental. O Workerman oferece diretamente um conjunto de soluções distribuídas para comunicação de longa duração [GatewayWorker framework](https://doc2.workerman.net). Basta adicionar servidores e iniciar com uma simples configuração, sem alteração no código de negócios, a capacidade do sistema é multiplicada. Se você está desenvolvendo aplicativos de longa duração TCP, é altamente recomendado o uso do [GatewayWorker](https://doc2.workerman.net), que é uma camada adicional em cima do Workerman, oferecendo uma interface mais rica e um poderoso processamento distribuído para aplicativos de longa duração.

## Escopo deste manual
Versões 3.x - 4.x do Workerman

## Usuários do Windows (leitura obrigatória)
O Workerman oferece suporte tanto para sistemas Linux quanto para Windows. A versão do Windows do Workerman **não depende de extensões** e, portanto, requer apenas a configuração adequada das variáveis de ambiente do PHP. Para obter mais informações sobre a instalação e requisitos da versão do Workerman para Windows, consulte [Leitura obrigatória para usuários do Windows](https://www.workerman.net/windows).

## Clientes
O protocolo de comunicação do WorkerMan é aberto e personalizável, portanto, teoricamente, o WorkerMan pode se comunicar com clientes de qualquer plataforma que use qualquer protocolo. Ao desenvolver um cliente, é possível se comunicar com o servidor com base no protocolo de comunicação correspondente.
