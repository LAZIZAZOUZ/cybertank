# Des infos utiles sur Burpsuite

## Le proxy

C'est un peu la base de Burp, on peut configurer un proxy pour intercepter les requêtes HTTP et les modifier avant de les envoyer au serveur. On peut aussi intercepter les réponses du serveur et les modifier avant de les envoyer au client.

Le plus simple est d'utiliser un proxy en extension de son navigateur par exemple **foxy-proxy** pour Firefox ou **SwitchyOmega** pour Chrome. Attention à bien configurer son proxy et son navigateur, il faut penser à ajouter le certificat de Burp dans les certificats de Firefox.

## La target

Une fois le proxy configuré, on peut commencer par mettre le proxy en mode off pour laisser passer toutes les requêtes et se balader sur le site cible. Cela va mapper les différents liens et branches du site dans l'onglet Target. On peut ainsi voir les différents liens et les requêtes HTTP associées.

## Le scoping

Une fois qu'on a fait le tour du site, on peut être amené à lancer le proxy pour intercepter des requêtes. Cependant, c'est très chiant si burp se met à arrêter absolument toutes les requêtes. On peut dans l'onglet target choisir un site ou une branche du site que l'on "ajoute dans le scope". Ca permet de n'intercepter que les requêtes qui sont liées à ce site ou cette branche.
On peut aussi configurer les options du proxy pour n'intercepter que certaines requêtes.

## Le repeater

Permet souvent de tester des injections. On capture une requête avec le proxy et on l'envoie sur le repeater. On peut ensuite modifier une partie de cette requête et la renvoyer au serveur. C'est pratique et rapide car pour chaque requête envoyé, burpsuite sauvegarde la réponse du serveur. On peut donc faire des tests en modifiant la requête et en regardant la réponse du serveur.

## L'intruder

L'intruder permet de faire du brute-force mais il faut noter qu'il est limité en terme de requêtes par seconde avec la version gratuite. Il faut donc utiliser des outils comme **wfuzz** ou **hydra** pour faire du brute-force plus efficacement.

Cependant, son mode de fonctionnement est simple, on envoie une requête HTTP interceptée dans l'intruder et on peut sélectionner les parties de cette requêtes qui doivent être bruteforcés. Cela se fait en mettant des balises de la forme '§' autour des parties à bruteforcer. On peut ensuite choisir un type d'attaque, il y a en 4 :
* sniper
* pitchfork
* cluster bomb
* battering ram

Dans toues les cas, ces attaques utilisent des listes de mots ou payload. Pour sélectionner les listes à utiliser il faut aller dans l'onglet payload et ajouter une liste de mot ou autre (on peut faire du traitement sur la liste comme en ajoutant des suffixes ou en la convertissant en base64 par exemple).

Enfin, pour voir si une attaque a un succès, on peut trier les réponses reçu par code HTTP. Si elles ont toutes le même code, on peut essayer de trier par longueur de la réponse.

### Bruteforce complexe avec cookie CSRF

Brupsuite peut être utile pour faire du bruteforce alors que les requêtes ont un cookie CSRF (Cross-Site Request Forgery). Globalement, c'est un cookie qui est différent pour chaque request et qui modifie aussi le cookie de session. Autrement dit, si on essaye simplement de bruteforce sans modifier ces cookie ça ne marchera pas car les requêtes seront rejetées même si le mot de passe est bon.

#### Etape 1 : Intruder classique

Pour commencer, il faut tout d'abord sélectionner l'endroit où est situé le mdp à bruteforce dans la requête et initialiser l'attaque que l'on veut faire avec intruder basiquement.

#### Etape 2 : Récupérer le cookie CSRF

Pour récupérer le cookie, on va créer une macro qui va venir récupérer la valeur du cookie de session et celui de CSRF. Il faut aller dans l'onglet macro et créer une nouvelle macro. Cela ouvre une fenêtre avec toutes les requêtes précédemment effectuées. Il faut selectionner une requête GET de la page de connexion qu'on essaye de bruteforce. Et c'est tout !

En gros, on va venir créer une nouvelle session à chaque nouveau mot de passe testé. Cela permet de contourner le cookie CSRF.

#### Etape 3 : Traiter la macro pour l'injecter dans l'attaque

Une fois que la macro est créée, il faut pouvoir injecter les données qu'elle contient dans la requête que l'on va envoyée avec intruder. Pour faire ça, on va aller dans l'onglet règle de traitement de session (Session Handling rules). On va ajouter une nouvelle règle. Dans la partie "Scope", on vient sélectionner uniquement intruder (car on veut que la règle ne s'applique que sur intruder). Et pour la partie URL scope on va venir cocher "Use suite scope" (mais faut bien au préalable avoir sélectionné le site cible en tant que scope). Dans la partie "Rule Actions", on vient sélectionner "Run a macro" et on choisit la macro que l'on vient de créer. 

Il faut à présent dire quelle partie de la requête de la macro va être utilisée. On sélectionne "update only the following parameters" et on rentre le nom qu'a le token dans la requête. Pour mon cas c'était "loginToken". Et dans la partie cookie, on sélectionne aussi "update only the following cookies" et on met le nom du cookie. Dans mon cas c'était "session". 

On sauvegarde et voilà ! On peut lancer l'attaque.

PS : Si les réponses de l'attaque sont des erreurs 403 c'est qu'il y a un pb. Il faut vérifier que la règle de traitement de session est bien appliquée à intruder et que le scope est bien configuré.
