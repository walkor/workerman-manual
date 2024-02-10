# Workerman supporta il multithreading?

Workerman ha una versione [MT multithread](https://github.com/walkor/workerman-MT) che dipende dall'estensione [pthreads](https://php.net/manual/zh/book.pthreads.php), ma poiché l'estensione pthreads non è ancora sufficientemente stabile, questa versione multi-thread di Workerman non è più mantenuta.

**Attualmente Workerman e i suoi prodotti correlati si basano su processi multipli a singolo thread.**
