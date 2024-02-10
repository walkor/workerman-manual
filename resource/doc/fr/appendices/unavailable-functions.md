# Fonctions non prises en charge

Fonction/Expression non prise en charge | Solution de rechange | Explication
----|------|----
pcntl_fork | Définir le nombre de processus à l'avance | 
php://input | [`$request->rawBody()`](http/request.md) | Utilisé pour récupérer les données brutes POST dans une application utilisant le protocole HTTP
exit | return | Utiliser exit entraînera la sortie du processus. Pour renvoyer une valeur, utilisez directement l'instruction return
die | return | Utiliser die entraînera la sortie du processus. Pour renvoyer une valeur, utilisez directement l'instruction return
Fonctions liées aux en-têtes, aux cookies et aux sessions | Voir les classes [`$request`](http/request.md) et [`$response`]([http/response.md) | 
set_time_limit | N/A | Peut seulement être défini à 0, sinon cela causera la sortie du processus Workerman après un certain temps
