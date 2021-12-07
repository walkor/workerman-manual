# Worker

Worker{

public int [count](./count.md);

public string [name](./name.md);

public string [user](./user.md);

public bool [reloadable](./reloadable.md);

public string [transport](./transport.md);

public array [connections](./connections.md);

public static bool [deamonize](./daemonize.md);

public static string [stdoutFile](./stdout_file.md);

public satic string [pidFile](./pid_file.md);

public static EventInterface [globalEvent](./global-event.md);

// callbacks

public callback [onWorkerStart](./on_worker_start.md);

public callback [onWorkerStop](./on-worker-stop.md);

public callback [onConnect](./on-connect.md);

public callback [onMessage](./on-message.md);

public callback [onClose](./on-close.md);

public callback [onBufferFull](./on-buffer-full.md);

public callback [onbufferDrain](./on-buffer-drain.md);

public callback [onError](./on-error.md);

}
