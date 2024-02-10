# Comment intégrer avec d'autres frameworks

**Question :**

Comment intégrer avec d'autres frameworks MVC tels que ThinkPHP, Yii, etc. ?

**Réponse :**

![workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

Il est **recommandé** de combiner avec d'autres frameworks MVC comme illustré dans le schéma ci-dessus (en prenant l'exemple de ThinkPHP) :

1. ThinkPHP et Workerman sont deux systèmes indépendants, déployés séparément (peuvent être déployés sur des serveurs différents) sans interférence mutuelle.

2. ThinkPHP fournit des pages web qui sont rendues et affichées dans le navigateur via le protocole HTTP.

3. Le Javascript des pages fournies par ThinkPHP établit une connexion websocket avec Workerman.

4. Une fois la connexion établie, ThinkPHP envoie un paquet de données à Workerman (contenant un nom d'utilisateur, un mot de passe ou un jeton quelconque) pour authentifier la connexion websocket appartenant à un utilisateur spécifique.

5. Workerman n'appelle l'interface socket que lorsque ThinkPHP a besoin de pousser des données vers le navigateur.

6. Toutes les autres requêtes sont traitées selon la méthode HTTP originale de ThinkPHP.

**En résumé :**

Workerman est utilisé comme un canal permettant de pousser des données vers le navigateur, et il n'est appelé que lorsqu'il est nécessaire de pousser des données vers le navigateur. Toutes les logiques métier sont gérées dans ThinkPHP.

Pour savoir comment ThinkPHP appelle l'interface socket de Workerman pour pousser des données, référez-vous à la section "FAQ - Pousser des données dans d'autres projets" dans [ThinkPHP5 manual](https://www.kancloud.cn/manual/thinkphp5/235128).

**ThinkPHP prend déjà en charge Workerman, voir la documentation officielle de ThinkPHP5 : [ThinkPHP5 manual](https://www.kancloud.cn/manual/thinkphp5/235128)**
