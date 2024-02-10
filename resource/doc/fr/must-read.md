# Quelques points importants que les développeurs Workerman doivent connaître
**1. Limites de l'environnement Windows**

Sous le système Windows, un seul processus Workerman prend en charge uniquement 200+ connexions.
Il n'est pas possible d'utiliser le paramètre "count" pour définir plusieurs processus sous Windows.
Les commandes telles que "status", "stop", "reload", "restart" ne sont pas disponibles sous Windows.
Les processus ne peuvent pas être démonisés sous Windows ; lorsque la fenêtre cmd est fermée, le service s'arrête.
Il n'est pas possible d'initialiser plusieurs écoutes dans un seul fichier sous Windows.
Ces limitations ne sont pas applicables sous Linux. Il est recommandé d'utiliser le système Linux pour un environnement en production, tandis que le développement peut se faire sous Windows.

**2. Workerman ne dépend pas d'Apache ou de Nginx**

Workerman est en soi un conteneur similaire à Apache/Nginx, tant que l'environnement PHP est OK, Workerman peut fonctionner.

**3. Workerman est démarré en ligne de commande**

La méthode de démarrage est similaire à celle d'Apache (généralement, l'espace Web ne prend pas en charge Workerman). L'interface de lancement est similaire à celle ci-dessous : 
![](image/screenshot_1495622774534.png)

**4. Les connexions longues doivent être accompagnées de pulsations cardiaques**

Les connexions longues nécessitent des pulsations cardiaques. Il est important de le répéter trois fois. Si une communication n'a pas lieu sur une longue période, le nœud de routage peut nettoyer la connexion et la fermer. Pour plus d'informations sur les pulsations cardiaques, consultez [l'explication sur les pulsations cardiaques de Workerman](faq/heartbeat.md) et [l'explication sur les pulsations cardiaques de GatewayWorker](https://www.workerman.net/doc/gateway-worker/heartbeat.html).

**5. Le protocole côté client et serveur doit correspondre pour communiquer**

C'est un problème très commun pour les développeurs. Par exemple, si le client utilise le protocole websocket, le serveur doit également utiliser le protocole websocket (le serveur doit être initié par ```new Worker('websocket://0.0.0.0...')```) pour pouvoir se connecter et communiquer. Il ne faut pas essayer d'accéder au port du protocole websocket dans la barre d'adresse du navigateur, ni essayer d'utiliser le protocole websocket pour accéder au port du protocole TCP nu. Les protocoles doivent correspondre.

Ce principe est similaire à celui de la communication avec un Anglais en utilisant l'anglais, ou avec un Japonais en utilisant le japonais. Les langages sont similaires aux protocoles de communication, les deux parties (client et serveur) doivent utiliser la même langue pour communiquer, sinon aucune communication n'est possible.

**6. Raisons possibles de l'échec de la connexion**

Lors de l'utilisation initiale de Workerman, un problème courant est l'échec de la connexion du client au serveur. Les raisons courantes sont les suivantes :
1. Le pare-feu du serveur (y compris le groupe de sécurité du serveur cloud) bloque la connexion (50% de chances).
2. Le client et le serveur utilisent des protocoles différents (30% de chances).
3. L'adresse IP ou le port sont mal renseignés (15% de chances).
4. Le serveur n'est pas en cours d'exécution.

**7. Ne pas utiliser les instructions exit die sleep**

L'utilisation de l'instruction d'arrêt die peut entraîner la sortie du processus et afficher l'erreur WORKER EXIT UNEXPECTED. Bien entendu, lorsque le processus se ferme, un nouveau processus redémarre immédiatement pour continuer à fournir le service. Pour revenir, l'instruction return peut être utilisée. L'instruction sleep permet au processus de dormir, pendant lequel aucune activité n'est exécutée et le cadre s'arrête également, empêchant ainsi le traitement de toutes les demandes de clients par ce processus.

**8. Ne pas utiliser la fonction pcntl_fork**

`pcntl_fork` est utilisé pour créer dynamiquement de nouveaux processus. Si `pcntl_fork` est utilisé dans le code métier, il peut entraîner la création de processus orphelins non récupérables, provoquant ainsi des perturbations dans le métier. L'utilisation de `pcntl_fork` dans le métier affectera également le traitement des événements tels que les connexions, les messages, la fermeture des connexions, les minuteries, etc., provoquant des dysfonctionnements imprévisibles.

**9. Ne pas inclure de boucle infinie dans le code métier**

Il ne faut pas inclure de boucle infinie dans le code métier, sinon le contrôle ne pourra pas être restitué au cadre Workerman et entraînera l'impossibilité de recevoir et traiter d'autres messages des clients.

**10. Redémarrer pour appliquer les modifications de code**

Workerman est un cadre à mémoire résidente. Pour voir les effets du nouveau code, il faut redémarrer Workerman.

**11. Recommandation d'utilisation du cadre GatewayWorker pour les applications de connexion longue**

De nombreux développeurs utilisent Workerman pour développer des applications de **connexion longue**, telles que la communication en temps réel, l'Internet des objets, etc. Il est recommandé d'utiliser directement le cadre GatewayWorker pour de telles applications de **connexion longue**. Il est spécialement conçu pour faciliter et rendre plus convivial le développement des arrière-plans d'applications de connexion longue sur la base de Workerman.

**12. Prend en charge une plus grande concurrence**
Si le nombre de connexions concurrentes de l'entreprise dépasse 1000 en ligne simultanément, il est recommandé de [optimiser le noyau Linux](appendices/kernel-optimization.md) et d'installer [l'extension event](appendices/install-extension.md).
