# ¿Workerman soporta multihilos?

Workerman tiene una versión de varios hilos [MT](https://github.com/walkor/workerman-MT) que depende de la extensión [pthreads](https://php.net/manual/zh/book.pthreads.php), pero debido a que la extensión pthreads aún no es lo suficientemente estable, la versión de Workerman de varios hilos ya no se mantiene.

**Actualmente, Workerman y sus productos periféricos se basan en procesos múltiples de un solo hilo.**
