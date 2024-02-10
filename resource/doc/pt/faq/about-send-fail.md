# Motivo de send_fail no status

**Sintoma:**

Ao executar o comando status, verifica-se a ocorrência de send_fail. Qual é o motivo?

**Resposta:**

O send_fail geralmente não é um grande problema, normalmente ocorre devido ao fechamento ativo da conexão pelo cliente ou à incapacidade do cliente de receber dados.

send_fail tem duas causas:

1. Quando a função send é chamada para enviar dados ao cliente e é detectado que o cliente foi desconectado, o contador de send_fail é incrementado. Como a desconexão foi iniciada pelo cliente, geralmente pode ser ignorada, pois é um fenômeno normal.

2. A velocidade de envio de dados pelo servidor é maior do que a velocidade de recebimento pelo cliente, resultando no acúmulo contínuo de dados no buffer do servidor (o Workerman cria um buffer de envio para cada cliente). Se o tamanho do buffer exceder o limite (padrão de 1 MB em TcpConnection::$maxSendBufferSize), os dados serão descartados, acionando o evento onError (se houver) e aumentando o contador de send_fail.

Por exemplo, quando um navegador é minimizado, o JavaScript pode ser suspenso, fazendo com que o navegador pare de receber os dados do servidor. Com os dados acumulando no buffer por muito tempo, excedendo o limite, cada chamada de send resultará no aumento do contador de send_fail.

**Conclusão:**

O send_fail causado pela desconexão do cliente geralmente não é motivo de preocupação.

Se o send_fail for causado pela interrupção do recebimento de dados pelo cliente, é necessário verificar se o cliente está funcionando corretamente.

Se a velocidade de recebimento de dados pelo cliente for **continuamente** menor que a velocidade de envio pelo servidor, é necessário considerar a otimização do fluxo de trabalho ou o aprimoramento do desempenho do cliente. Se a escassez de largura de banda estiver causando problemas no envio, pode ser necessário aumentar a largura de banda do servidor.
