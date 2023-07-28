# Les bases de wireshark

## Introduction

Wireshark est un outil d'analyse de paquets réseau. Il permet de capturer et d'analyser le trafic réseau. Il est disponible sur Windows, Linux et Mac.
* [Site officiel](https://www.wireshark.org/)
* [Download](https://www.wireshark.org/download.html)
* [Documentation](https://www.wireshark.org/docs/)

### Le modèle OSI

Le modèle OSI est un modèle de communication entre les ordinateurs. Il est composé de 7 couches :


| Numéro | Nom | Description |
| --- | --- | --- |
| 7 | Application | Protocoles de haut niveau |
| 6 | Présentation | Conversion des données |
| 5 | Session | Gestion des sessions |
| 4 | Transport | Gestion des connexions |
| 3 | Réseau | Routage des paquets |
| 2 | Liaison de données | Gestion des erreurs |
| 1 | Physique | Transmission des données |

#### Exemple de protocoles par couche du modèle OSI

| Numéro | Nom | Protocoles |
| --- | --- | --- |
| 7 | Application | HTTP, FTP, SMTP, DNS, DHCP, SSH, Telnet, SNMP, POP3, IMAP, SMB, AFP, ... |
| 6 | Présentation | SSL, TLS, ... |
| 5 | Session | NetBIOS, ... |
| 4 | Transport | TCP, UDP, ... |
| 3 | Réseau | IP, ICMP, ARP, ... |
| 2 | Liaison de données | Ethernet, PPP, ... |
| 1 | Physique | RJ45, ... |
  
## Les filtres 

Les filtres permettent de filtrer les paquets capturés. Ils permettent de filtrer sur les protocoles, les adresses IP, les ports, etc.

### Syntaxe

La syntaxe des filtres est la suivante :

```bash
<protocole> <opérateur> <valeur>
```

### Opérateurs

Il existe plusieurs opérateurs :

* `==` : égal à
* `!=` : différent de
* `>` : supérieur à
* `<` : inférieur à
* `>=` : supérieur ou égal à
* `<=` : inférieur ou égal à
* `&&` : ET
* `||` : OU
* `!` : NON

#### Exemples de filtres

```bash

# Filtre sur le protocole HTTP
http

# Filtre sur le protocole HTTP et le port 80
http && port 80

# Filtre sur le protocole HTTP et le port 80 et l'adresse IP
http && port 80 && ip.addr == <ip>

# Filtre sur le protocole HTTP et le port 80 et l'adresse IP et la méthode GET

http && port 80 && ip.addr == <ip> && http.request.method == GET

# Filtre sur le protocole HTTP et le port 80 et l'adresse IP et la méthode GET et le fichier index.html

http && port 80 && ip.addr == <ip> && http.request.method == GET && http.request.uri == /index.html
```

## Protocoles 

### ARP

ARP signifie **Address Resolution Protocol**. Il permet de faire la correspondance entre une adresse IP et une adresse MAC. C'est un protocole de la **couche 3** du modèle OSI.
Ils contiennent des messages REQUEST et des messages RESPONSE. Pour identifier les paquets, l'en-tête du message contient l'un des deux codes d'opération suivants :
* 1 : ARP Request
* 2 : ARP Reply

Il est utile de noter que la plupart des appareils s'identifient eux-mêmes ou que Wireshark les identifie, comme **Intel_78**. Un exemple de trafic suspect serait de nombreuses requêtes provenant d'une source non reconnue. Vous devez cependant activer un paramètre dans Wireshark pour résoudre les adresses physiques. Pour activer cette fonction, allez dans *Affichage > Résolution des noms >* Assurez-vous que l'option *Résoudre les adresses physiques* est cochée.

#### Exemple de paquet ARP

![Paquet ARP](Images/Exemple de paquet ARP 1.png)

En regardant les détails du paquet ci-dessus, les détails les plus importants du paquet sont soulignés en rouge. 
* L'Opcode est l'abréviation de code d'opération et indique s'il s'agit d'une requête ou d'une réponse ARP.
* La cible du paquet, qui dans ce cas, est une demande de diffusion à tous.

En regardant les détails du paquet ci-dessus, nous pouvons voir d'après l'Opcode qu'il s'agit d'un paquet ARP Reply. Nous pouvons également obtenir d'autres informations utiles telles que les adresses MAC et IP qui ont été envoyées avec la réponse, puisqu'il s'agit d'un paquet de réponse, nous savons qu'il s'agit des informations envoyées avec le message.

ARP est l'un des protocoles les plus simples à analyser, tout ce dont vous devez vous souvenir est d'identifier s'il s'agit d'un paquet de demande ou de réponse et par qui il est envoyé.

Liste des filtres pour le protocole ARP :

[https://www.wireshark.org/docs/dfref/a/arp.html](https://www.wireshark.org/docs/dfref/a/arp.html)

### ICMP

ICMP signifie **Internet Control Message Protocol**. Il permet de faire des requêtes et des réponses pour tester la connectivité entre deux hôtes. C'est un protocole de la **couche 3** du modèle OSI.
Il est très connu car c'est le protocole associé à la commande `ping`.

Les paquets ICMP ont une structure similaire à celle des paquets IP. Ils contiennent des messages REQUEST et des messages RESPONSE. Pour identifier les paquets, l'en-tête du message contient l'un des deux codes d'opération suivants (**type**):

* 8 : ICMP Request
* 0 : ICMP Reply
  
Si ces codes sont modifiés ou ne semblent pas corrects, il s'agit généralement d'un signe d'activité suspecte.

### TCP

TCP signifie **Transmission Control Protocol**. Il permet de faire des connexions entre deux hôtes. C'est un protocole de la **couche 4** du modèle OSI.

Lors de l'analyse des paquets TCP, Wireshark peut être très utile et **coder les paquets par couleur en fonction du niveau de danger**. Si vous ne vous souvenez pas du code couleur, relisez la documentation de wireshark et rafraîchissez vos connaissances sur la façon dont Wireshark utilise les couleurs pour faire correspondre les paquets.

TCP peut donner un aperçu utile d'un réseau lorsqu'il est analysé, mais il peut aussi être difficile à analyser en raison du nombre de paquets qu'il envoie. C'est là que vous devrez peut-être utiliser d'autres outils tels que **RSA NetWitness** et **NetworkMiner** pour filtrer et analyser davantage les captures.

#### Analyse des paquets TCP

Pour analyser les paquets TCP, nous n'entrerons pas dans les détails de chacun d'entre eux, mais nous examinerons quelques-uns de leurs comportements et de leurs structures.

Ci-dessous, nous voyons les détails d'un paquet SYN. La principale chose que nous voulons observer dans un paquet TCP est le numéro de séquence et le numéro d'accusé de réception. Ceux-ci sont utilisés pour identifier les paquets et les connexions TCP :
* Le numéro de séquence (**sequence number**) est le numéro de séquence du paquet
* Le numéro d'accusé de réception (**Acknowledgment number**) est le numéro de séquence du paquet suivant que l'hôte attend de recevoir. Cela permet de s'assurer que les paquets sont reçus dans le bon ordre et qu'aucun paquet n'est perdu.

Dans Wireshark, nous pouvons également voir le numéro de séquence original en naviguant vers *edit > preferences > protocols > TCP > relative sequence numbers* (uncheck boxes).

En règle générale, **les paquets TCP doivent être examinés dans leur ensemble** pour raconter une histoire, plutôt que d'être examinés un par un dans les détails.

### DNS 

DNS signifie **Domain Name System**. Il permet de faire la correspondance entre un nom de domaine et une adresse IP. C'est un protocole de la **couche 7** du modèle OSI.

#### Port DNS

Le port DNS est le port 53. C'est du TCP/IP.

#### Requête DNS

Les requêtes DNS sont des requêtes envoyées par un client DNS à un serveur DNS. Elles sont envoyées sur le port 53 en UDP ou TCP. Elles sont composées d'un en-tête et d'une question. L'en-tête contient les informations suivantes :
* ID : identifiant de la requête
* QR : 0 pour une requête, 1 pour une réponse
* Opcode : 0 pour une requête standard
* AA : 1 si le serveur est autoritaire pour le domaine
* TC : 1 si le message est tronqué
* RD : 1 si le client souhaite une requête récursive
* RA : 1 si le serveur peut faire une requête récursive
* Z : 0
* RCODE : 0 si la requête est réussie


#### Points suspects dans les paquets DNS

Vous devez garder à l'esprit un certain nombre d'éléments décrits ci-dessous lorsque vous analysez des paquets DNS.

* Query-Response 
* DNS-Server Only
* UDP

Si l'un de ces éléments n'est pas à sa place, les paquets doivent faire l'objet d'un examen plus approfondi et être considérés comme suspects.

### HTTP

HTTP signifie **HyperText Transfer Protocol**. Il permet de faire des requêtes et des réponses pour récupérer des pages web. C'est un protocole de la **couche 7** du modèle OSI. HTTP est l'un des protocoles les plus directs pour l'analyse des paquets. Le protocole va droit au but et n'inclut pas de poignée de main ou de conditions préalables à la communication.
HTTP n'est pas un protocole que l'on voit beaucoup, car HTTPS est désormais plus couramment utilisé ; cependant, HTTP est encore souvent utilisé et peut être très facile à analyser si l'on en a l'occasion.

#### Port HTTP

Le port HTTP est le port 80. C'est du TCP/IP.

#### Requête HTTP

Les requêtes HTTP sont des requêtes envoyées par un client HTTP à un serveur HTTP. Elles sont envoyées sur le port 80 en TCP. Elles sont composées d'une ligne de requête, d'en-têtes et d'un corps. La ligne de requête contient les informations suivantes :

* Méthode : GET, POST, PUT, DELETE, ...
* URI : Uniform Resource Identifier
* Version : HTTP/1.1, HTTP/2, ...

#### Analyse de paquets HTTP

HTTP est utilisé pour envoyer des requêtes GET et POST à un serveur web afin de recevoir des choses telles que des pages web. Savoir analyser HTTP peut être utile pour repérer rapidement des éléments tels que SQLi, les Web Shells et d'autres vecteurs d'attaque liés au web.

Lorsque vous analysez des paquets HTTP, vous devez vous concentrer sur les éléments suivants :

Element | Description
--- | ---
Méthode | La méthode utilisée pour la requête. Les méthodes les plus courantes sont GET et POST.
URI | L'URI est l'adresse de la page web demandée.
Version | La version du protocole HTTP utilisée. Les versions les plus courantes sont HTTP/1.1 et HTTP/2.
Host | L'hôte est l'adresse IP du serveur web.
User-Agent | Le User-Agent est le navigateur utilisé pour envoyer la requête.
Accept | L'Accept est le type de données acceptées par le navigateur.
Cookie | Le Cookie est le cookie envoyé par le navigateur.

### HTTPS 

HTTPS signifie **HyperText Transfer Protocol Secure**. Il permet de faire des requêtes et des réponses pour récupérer des pages web. C'est un protocole de la **couche 7** du modèle OSI. HTTPS est une version sécurisée de HTTP. Il utilise le protocole TLS pour sécuriser les communications. Il est donc plus difficile à analyser que HTTP.Il peut être l'un des protocoles les plus difficiles à comprendre du point de vue de l'analyse des paquets et il peut être déroutant de comprendre les étapes à suivre pour analyser les paquets HTTPS.

#### Création d'un tunnel sécurisé HTTPS

Lorsqu'un client se connecte à un serveur HTTPS, il doit d'abord établir une connexion TCP/IP avec le serveur. Une fois la connexion TCP/IP établie, le client et le serveur doivent établir une connexion TLS. Pour ce faire, le client et le serveur doivent négocier les paramètres de la connexion TLS. Une fois la connexion TLS établie, le client et le serveur peuvent commencer à communiquer en utilisant le protocole HTTP.
Etapes :

1. Le client et le serveur conviennent d'une version du protocole
2. Le client et le serveur sélectionnent un algorithme cryptographique
3. Le client et le serveur peuvent s'authentifier l'un l'autre ; cette étape est facultative.
4. Création d'un tunnel sécurisé avec une clé publique

#### Analyse des paquets HTTPS 

Les paquets HTTPS s'analysent comme ceux HTTP à la différence près qu'il faut entrer la clé de chiffrement dans wireshark afin de déchiffrer les messages. Pour cela, il faut aller dans *Edit > Preferences > Protocols > TLS > Edit > + > Protocol : http > Key File : <fichier de clé> > Password : <mot de passe>*.

Exemple de configuration de la clé : 

* IP address : 127.0.0.1
* Port : start_tls
* Protocol : http
* Key File : <fichier de clé>
* Password : <mot de passe>
 
## Outils d'analyse wiresark

### Organiser la Hierarchie des protocoles 

Nous pouvons utiliser certaines des fonctions intégrées de Wireshark pour nous aider à assimiler toutes ces données et à **les organiser en vue d'une analyse ultérieure**. Nous pouvons commencer par examiner une fonction très utile de Wireshark pour organiser les protocoles présents dans une capture, à savoir la **Hiérarchie des protocoles**. Naviguez vers *Statistiques > Hiérarchie des protocoles*.

Ces informations peuvent être très utiles dans des applications pratiques telles que la chasse aux menaces pour identifier les divergences dans les captures de paquets.

### Organiser les URI 

La prochaine fonctionnalité de Wireshark que nous allons examiner est l'exportation d'objets HTTP. Cette fonction nous permettra d'**organiser tous les URI** demandés dans la capture. Pour utiliser Export HTTP Object, naviguez vers *File > Export Objects > HTTP*.

Comme pour la hiérarchie des protocoles, cela peut être utile pour identifier rapidement d'éventuelles divergences dans les captures.

### Points de terminaison ou Endpoint 

La  fonction points de terminaison (Endpoint) permet à l'utilisateur d'organiser tous les points de terminaison et toutes les adresses IP trouvés dans une capture spécifique. Tout comme les autres fonctionnalités, elle peut être utile pour identifier l'origine d'une anomalie. Pour utiliser la fonction Points d'extrémité, accédez à *Statistiques > Points d'extrémité*.

### Organiser les flux de données 

Afin d'organiser le flux de données, nous pouvons utliser la fonction d'exportation d'objets HTTP. Pour accéder à cette fonction, naviguez vers Fichier > Exporter des objets > HTTP.