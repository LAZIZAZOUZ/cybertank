# Metasploit : les bases

## Intro fonctionnement

Metasploit se lance avec la commande ```msfconsole```.

C'est une interface qui permet de lancer des exploits sur une cible. Quand on rentre dans la console metasploit, c'est une console à part du terminal classique.

Les commandes utiles à garder en tête sont ```help ma_commande``` et ```history``` qui permet de lister l'historique des commandes effectuées.


Dans metasploit, il y a des **modules** que l'on sélectionne en utilisant la commande ```use le/chemin/de/mon/module```. Les modules sont un ensemble de code qui va s'exécuter automatiquement. La plupart du temps les modules sont des **exploits**. Ils permettent d'exploiter une faille mais il y a aussi des modules permettant de repérer des vulnérabilités ou autre.

Il faut garder en tête qu'avec metasploit il y a une notion de profondeur. Lorsqu'on utilise un module on est au niveau de ce module uniquement, quand on est dans la payload d'un module et qu'on fait des modifications on est dans la payload précise uniquement, etc. Pour revenir d'un niveau en arrière on peut utiliser la commande : ```back```

## Utiliser une exploit

### Rechercher un module

Il y a un grand nombre de modules présents avec metasploit. Il est donc nécessaire de faire des recherches au sein de ces modules. Pour cela on peut utiliser la commande :
```search ce que je veux chercher```
Il y a plusieurs paramètres possibles avec la fonction search. Plus d'infos en faisant un ```help search```.
Parmis les paramètres utiles il y a le fait de pouvoir sélectionner le rang des résultats. En effet, **chaque exploit à un rang** qui correspond globalement à son taux de réussite. 
On peut aussi sélectionner les modules ayant **la fonctionnalité "check" qui permet de checker si l'exploit est possible** sur la cible sans le lancer pour autant.


### Sélectionner un module

Directement après une recherche on peut faire un ```use [numéro du module dans la recherche]``` ce qui vient sélectionner le module souhaité. Ou alors on peut marquer  le nom complet du module après le use.

### Paramétrer le module

Avant de paramétrer le module il vaut mieux comprendre ce qu'il fait et de quels paramètres il a besoin. Pour ça on peut faire un : ```info``` qui nous donne toutes les infos sur le module. Il y a aussi une liste de paramètres qu'on peut entrer en faisant :
```set mon_paramètre sa_valeur```

**Attention !** le paramètre sera sauvegardé en tant que variable local de ce module. Si on change de module il faudra refaire les paramètres.

Pour faire des **paramètres globaux** il faut utiliser :
```setg mon_paramètre_global sa_valeur```

On peut faire un ```show options``` pour voir si les paramètres ont bien été mis correctement

On peut supprimer un paramètre local avec la commande ```unset nom_du_paramètre```. On peut supprimer tous les paramètres locaux avec ```unset all```. Pour faire de même avec les paramètre globaux : ```unsetg```

### Trouver des payloads

Une fois que l'exploit sera lancé, une première faille sera créée et ça va ouvrir une porte appelée **session**. Suite à quoi on peut envoyer des **payloads** qui sont des morceaux de code qui vont s'exécuter sur la machine cible. 
Pour trouver des payloads compatibles avec le module utilisé, on peut faire un :
```show payloads```

### Lancer l'exploit 

Avant de lancer l'exploit, si le module le permet, on peut checker si l'exploit est faisable sur la cible avec :
```check```

Pour lancer une exploit il suffit de taper la commande ```exploit``` ou ```run```.

Si on rajoute l'option ```exploit -z``` ça va lancer l'exploit et mettre la session ouverte en arrière plan.

### Gérer les sessions

Une fois l'exploit réussi, cela ouvre une session. On peut continuer sur cette session directement ou la mettre en arrière plan avec la commande :
```background```

Pour lister les sessions actives en arrière plan on peut faire un :
```sessions```

Pour intéragir avec une session :
```session -i [numéro de la sesssion]```