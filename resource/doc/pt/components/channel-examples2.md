# Exemplo 1
**``` (Requer Workerman versão >= 3.3.0) ```**

Sistema de envio em grupo baseado em multiprocessos com base no Worker

``` php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // Mapeamento global de conexões para grupos
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // Conectar o cliente do Channel ao servidor do Channel
        Channel\Client::connect('127.0.0.1', 2206);

        // Ouvir o evento de envio de mensagem para grupos globais
        Channel\Client::on('send_to_group', function($event_data){
            $group_id = $event_data['group_id'];
            $message = $event_data['message'];
            global $group_con_map;
            var_dump(array_keys($group_con_map));
            if (isset($group_con_map[$group_id])) {
                foreach ($group_con_map[$group_id] as $con) {
                    $con->send($message);
                }
            }
        });
    };
    $worker->onMessage = function(TcpConnection $con, $data){
        // Entrar na mensagem do grupo {"cmd":"add_group", "group_id":"123"}
        // ou enviar mensagem para grupo {"cmd":"send_to_group", "group_id":"123", "message":"Esta é a mensagem"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // Conectar-se a um grupo
            case "add_group":
                global $group_con_map;
                // Adicionar a conexão ao array de grupo correspondente
                $group_con_map[$group_id][$con->id] = $con;
                // Registrar em quais grupos essa conexão ingressou, facilitando a limpeza do group_con_map no evento onclose
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // Enviar mensagem para o grupo
            case "send_to_group":
                // O cliente do Channel envia um evento de envio de mensagem para grupos para todos os processos de todos os servidores
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // Importante: excluir a conexão dos dados globais do grupo quando a conexão é fechada para evitar vazamento de memória
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // Iterar sobre todos os grupos que a conexão entrou e excluir os dados correspondentes do group_con_map
        if (isset($con->group_id)) {
            foreach ($con->group_id as $group_id) {
                unset($group_con_map[$group_id][$con->id]);
                if (empty($group_con_map[$group_id])) {
                    unset($group_con_map[$group_id]);
                }
            }
        }
    };

    Worker::runAll();
```

## Teste (supondo que todas as execuções são locais em 127.0.0.1)
1. Execute o servidor
``` 
php start.php start
Workerman[del.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Versão do Workerman: 3.4.2          Versão do PHP: 7.1.3
------------------------ TRABALHADORES -------------------------------
usuário        trabalhador     ouvir                     processos status
liliang        ChannelServer   frame://0.0.0.0:2206       1         [OK] 
liliang        none            websocket://0.0.0.0:1234   12        [OK] 
----------------------------------------------------------------
Pressione Ctrl-C para sair. Iniciado com sucesso.

```

2. Conectar o cliente ao servidor

Abra o navegador Chrome, pressione F12 para abrir o console de depuração, na guia Console, insira o seguinte (ou coloque o código abaixo em uma página HTML e o execute com JS)

```javascript
// Supondo que o IP do servidor seja 127.0.0.1, altere para o IP real do servidor ao testar
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
    ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"Esta é a mensagem"}');
};

```
