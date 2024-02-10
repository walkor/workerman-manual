# Fermeture du terminal entraînant la fermeture du service
**Question :**

Pourquoi Workerman se ferme-t-il automatiquement lorsque je ferme le terminal ?

**Réponse :**

Workerman a deux modes de démarrage, le mode de débogage debug et le mode de processus daemon. 

L'exécution de ```php xxx.php start``` entre en mode debug, utilisé pour le développement et le débogage des problèmes. Lorsque le terminal est fermé, Workerman se ferme également.

L'exécution de ```php xxx.php start -d``` entre en mode daemon, le terminal fermé n'a aucun impact sur Workerman.

Si vous souhaitez que Workerman ne soit pas affecté par la fermeture du terminal, vous pouvez démarrer en mode daemon.
