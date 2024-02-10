```php
# Ejemplo 1
**``` (Se requiere Workerman versión>=3.3.0) ```**

Sistema de envío multiproceso basado en Worker

```php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // Mapeo global de grupos a conexiones
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // Cliente de Canal se conecta al servidor de Canal
        Channel\Client::connect('127.0.0.1', 2206);

        // Escucha el evento de envío de mensajes de grupo global
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
        // Unirse al mensaje de grupo {"cmd":"add_group", "group_id":"123"}
        // o enviar mensaje {"cmd":"send_to_group", "group_id":"123", "message":"Este es un mensaje"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // Conectar y unirse al grupo
            case "add_group":
                global $group_con_map;
                // Agregar la conexión al arreglo de grupo correspondiente
                $group_con_map[$group_id][$con->id] = $con;
                // Registrar a qué grupos se ha unido esta conexión, para limpiar los datos de group_con_map correspondientes en onclose
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // Enviar mensaje al grupo
            case "send_to_group":
                // Cliente de Canal para distribuir el evento de envío de grupo a todos los procesos de todos los servidores
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // Es importante eliminar la conexión de los datos de grupo globales al cerrarse para evitar fugas de memoria
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // Recorrer todos los grupos a los que se ha unido la conexión y eliminar los datos correspondientes de group_con_map
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

## Prueba (asumiendo que se ejecutan todos en la máquina local 127.0.0.1)
1. Ejecutar el servidor
```bash
php start.php start
Workerman[del.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.4.2          PHP version:7.1.3
------------------------ WORKERS -------------------------------
user          worker         listen                    processes status
liliang       ChannelServer  frame://0.0.0.0:2206       1         [OK] 
liliang       none           websocket://0.0.0.0:1234   12        [OK] 
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

2. Conectar el cliente al servidor

Abrir el navegador Chrome, presionar F12 para abrir la consola de depuración, en la pestaña de "Console" ingresar (o colocar el siguiente código en una página HTML y ejecutarlo mediante JavaScript)

```javascript
// Suponiendo que la IP del servidor es 127.0.0.1, cambiar por la IP real del servidor en caso de prueba
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"Este es un mensaje"}');
};
```
