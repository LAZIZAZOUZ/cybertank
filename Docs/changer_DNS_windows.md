# Changement de DNS windows

## Connaitre son DNS actuel 
``` cmd
ipconfig /all | findstr "Server DNS"
```
Pour trouver le DNS rapidement. 
Personnellement ça me met l'IP de ma box ce qui est normal car mes requêtes sont envoyées à ma box qui renvoie vers le DNS de son choix et ça ça me plait pas. 


## En utilisant le cmd

Connaitre le nom de l'interface réseau pour laquelle on va changer le DNS
``` bash
netsh interface show interface
```
Personnellement c'est Ethernet

Du coup on va changer le serveur DNS du protocol IPv4 et IPv6 séparément. 

### IPv4 

Je commence par IPv4 et je vais mettre le DNS public google : *8.8.8.8*  (principal) et *8.8.4.4* (secoure), on va le faire en deux fois :

/!\ il faut lancer le terminal en administrateur
``` bash
netsh interface ipv4 set dnsservers "Nom interface réseau" static [Adresse IP du serveur DNS primaire] primary
```

``` bash
netsh interface ipv4 add dnsservers "Nom interface réseau" [Adresse IP du serveur DNS secondaire] index=2
```

### IPv6

C'est la même chose mais avec des adresses IPv6 : *2001:4860:4860::8888* (principale sur le port 8888) et secondaire : *2001:4860:4860::8844*

``` bash
netsh interface ipv6 set dnsservers "Nom interface réseau" static [Adresse IP du serveur DNS primaire] primary
```

``` bash
netsh interface ipv6 add dnsservers "Nom interface réseau" [Adresse IP du serveur DNS secondaire] index=2
```

On peut vérifier que ça a bien marché : 

### Connaitre le DNS d'une interface réseau particulière pour un protocole particulier 

``` bash
netsh interface Nom_protocole show dnsservers "Nom interface"
``` 

## Remettre le DNS de base 

On peut réatribuer le DNS de base qui sera attribué automatiquement via le DHCP
``` bash
netsh interface ipv4 set dnsservers "Nom de l'interface réseau" dhcp
```