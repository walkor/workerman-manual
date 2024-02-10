# Iniciar e parar

Observe que os comandos de inicialização e encerramento do Workerman são todos realizados na linha de comando.

Para iniciar o Workerman, você precisa ter um arquivo de entrada que defina a porta e o protocolo para o serviço. Você pode consultar a seção [Exemplos de Desenvolvimento Simples](../getting-started/simple-example.md) para obter mais informações.

Tomando o [workerman-chat](https://www.workerman.net/workerman-chat) como exemplo, o arquivo de entrada de inicialização é start.php.

### Iniciar

Inicie no modo de depuração (debug)

 ```php start.php start```

Inicie no modo daemon

 ```php start.php start -d```

### Parar
 ```php start.php stop```

### Reiniciar
 ```php start.php restart```

### Reinicialização suave
 ```php start.php reload```

### Verificar o estado
 ```php start.php status```
 
### Verificar o estado da conexão (requer Workerman versão >= 3.5.0)
```php start.php connections```



## Diferença entre modos de depuração e daemon

1. Ao iniciar no modo de depuração, as funções de impressão como echo, var_dump, print no código serão exibidas diretamente no terminal.

2. Ao iniciar no modo daemon, as funções de impressão como echo, var_dump, print no código serão redirecionadas por padrão para o arquivo /dev/null e você pode configurar o caminho do arquivo usando ```Worker::$stdoutFile = '/your/path/file';```.

3. Ao iniciar no modo de depuração, o Workerman será encerrado e sairá quando o terminal for fechado.

4. Ao iniciar no modo daemon, o Workerman continuará a funcionar normalmente em segundo plano mesmo após o fechamento do terminal.



## O que é reinicialização suave?

Veja [Princípio de Reinicialização Suave](../faq/reload-principle.md)
