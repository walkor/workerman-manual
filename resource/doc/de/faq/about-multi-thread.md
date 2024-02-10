# Unterstützt Workerman Multithreading?

Workerman verfügt über eine [MT-Multithreading-Version](https://github.com/walkor/workerman-MT), die von der [pthreads-Erweiterung](https://php.net/manual/zh/book.pthreads.php) abhängt. Aufgrund der unzureichenden Stabilität der pthreads-Erweiterung wird die Entwicklung dieser Workerman-Multithreading-Version jedoch nicht mehr fortgesetzt.

**Derzeit basieren Workerman und seine Umgebung auf einem Multiprozess-Einzelthread-Modell.**
