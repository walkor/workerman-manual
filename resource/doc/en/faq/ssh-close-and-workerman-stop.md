# Terminal closure leads to service shutdown
**Question:**

Why does Workerman shut down on its own when I close the terminal?

**Answer:**

Workerman has two startup modes, debug mode and daemon mode.

Running ```php xxx.php start``` enters debug mode, which is used for debugging during development. When the terminal is closed, Workerman will also close.

Running ```php xxx.php start -d``` enters daemon mode, and closing the terminal will not affect Workerman.

If you want Workerman to be unaffected by the terminal, you can use daemon mode for startup.
