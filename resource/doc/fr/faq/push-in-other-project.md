# Utiliser Workerman pour envoyer des données aux clients dans d'autres projets

**Question :**

J'ai un projet web ordinaire et je souhaite appeler l'API de Workerman dans ce projet pour envoyer des données aux clients.

**Réponse :**

**Concernant Workerman, vous pouvez consulter les liens suivants :**

- [Exemple de push avec le composant Channel](../components/channel-examples.md) (supporte le multi-processus/cluster de serveurs, nécessite le téléchargement du composant Channel)

- [Push basé sur le Worker](https://www.workerman.net/q/508) (un seul processus, le plus simple)

**En ce qui concerne webman, veuillez consulter le lien suivant :**

- [Plug-in de push webman](https://www.workerman.net/plugin/2)

**Concernant GatewayWorker, veuillez consulter le lien suivant :**

- [Envoyer des push à partir d'autres projets via GatewayWorker](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html) (prise en charge du multi-processus/cluster de serveurs, prise en charge des groupes, des diffusions de groupe et des envois individuels)

**Concernant PHPSocket.IO, veuillez consulter le lien suivant :**

- [Push de messages web](https://www.workerman.net/web-sender) (par défaut, un seul processus, basé sur socket.io, meilleure compatibilité avec les navigateurs)
