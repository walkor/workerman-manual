# workerman example not working
## Symptom
Workerman has been properly started, but the examples provided on the official website or the downloaded demo do not work, for example, the page cannot be opened or socket connection fails.

## Solution
Generally, when Workerman starts without errors but the page cannot be opened or the connection fails, it is usually caused by the server firewall. Please disable the server's firewall first and then run the tests. If it is confirmed to be a firewall issue, please reset the firewall rules.
