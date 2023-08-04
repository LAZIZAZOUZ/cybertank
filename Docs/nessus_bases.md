# Nessus 

## Intro 

Nessus est un scanner de vulnérabilités. Il permet de scanner un réseau et de trouver les failles de sécurité.
Il utilise des techniques similaires à Nmap pour trouver et signaler les vulnérabilités, qui sont ensuite présentées dans une interface graphique agréable à consulter.
Nessus est différent des autres scanners car il ne fait pas de suppositions lors de l'analyse, comme supposer que l'application web fonctionne sur le port 80 par exemple.

Nessus propose un service gratuit et un service payant, dans lequel certaines fonctionnalités sont exclues du service gratuit pour vous inciter à acheter le service payant.
Le prix est similaire à celui de Burp Suite, donc à moins que vous n'ayez un peu d'argent, nous utiliserons la version gratuite.

Les prix sont ici : [https://www.tenable.com/products/nessus](https://www.tenable.com/products/nessus)

## Installation

Pour installer Nessus, rendez-vous sur le site officiel de Nessus et téléchargez la version appropriée pour votre système d'exploitation. Il faut aussi s'enrigistrer sur le site : [nessus login](https://www.tenable.com/products/nessus/nessus-essentials)



## Démarrage de Nessus

Pour démarrer Nessus, il suffit de lancer le service nessusd.service. Pour moi, c'est dans le dossier /bin/systemctl. Pour lancer le service, il faut utiliser la commande suivante :

``` bash
sudo /bin/systemctl start nessusd.service
```

Il suffit ensuite de se rendre sur firefox et de se connecter à l'adresse suivante : [https://localhost:8834](https://localhost:8834)

## Fonctionnement de Nessus

Nessus fonctionne en deux parties : un serveur et un client. Le serveur est le service que nous avons démarré précédemment. Le client est l'interface graphique que nous utilisons pour interagir avec le serveur.

Le serveur est responsable de l'analyse des vulnérabilités et le client est responsable de la configuration de l'analyse et de l'affichage des résultats.

## Configuration de Nessus

Pour configurer Nessus, il faut se rendre sur l'interface web et se connecter. Une fois connecté, il faut créer un utilisateur. Cet utilisateur sera utilisé pour se connecter à l'interface web et pour se connecter au serveur Nessus.

Une fois l'utilisateur créé, il faut se connecter à l'interface web avec cet utilisateur. Une fois connecté, il faut créer un scan. Pour créer un scan, il faut cliquer sur le bouton "New Scan" en haut à gauche de l'interface web.


### Configuration du scan

Une fois le scan créé, il faut le configurer. Il faut lui donner un nom, une description et choisir le type de scan que l'on veut effectuer. Il existe plusieurs types de scan, mais nous utiliserons le scan "Basic Network Scan" pour l'instant.

L'option "Policies" permet de créer des modèles personnalisés de scan.

L'option "Plugin Rules" nous permet de modifier les propriétés des plugins, par exemple en les masquant ou en modifiant leur gravité