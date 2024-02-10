# O Workerman suporta múltiplas threads?

O Workerman tem uma versão de múltiplas threads (MT) que depende da extensão [pthreads](https://php.net/manual/zh/book.pthreads.php), mas devido à instabilidade da extensão pthreads, esta versão do Workerman de múltiplas threads não está mais sendo mantida.

**Atualmente, o Workerman e seus produtos associados são baseados em múltiplos processos single-thread.**
