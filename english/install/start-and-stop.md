# Start and Stop

Please note that Workerman start and stop commands are executed in the command line.

To start Workerman, you first need an entry file that defines the port and protocol for the service to listen on. You can refer to the [Getting Started - Simple Development Example](../getting-started/simple-example.md) for more information.

Using [workerman-chat](https://www.workerman.net/workerman-chat) as an example, its entry point is start.php.

### Start

Start in debug mode

```php start.php start```

Start in daemon mode

```php start.php start -d```

### Stop

```php start.php stop```

### Restart

```php start.php restart```

### Graceful Restart

```php start.php reload```

### View Status

```php start.php status```

### View Connection Status (requires Workerman version >=3.5.0)

```php start.php connections```

## Difference between debug and daemon modes

1. When started in debug mode, echo, var_dump, print and similar output functions in the code will be printed directly to the terminal.

2. When started in daemon mode, echo, var_dump, print and similar output will be redirected by default to the /dev/null file. This behavior can be changed by setting ```Worker::$stdoutFile = '/your/path/file';``` to specify the file path.

3. When started in debug mode, Workerman will shut down and exit when the terminal is closed.

4. When started in daemon mode, Workerman will continue to run in the background even after the terminal is closed.

## What is Graceful Restart?

Please refer to [Graceful Restart Principle](../faq/reload-principle.md).
