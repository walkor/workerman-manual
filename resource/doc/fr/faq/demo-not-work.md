# Exemple de Workerman ne fonctionne pas

## Symptômes
Workerman est correctement démarré, mais les exemples fournis sur le site officiel ou les démonstrations téléchargées ne fonctionnent pas, par exemple la page ne s'ouvre pas ou la connexion au socket échoue.

## Solution
En général, lorsque Workerman démarre sans erreur mais que la page ne peut pas être ouverte ou que la connexion échoue, le problème est généralement dû au pare-feu du serveur. Veuillez d'abord désactiver le pare-feu du serveur, puis effectuer un test. Si le problème est confirmé dû au pare-feu, veuillez réinitialiser les règles du pare-feu.
