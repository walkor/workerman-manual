# Example 1
**``` (Requires Workerman version >= 3.3.0) ```**

Multi-process group push system based on Worker

``` php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // Global group to connection mapping array
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // Connect to the Channel server as a Channel client
        Channel\Client::connect('127.0.0.1', 2206);

        // Listen for global group message sending events
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
        // Join a group message {"cmd":"add_group", "group_id":"123"}
        // or broadcast message {"cmd":"send_to_group", "group_id":"123", "message":"这个是消息"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // Join group
            case "add_group":
                global $group_con_map;
                // Add the connection to the corresponding group array
                $group_con_map[$group_id][$con->id] = $con;
                // Record which groups this connection has joined, for cleaning up group_con_map data corresponding to the groups in onclose
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // Broadcast message to the group
            case "send_to_group":
                // Channel\Client broadcasts the group sending message event to all servers and processes
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // Very important, delete the connection from the global group data when the connection is closed to avoid memory leaks
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // Traverse all groups joined by the connection, delete the corresponding data from group_con_map
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
  
## Test (Assuming all running on local machine 127.0.0.1)
1. Run the server
```  
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

2. Client connects to the server.  

Open the Chrome browser, press F12 to open the developer tools, and in the Console tab input (or put the following code into an HTML page and run it with JavaScript)  

```javascript
// Assume the server ip is 127.0.0.1, please change to the actual server ip when testing
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"这个是消息"}');
};
```
