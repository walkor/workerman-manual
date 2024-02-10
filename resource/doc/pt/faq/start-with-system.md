## Como iniciar automaticamente o Workerman ao ligar o sistema no Linux

Abra o arquivo /etc/rc.local e adicione o seguinte código antes de "exit 0":

```bash
ulimit -HSn 102400
/usr/bin/env php /caminho/para/o/arquivo/start.php start -d

exit 0
```
