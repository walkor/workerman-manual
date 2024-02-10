# Relação com o Apache e o Nginx

**Pergunta:**

Qual é a relação entre o Workerman e o Apache/nginx/php-fpm? Eles entram em conflito um com o outro?

**Resposta:**

O Workerman não tem nenhuma relação com o Apache/nginx/php-fpm e sua execução não depende do Apache/nginx/php-fpm. Eles são contêineres independentes que operam sem interferir um com o outro e não entram em conflito (desde que não estejam ouvindo na mesma porta).

O Workerman é um framework de servidor de soquetes genérico que oferece suporte a conexões persistentes e vários protocolos, como HTTP, WebSocket e protocolos personalizados. O Apache/nginx/php-fpm, por outro lado, geralmente é utilizado para desenvolver projetos da web que usam o protocolo HTTP.

Se um servidor já estiver implantado com o Apache/nginx/php-fpm, a implantação do Workerman não afetará a operação deles.
