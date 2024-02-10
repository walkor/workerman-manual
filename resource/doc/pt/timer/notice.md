# Nota
## Considerações sobre o uso de temporizadores
1. Apenas adicione temporizadores no callback ```onXXXX```. Temporizadores globais são recomendados para serem configurados no callback ```onWorkerStart```, e temporizadores para uma conexão específica são recomendados para serem configurados no ```onConnect```.

2. As tarefas agendadas são executadas no processo atual (não inicia novos processos ou threads). Se a tarefa for muito pesada (especialmente se envolver I/O de rede), pode causar bloqueio no processo e impedir temporariamente o processamento de outras tarefas. Portanto, é melhor executar tarefas demoradas em processos separados, por exemplo, estabelecendo um ou mais processos Worker separados.

3. Se o processo atual estiver ocupado com outras tarefas, ou se uma tarefa não for concluída no tempo esperado e for hora de iniciar o próximo ciclo, a execução aguardará a conclusão da tarefa atual antes de prosseguir. Isso pode fazer com que o temporizador não seja executado no intervalo esperado. Ou seja, as tarefas do processo atual são executadas em série, enquanto em vários processos, as tarefas entre os processos são executadas em paralelo.

4. Deve-se ter cuidado ao configurar temporizadores em vários processos, pois pode causar problemas de concorrência. Por exemplo, o seguinte código imprimirá "hi" cinco vezes por segundo:
   ```php
   $worker = new Worker();
   // 5 processos
   $worker->count = 5;
   $worker->onWorkerStart = function(Worker $worker) {
       // 5 processos, cada um com um temporizador como este
       Timer::add(1, function(){
           echo "hi\r\n";
       });
   };
   Worker::runAll();
   ```
   Se desejar que apenas um processo execute o temporizador, consulte o [Exemplo 2 do Timer::add](add.md).

5. Pode haver um erro de aproximadamente 1 milissegundo.

6. Não é possível excluir temporizadores entre processos. Por exemplo, um temporizador configurado no processo A não pode ser excluído diretamente chamando a interface Timer::del no processo B.

7. Os IDs dos temporizadores podem se repetir entre processos diferentes, mas não se repetirão dentro do mesmo processo.

8. Alterar o horário do sistema afetará o comportamento dos temporizadores, portanto, após alterar o horário do sistema, é recomendável reiniciar usando restart.
