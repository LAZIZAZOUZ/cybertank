# L'écoute de ports

## Méthode NETCAT

Pour écouter un port on peut utiliser la commande :
```bash
nc -lvp [port]
```
* TAG | Description
* --- | ---
* ```-l``` | Ecouter
* ```-v``` | Verbose
* ```-p``` | Port

On peut aussi écouté un port TCP ou UDP en rajoutant ```-u``` ou ```-t```.

#### Exemple : Ecouter un port UDP

Pour écouter un port UDP on peut utiliser la commande :
```bash
nc -l -u -p [port]
```

## Méthode TCPDUMP

Pour écouter un port on peut utiliser la commande :
```bash
sudo tcpdump port [port]
```
TAG | Description
--- | ---
```port``` | Filtre port

#### Exemple : Ecouter un port pour une requête ICMP

Pour écouter les requêtes ICMP on peut utiliser la commande :
```bash
sudo tcpdump ip proto \\icmp -i tun0
```
TAG | Description
--- | ---
```ip proto \\icmp``` | Filtre ICMP
```-i``` | Interface


## Envoyer un fichier

Pour envoyer un fichier on peut utiliser la commande :
```bash
nc [ip] [port] < [fichier]
```
TAG | Description
--- | ---
```<``` | Redirection

## Récupérer un fichier

Pour récupérer un fichier on peut utiliser la commande :
```bash
nc -lvp [port] > [fichier]
```
TAG | Description
--- | ---
```>``` | Redirection

## Récupérer un shell

Pour récupérer un shell on peut utiliser la commande :
```bash
nc -lvp [port] -e /bin/bash
```
TAG | Description
--- | ---
```-e``` | Executer



