# Les bases de Nmap : des fonctionnalités utiles

## Enregistrer le résultat
ça peut être pratique d'enregistrer le résultat d'un scan.

L’option ```-oN``` permet d’enregistrer le résultat du portscan dans un fichier au format texte :
```
nmap -oN output.txt 192.168.1.1
```

Pour enregistrer le résultat du scan de port dans un fichier au format XML :
```
nmap -oX output.xml 192.168.1.1
```

## Analyser un hôte ou un réseau 

#### Scanner plusieurs hôtes 

En paramètre : 
```
nmap [ip1] [ip2] [ip3]
```

Avec une liste prédéfinie : 
```
nmap -iL chemin/liste_ip.txt
```

On peut directement mettre une adresse web en paramètre :
```
nmap www.google.com
```

#### Exclure  IP ou Hôtes ou Réseaux 
En paramètre :
```
nmap 192.168.1.0/24 --exclude 192.168.1.1
nmap 192.168.1.0/24 --exclude 192.168.1.1 192.168.1.5
nmap 192.168.1.0/24 --exclude 192.168.1.1,2,3
```
Depuis une liste : 
```
nmap 192.168.1.0/24 --excludefile exclusion.txt
```

#### Scanner des ports en particulier 

Il faut préciser le type de protocole utilisé par le port : T pour TCP/IP, U pour UDP 
Exemple : 
```
nmap -p T:80 [ip]
nmap -p U:53 [ip]
nmap -p T:1-1024 [ip]
nmap -p U:53,9,113,T:21-25,80,443,8080 [ip]
```

On peut obtenir toutes les infos possibles sur un port en particulier en lançant la commande :
```
nmap -p [port] -A [ip]
```
**Attention :** cette commande est longue à s'exécuter donc il faut bien la limiter à un seul et unique port.


#### Recherche numéro de version 
On peut trouver le numéro de version du service qui s'exécute sur des hôtes avec :
```
nmap -sV [ip]
```
#### Trouver l'OS 
En utilisant l'option ```-O```
#### Désactiver les requêtes ping 
C'est très utile car certains parefeu bloque par défaut toutes les requêtes ICMP (ping) et nmap ping par défaut donc une cible peut être enregistrée comme down juste à cause de ça.
```
nmap -Pn [ip]
```
#### Désactiver la résolution DNS
```
nmap -n [ip]
```
#### Changer de type de scan 

L’option ```-P``` permet de changer le type de scan :
```-Pn```: Traitez tous les hôtes comme en ligne – ignorez la découverte des hôtes
```-PS/PA/PU/PY[portlist]```: TCP SYN/ACK, UDP or SCTP découverte vers des ports donnés
```-PE / PP / PM```: sondes de découverte d’écho, d’horodatage et de demande de masque de réseau ICMP
```-PO [liste de protocoles]```: Ping de protocole IP

Exemples : 
Analyser les hôtes distants à l’aide de TCP ACK (PA) et TCP Syn (PS) :
```
nmap -PS 192.168.1.1
```

Analyser l’hôte distant pour des ports spécifiques avec TCP ACK :
```
nmap -PA -p 22,80 192.168.1.1
```

## Utiliser des scripts 

Le Nmap Scripting Engine (NSE) est un ajout incroyablement puissant à Nmap, qui étend considérablement ses fonctionnalités. Les scripts NSE sont écrits dans le langage de programmation Lua, et peuvent être utilisés pour faire une variété de choses : de l'analyse des vulnérabilités, à l'automatisation des exploits pour ces vulnérabilités. Le NSE est particulièrement utile pour la reconnaissance, mais il convient de garder à l'esprit l'étendue de la bibliothèque de scripts.

De nombreuses catégories sont disponibles. Parmi les catégories utiles, on peut citer

* ```safe```: n'affectera pas la cible
* ```intrusive``` : pas sûr : susceptible d'affecter la cible
* ```vuln``` : recherche de vulnérabilités
* ```exploit```: Tentative d'exploitation d'une vulnérabilité
* ```auth```: Tentative de contournement de l'authentification pour les services en cours d'exécution (par exemple, connexion anonyme à un serveur FTP)
* ```brute```: Tentative de forcer les informations d'identification des services en cours d'exécution.
* ```discovery```: Tenter d'interroger les services en cours d'exécution pour obtenir des informations supplémentaires sur le réseau (par exemple, interroger un serveur SNMP).
  
Une liste plus exhaustive est disponible [ici](https://nmap.org/book/nse-usage.html).

#### Scanner les vulnérabilités

Il existe un script prédéfini présent dans la commande dans Nmap qui permet aux utilisateurs d’exécuter un scan de vulnérabilités. Cela est donc très pratique pour s’assurer que votre système est à jour et non vulnérable.
On peut utiliser ces scripts prédéfinis ou posséder leur langage de programmation Lua pour dériver une fonctionnalité spécifique qui peut aider à la détection CVE.
Pour cela, on utilise l’option ```-script vuln``` :
```
nmap -Pn –script vuln [ip]
```

On peut simplement utiliser le vérificateur de logiciels malveillants Google SafeBrowsing par la commande:
```
nmap -p80 –script http-google-malware www.malekal.com
```

#### Pour aller plus loin 

On peut utiliser un script disponible sur git pour avoir plus de vulnérabilités checkés.

Etape 1 : Se mettre dans le répertoire :
```
cd /usr/share/nmap/scripts
```
Etape 2 : Cloner le repo :
```
git clone https://github.com/vulnersCom/nmap-vulners.git
```
Etape 3 : Utiliser vulners:
```
nmap -sV --script nmap-vulners/ [ip cible]
```


## Lancer une attaque brutforce

On utilise l’option ```-script``` pour spécifier le type d’attaque.

```
nmap -sV –script http-wordpress-brute –script-args ‘userdb=users.txt,passdb=passwds.txt,http-wordpress-brute.hostname=domain.com, http-wordpress-brute.threads=3,brute.firstonly=true’ [ip]
```

#### Contre MS-SQL

```
nmap -p 1433 –script ms-sql-brute –script-args userdb=customuser.txt,passdb=custompass.txt [ip]
```

### Contre FTP

```
nmap –script ftp-brute -p 21 192.168.1.105
```

## Contourner un firewall tips

#### Utiliser les switch ```-sN```, ```-sF``` ou ```-sX```

Se référer au document *La base de Nmap : les scans*

#### Bloquer les requêtes ICMP

Votre hôte Windows typique, avec son pare-feu par défaut, bloque tous les paquets ICMP. Cela pose un problème : non seulement nous utilisons souvent ping pour établir manuellement l'activité d'une cible, mais Nmap fait la même chose par défaut. Cela signifie que Nmap enregistrera un hôte avec cette configuration de pare-feu comme mort et ne prendra pas la peine de le scanner.

Nous avons donc besoin d'un moyen de contourner cette configuration. Heureusement, Nmap fournit une option pour cela : ```-Pn```, qui dit à Nmap de ne pas prendre la peine d'envoyer un ping à l'hôte avant de le scanner. Cela signifie que Nmap traitera toujours le(s) hôte(s) cible(s) comme étant vivant(s), contournant ainsi le bloc ICMP ; cependant, cela a pour prix de prendre potentiellement beaucoup de temps pour compléter le scan (si l'hôte est vraiment mort, alors Nmap vérifiera et revérifiera chaque port spécifié).

#### Fragmenter le paquet 

```-f``` : Utilisé pour fragmenter les paquets (c'est-à-dire les diviser en petits morceaux), ce qui réduit la probabilité que les paquets soient détectés par un pare-feu ou un système de détection d'intrusion.

```--mtu <nombre>``` : Comme ```-f``` mais avec plus de contrôle. Accepte une taille maximale d'unité de transmission à utiliser pour les paquets envoyés. Il doit s'agir d'un multiple de 8.

#### Ajouter un délai 

```--scan-delay <time>ms``` : Utilisé pour ajouter un délai entre les paquets envoyés. C'est très utile si le réseau est instable, mais aussi pour éviter les déclenchements de pare-feu/IDS basés sur le temps qui peuvent être en place.

#### Faire une mauvaise checksum 

```--badsum``` : Est utilisé pour générer une somme de contrôle invalide pour les paquets. Toute pile TCP/IP réelle laisserait tomber ce paquet, mais les pare-feux peuvent potentiellement répondre automatiquement, sans prendre la peine de vérifier la somme de contrôle du paquet. Ainsi, ce commutateur peut être utilisé pour déterminer la présence d'un pare-feu/IDS.

## Autres options utiles

* ```–exclude``` : Exclure des hôtes du scan
* ```-n``` : Désactiver la résolution DNS
* ```–open``` : Afficher que les ports ouverts
* ```-oN``` : Enregistrer le résultat du scan dans un fichier au formate texte
* ```-oX``` :	Enregistrer le résultat du scan dans un fichier au formate XML
* ```-p``` : Spécifier les ports réseaux à scanner
* ```-Pn``` :	Désactiver la découverte d’hôte
* ```-r``` : Analyser les ports consécutivement
* ```-sT``` :	Faire un scan de port TCP
* ```-sU``` :	Faire un scan de port UDP
* ```-sV``` :	Trouver les versions du service
* ```-script``` :	Utilise un script interne à nmap pour scan de vulnérabilité, bruteforce, etc
* ```-v``` :
* ```-vv``` :	Mode bavard