# Les bases de Nmap : les scans

La base des bases c'est de faire un  
```
man nmap

```
Pour voir tous les switchs disponibles. 

Une autre manière d'avoir des infos est d'aller sur : [le site officiel nmap](https://nmap.org/book/)

## Scan du réseau 

Pour commencer et avoir une vue d'ensemble du réseau le mieux est de connaitre les IPs des machines actives sur le réseau local. Pour faire ça on peut demander à nmap de faire un scan de type ping (protocole ICMP) en utilisant ```-sn```.
Il faut donner  le masque du réseau sur lequel il va faire son scan avec un /24 (ou /16 ou /8 ou autre) avec l'ip du broadcast ou alors mettre des * :
```
nmap -sn -vv 192.168.0.0-254
nmap -sn -vv 192.168.0.0/24
nmap -sn -vv 192.168.0.*
```
Pro tip : pour n'afficher que les hosts qui sont up on peut associer à la commande nmap : ```grep -v "down"``` qui demande de ne pas afficher toutes les lignes où il y a marqué "down"

Rappel, le ```-vv``` c'est pour verbosity c'est pour que les résultats soient affichés avec plus de détails. ```-v``` donne un niveau de détail, ```-vv``` en donne 2.

## Les scans basiques et moins basiques

Les scans basiques :
* Scan de connexion TCP : ```-sT ```
* Scan de connexion SYN ( TCP discret ou demi-ouvert) : ```-sS```
* Scan de connexion UDP : ```-sU```

Les moins basiques :
* Scan de connexion NULL (TCP sans flag) : ```-sN```
* Scan de conexion FIN (TCP avec flag de fermeture) : ```-sF```
* Scan de connexion Xmas (TCP avec 3 flags) : ```-sX```

## TCP basique (scan de base de Nmap sans sudo)

En utilisant le switch ```-sT```, Nmap va essayer une connexion TCP/IP ( 3 way handshake), il envoie un paquet de synchronisation SYN puis s'attend à recevoir un packet de synchronisation et de prise en compte : SYN/ACK et finit par répondre un packet de prise en compte : ACK (acknoledgement).

Si la connexion est fermée, pour toute requête SYN sera envoyé un paquet de rejet appelé reset : RST

Petit rappel, après une connexion complète TCP/IP, on a un flux d'ouvert qu'il faut penser à fermer.

**Si le port est derrière un firewall, le peu importe qu'il soit ouvert ou non, si notre flux n'est pas explicitement ouvert notre SYN sera drop par le firewall et on resevra un RST donc ce n'est pas forcément le meilleur moyen de connaitre les ports ouverts**

## Scan SYN (scan de base de Nmap en sudo)

C'est un scan plus discret car dit à moitié ouvert. Il se lance avec ```-sS```. Nmap va envoyer un requête SYN puis s'il reçoit une réponse SYN/ACK, au lieu d'ouvrir le flux en envoyant un paquet ACK, il envoie un packet RST. Il ferme donc directement le flux mais prend connaissance que le port est ouvert.

Avantages : 
* Il peut être utilisé pour contourner les anciens systèmes de détection d'intrusion, car ils recherchent une poignée de main complète à trois voies. Ce n'est souvent plus le cas avec les solutions IDS modernes ; c'est pour cette raison que les analyses SYN sont encore souvent appelées analyses "furtives".
* Les analyses SYN ne sont souvent pas enregistrées par les applications qui écoutent sur des ports ouverts, car la pratique courante consiste à enregistrer une connexion une fois qu'elle a été entièrement établie. Une fois de plus, cela contribue à l'idée que les analyses SYN sont furtives.
* Sans avoir à se préoccuper de l'établissement (et de la déconnexion) d'une poignée de main à trois voies pour chaque port, les balayages SYN sont nettement plus rapides qu'un balayage TCP Connect standard.

Inconvénients : 
* Ils nécessitent les permissions sudo pour fonctionner correctement sous Linux. En effet, les analyses SYN nécessitent la capacité de créer des paquets bruts (par opposition à la poignée de main TCP complète), ce qui est un privilège que seul l'utilisateur root possède par défaut.
* Les services instables sont parfois interrompus par les analyses SYN, ce qui peut s'avérer problématique si un client a fourni un environnement de production pour le test.

En définitive, les avantages l'emportent sur les inconvénients.

Pour cette raison, les scans SYN sont les scans par défaut utilisés par Nmap s'il est exécuté avec les permissions sudo. S'il est exécuté sans les permissions sudo, Nmap utilise par défaut le scan TCP Connect que nous avons vu dans la tâche précédente

## Scan UDP

Les scans UDP sont très lent car le protocole UDP est fait pour la rapidité et n'a pas "d'accusé de réception", on peut connaitre les ports fermés mais on ne peut pas différencier les ports ouverts de ceux bloqués par firewall (et qui drop les paquets). On l'active avec ```-sU```.

Lorsqu'un paquet est envoyé à un port UDP ouvert, il ne devrait pas y avoir de réponse. Lorsque cela se produit, Nmap se réfère au port comme étant ouvert|filtré. En d'autres termes, il suspecte que le port est ouvert, mais qu'il pourrait être protégé par un pare-feu. S'il obtient une réponse UDP (ce qui est très inhabituel), le port est marqué comme ouvert. Le plus souvent, il n'y a pas de réponse, auquel cas la demande est envoyée une deuxième fois pour une double vérification. S'il n'y a toujours pas de réponse, le port est marqué comme ouvert|filtré et Nmap continue.

Lorsqu'un paquet est envoyé à un port UDP fermé, la cible doit répondre par un paquet ICMP (ping) contenant un message indiquant que le port est inaccessible. Cela identifie clairement les ports fermés, que Nmap marque comme tels et passe à autre chose.

*Lorsqu'il scanne des ports UDP, Nmap envoie généralement des requêtes complètement vides - juste des paquets UDP bruts. Cela dit, pour les ports qui sont généralement occupés par des services bien connus, il enverra plutôt une charge utile spécifique au protocole qui est plus susceptible de susciter une réponse à partir de laquelle un résultat plus précis peut être tiré.
*
### Améliorer la rapidité du scan

Comme le scan UDP est uuuuultra lent on peut le lancer que sur les ports les plus connus/fréquement utilisés. Par exemple si on veut faire le scan que sur les 50 ports les plus connus on peut faire : 
```--top-ports 50 ```
On peut faire ça avec n'importe quel type de scan mais celui avec lequel c'est le plus utilisé est avec le scan udp :
```nmap -sU --top-ports 50 192.10.0.5```

## Contourner les firewall

Ces 3 scans sont encore un peu plus furtifs que le scan SYN. L'objectif ici est, bien sûr, d'échapper aux pare-feux. De nombreux pare-feux sont configurés pour rejeter les paquets TCP entrants vers les ports bloqués qui ont le drapeau SYN activé (bloquant ainsi les nouvelles demandes d'établissement de connexion). En envoyant des demandes qui ne contiennent pas le drapeau SYN, nous contournons efficacement ce type de pare-feu. Bien que ce soit une bonne chose en théorie, la plupart des solutions IDS modernes sont sensibles à ces types d'analyse, il ne faut donc pas compter sur leur efficacité à 100 % lorsqu'il s'agit de systèmes modernes.
Attention, en pratique Microsoft Windows (et un grand nombre de dispositifs de réseau Cisco) sont connus pour répondre par un RST à tout paquet TCP malformé, que le port soit ouvert ou non. Il en résulte que tous les ports apparaissent comme étant fermés.

### Scan NULL ```-sN```

Simplement, le scan envoie une requête TCP remplie de 0 et attend en retour une réponse de RST.

### Scan FIN ```-sF```

Le scan envoie une requête TCP avec uniquement le bit de "FIN" d'activé et s'attend à une réponse RST.

### Scan Xmas ```-sX```

Le scan envoie un reqête TCP avec les bits Urgent(URG), Push (PSH) et Fin (FIN) d'activés

#### Rappel sur la structure d'un flag TCP/IP :

3 octets composés de : 
* 3 bits réservés - 1 bit "Nonce"
* 1 bit CWR (Congestion Windows Reduced) - 1 bit  ECN (Echo) - 1 bit URG (Urgent) - 1 bit ACK (Acknoledgement)
* 1 bit PSH (Push) - 1 bit RST (Reset) - 1 bit SYN (Synchronisation) - 1 bit FIN 
