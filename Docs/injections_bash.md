# Les injections en bash

## Introduction

L'injection de commande se produit lorsque le code côté serveur (comme PHP) dans une application web fait un appel à une fonction qui interagit directement avec la console du serveur. Une vulnérabilité de type injection de commande permet à un pirate de tirer parti de cet appel pour exécuter des commandes du système d'exploitation de manière arbitraire sur le serveur. Les possibilités pour l'attaquant sont infinies : il peut lister des fichiers, lire leur contenu, exécuter des commandes de base pour faire de la reconnaissance sur le serveur ou tout ce qu'il veut, comme s'il était assis devant le serveur et émettait des commandes directement dans la ligne de commande.

L'injection est une vulnérabilité OWASP du top 10 bien connue.

## Les commandes "inline"

### Contexte de l'injection

Imaginons que l'on remarque qu'un code php exécute une commande du type :
```php 
<?php
passthru("perl /usr/bin/cowsay -f $variable_de_lutilisateur");
?>
```

La fonction `passthru` exécute simplement une commande dans la console du système d'exploitation et renvoie la sortie au navigateur de l'utilisateur. Vous pouvez voir que notre commande est formée par la concaténation de la variables $mooing à la fin de celle-ci. Puisque nous pouvons manipuler cette variable, nous pouvons essayer d'injecter des commandes supplémentaires en utilisant des astuces simples. 

### Injection de commandes

Maintenant que nous savons comment l'application fonctionne , nous allons tirer parti d'une fonctionnalité de Bash appelée "commandes en ligne" pour abuser du serveur Cowsay et exécuter n'importe quelle commande arbitraire. Bash vous permet d'exécuter des commandes à l'intérieur de commandes. Cette fonctionnalité est utile pour de nombreuses raisons, mais dans notre cas, elle sera utilisée pour injecter une commande dans le serveur Cowsay afin de la faire exécuter.

Pour exécuter des commandes inline, il suffit de les enfermer dans le format suivant `$(votre_commande_ici)`. Si la console détecte une commande inline, elle **l'exécutera d'abord** et utilisera ensuite le résultat comme paramètre de la commande externe. 

Les commandes les plus pratiques à injecter peuvent être trouvées dans l'onglet [commandes utiles](https://lazizazouz.github.io/cybertank/linux_commandes_utiles/).


