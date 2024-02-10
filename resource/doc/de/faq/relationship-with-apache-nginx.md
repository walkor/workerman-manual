# Die Beziehung zu Apache und Nginx
**Frage:**

Was ist die Beziehung zwischen Workerman und Apache/nginx/php-fpm? Gibt es Konflikte zwischen Workerman und Apache/nginx/php-fpm?

**Antwort:**

Workerman hat keine direkte Beziehung zu Apache/nginx/php-fpm und ist nicht von ihnen abhängig. Sie sind unabhängige Container, die sich nicht gegenseitig beeinflussen, solange sie nicht denselben Port überwachen.

Workerman ist ein allgemeines Socket-Server-Framework, das Langverbindungen und verschiedene Protokolle wie HTTP, WebSocket und benutzerdefinierte Protokolle unterstützt. Apache/nginx/php-fpm werden im Allgemeinen nur für die Entwicklung von Webprojekten mit dem HTTP-Protokoll verwendet.

Wenn ein Server bereits Apache/nginx/php-fpm implementiert hat, wird die Bereitstellung von Workerman den Betrieb dieser nicht beeinträchtigen.
