# Notas importantes
## Consideraciones sobre el uso del temporizador
1. Solo se pueden agregar temporizadores en las devoluciones de llamada ```onXXXX```. Se recomienda configurar temporizadores globales en la devolución de llamada ```onWorkerStart```, y temporizadores para una conexión específica en ```onConnect```.

2. Las tareas de temporización agregadas se ejecutan en el proceso actual (no inician un nuevo proceso o hilo). Si la tarea es pesada (especialmente si involucra operaciones de E/S de red), puede provocar bloqueos en el proceso, impidiendo temporalmente que maneje otras operaciones comerciales. Por lo tanto, es mejor ejecutar tareas que consumen mucho tiempo en un proceso separado, como iniciar uno o varios procesos Worker.

3. Si el proceso actual está ocupado con otras operaciones comerciales, o si una tarea no se ha completado en el momento previsto y llega el siguiente período de ejecución, se esperará a que la tarea actual se complete antes de ejecutar la siguiente. Esto puede provocar que el temporizador no se ejecute a intervalos de tiempo esperados. Es decir, las operaciones comerciales en el proceso actual se ejecutan en serie, mientras que en múltiples procesos, las tareas se ejecutan en paralelo.

4. Es importante tener en cuenta que configurar tareas de temporización en varios procesos puede ocasionar problemas de concurrencia. Por ejemplo, el siguiente código imprimirá "hi" 5 veces por segundo.
```php
$worker = new Worker();
// 5 procesos
$worker->count = 5;
$worker->onWorkerStart = function(Worker $worker) {
    // 5 procesos, cada uno con un temporizador similar
    Timer::add(1, function(){
        echo "hi\r\n";
    });
};
Worker::runAll();
```
Si solo se desea que un proceso ejecute el temporizador, consulte [Ejemplo 2 de Timer::add](add.md).

5. Puede haber un margen de error de alrededor de 1 milisegundo.

6. No es posible eliminar temporizadores entre procesos. Por ejemplo, un temporizador configurado en el proceso A no se puede eliminar directamente desde el proceso B utilizando la interfaz Timer::del.

7. Es posible que los identificadores de temporizadores sean repetidos entre diferentes procesos, pero no se repetirán dentro del mismo proceso.

8. Cambiar la hora del sistema afectará el comportamiento de los temporizadores, por lo que se recomienda reiniciar con restart después de cambiar la hora del sistema.
