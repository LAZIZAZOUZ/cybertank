# Cracker le mdp d'un wifi

## Les types de wifi :

Il ya 3 types de réseau wifi : 

### WEP 

Les réseaux WEP pour Wired Equivalent Privacy ne sont normalement plus utilisés car ils a été prouvé qu'ils ont une faille de sécurité qui permet, après avoir capturé assez de paquets, de faire une attaque statistique pour déterminer le mdp.

### WPA2-PSK

Les réseaux Wifi auxquels vous vous connectez en fournissant un mot de passe qui est le même pour tout le monde

### WPA2-EAP

Réseaux Wifi auxquels vous vous authentifiez en fournissant un nom d'utilisateur et un mot de passe, qui sont envoyés à un serveur RADIUS(Un serveur pour l'authentification des clients, pas seulement pour le wifi).

## Type d'authentification

Les réseaux WPA2 utilise une authentification appelée 4-way handshake. Globalement, sans rentrer dans les détails ça permet au client et à l'AP de prouver qu'ils connaissent la clé, sans se le dire l'un à l'autre. Le WPA et le WPA2 utilisent pratiquement la même méthode d'authentification, de sorte que les attaques sont les mêmes.

## Vulnérabilité

Il n'y en a pas de connu pour le WPA2, il faut donc passer par une attaque bruteforce.
A noter que le nombre de caractère minimal pour un mdp de WPA2 c'est 8.

Les clés du WPA sont dérivées de l'ESSID (Un SSID (Le "nom" du réseau que vous voyez lorsque vous essayez de vous connecter) qui *peut* s'appliquer à plusieurs points d'accès, par exemple un bureau d'entreprise, formant normalement un réseau plus grand. Pour Aircrack, ils font normalement référence au réseau que vous attaquez.) et du mot de passe du réseau. L'ESSID agit comme un sel, ce qui rend les attaques par dictionnaire plus difficiles. Cela signifie que pour un mot de passe donné, la clé variera toujours pour chaque point d'accès. Cela signifie qu'à moins de calculer à l'avance le dictionnaire pour ce seul point d'accès/adresse MAC, vous devrez essayer des mots de passe jusqu'à ce que vous trouviez le bon.

## Outil utilisé

Pour les réseaux wifi on va utiliser l'outil aircrack-ng pour le bruteforce : [site de l'outil](https://aircrack-ng.org)

