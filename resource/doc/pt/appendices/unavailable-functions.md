# Funções não Suportadas

Função/Declaração não Suportada  | Alternativa | Descrição
----|------|----
pcntl_fork | Definir o número de processos antecipadamente| 
php://input | [`$request->rawBody()`](http/request.md)| Usado para aplicativos sob o protocolo HTTP para obter os dados brutos POST
exit | return | Usar exit fará com que o processo termine, se quiser retornar, use diretamente a declaração return
die | return | Usar die fará com que o processo termine, se quiser retornar, use diretamente a declaração return
header cookie sessão funções relacionadas | Consulte a classe [`$request`](http/request.md) e [`$response`](http/response.md) | 
set_time_limit| Nenhum | Apenas pode ser configurado como 0, caso contrário, fará com que o processo do workerman termine após um período de tempo determinado
