# Fechamento do terminal resulta no encerramento do serviço
**Pergunta:**

Por que o Workerman se encerra quando fecho o terminal?

**Resposta:**

O Workerman possui dois modos de inicialização, modo de depuração debug e modo de processo daemon.  
Executar o comando ```php xxx.php start``` inicia o modo de depuração debug, usado para desenvolver e depurar problemas. Quando o terminal é fechado, o Workerman será encerrado.

Executar o comando ```php xxx.php start -d``` inicia o modo de processo daemon, onde o fechamento do terminal não afeta o funcionamento do Workerman.

Se desejar que o Workerman não seja afetado pelo fechamento do terminal, é possível iniciar em modo daemon.
