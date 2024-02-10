# Exemple 1
**``` (Workerman version requise >=3.3.0) ```**

Système de diffusion multi-processus basé sur les Workers

``` php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // Tableau de mappage global des groupes de connexions
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // Le client de canal se connecte au serveur de canal
        Channel\Client::connect('127.0.0.1', 2206);

        // Ecouter l'événement d'envoi de messages de groupe global
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
        // Joindre un message de groupe {"cmd":"add_group", "group_id":"123"}
        // ou envoyer un message de groupe {"cmd":"send_to_group", "group_id":"123", "message":"C'est le message"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // Joindre un groupe de connexion
            case "add_group":
                global $group_con_map;
                // Ajouter la connexion au tableau de groupe correspondant
                $group_con_map[$group_id][$con->id] = $con;
                // Enregistrer à quel groupe cette connexion a rejoint, pour faciliter le nettoyage des données du group_con_map correspondant au groupe lors de la fermeture
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // Envoyer un message à un groupe
            case "send_to_group":
                // Channel\Client diffuser un événement d'envoi de groupe à tous les processus de tous les serveurs
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // Il est très important de supprimer la connexion des données globales de groupe lorsqu'elle est fermée, pour éviter les fuites de mémoire
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // Parcourir tous les groupes auxquels la connexion a été ajoutée et supprimer les données correspondantes de group_con_map
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

## Test (supposons que tout fonctionne sur le même ordinateur 127.0.0.1)
1. Exécution du serveur
```php
php start.php start
Workerman[del.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.4.2          PHP version:7.1.3
------------------------ WORKERS -------------------------------
user          worker         listen                    processes status
liliang       ChannelServer  frame://0.0.0.0:2206       1         [OK] 
liliang       none           websocket://0.0.0.0:1234   12        [OK] 
----------------------------------------------------------------
Appuyez sur Ctrl-C pour quitter. Démarrage réussi.
```

2. Connexion du client au serveur
Ouvrez le navigateur Chrome, appuyez sur F12 pour ouvrir la console de débogage, puis saisissez ce qui suit dans la console (ou insérez le code suivant dans une page HTML et exécutez-le avec JavaScript)

```javascript
// Supposons que l'adresse IP du serveur soit 127.0.0.1, veuillez la remplacer par l'adresse IP réelle du serveur pour les tests
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"C'est le message"}');
};
```
