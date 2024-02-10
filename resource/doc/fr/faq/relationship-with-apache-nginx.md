# Relation avec Apache et Nginx
**Question :**

Quelle est la relation entre Workerman et Apache/nginx/php-fpm ? Y a-t-il des conflits entre Workerman et Apache/nginx/php-fpm ?

**Réponse :**
Workerman n'a aucune relation avec Apache/nginx/php-fpm, et son fonctionnement ne dépend pas d'Apache/nginx/php-fpm. Ils sont tous des conteneurs indépendants qui ne se perturbent pas et ne entrent pas en conflit (tant qu'ils n'écoutent pas le même port).

Workerman est un framework de serveur de socket générique qui prend en charge les connexions persistantes et divers protocoles tels que HTTP, WebSocket et des protocoles personnalisés. En revanche, Apache/nginx/php-fpm est généralement utilisé pour le développement de projets Web utilisant le protocole HTTP.

Si un serveur a déjà été déployé avec Apache/nginx/php-fpm, le déploiement de Workerman n'aura aucun impact sur leur fonctionnement.
