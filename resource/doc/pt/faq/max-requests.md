# Como configurar o Workerman para reiniciar o processo atual após um número específico de solicitações
Para tornar o Workerman mais compacto, não fornece diretamente essa configuração, mas você pode implementar essa funcionalidade com algumas linhas de código.

```php
$worker->onMessage = function($connection, $data) {
    static $request_count;
    // Processamento de negócios omitido
    if(++$request_count > 10000) {
        // Após 10000 solicitações, encerrar o processo atual, o processo principal automaticamente reiniciará um novo processo
        Worker::stopAll();
    }
};
```
