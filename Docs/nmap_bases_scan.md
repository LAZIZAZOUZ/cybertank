# Les bases de Nmap : les scans

La base des bases c'est de faire un  
```
man nmap

```
Pour voir tous les switchs disponibles. 

Une autre manière d'avoir des infos est d'aller sur : [le site officiel nmap](https://nmap.org/book/)

Le comportement par défaut de Nmap est d'utiliser les hôtes en ligne par reverse-DNS. Parce que les noms d'hôtes peuvent révéler beaucoup de choses, cette étape peut être utile. Cependant, si vous ne voulez pas envoyer de telles requêtes DNS, vous pouvez utiliser l'option `-n` pour sauter cette étape.

Par défaut, Nmap recherche les hôtes en ligne ; cependant, vous pouvez utiliser l'option `-R` pour interroger le serveur DNS même pour les hôtes hors ligne. Si vous voulez utiliser un serveur DNS spécifique, vous pouvez ajouter l'option `--dns-servers DNS_SERVER`.

## Scan du réseau 

Pour commencer et avoir une vue d'ensemble du réseau le mieux est de connaitre les IPs des machines actives sur le réseau local. Pour faire ça on peut demander à nmap de faire un scan de type ping (protocole ICMP) en utilisant ```-sn```.
Il faut donner  le masque du réseau sur lequel il va faire son scan avec un /24 (ou /16 ou /8 ou autre) avec l'ip du broadcast ou alors mettre des * :

```bash 
nmap -sn -vv 192.168.0.0-254
nmap -sn -vv 192.168.0.0/24
nmap -sn -vv 192.168.0.*
```
Pro tip : pour n'afficher que les hosts qui sont up on peut associer à la commande nmap : ```grep -v "down"``` qui demande de ne pas afficher toutes les lignes où il y a marqué "down"

Rappel, le ```-vv``` c'est pour verbosity c'est pour que les résultats soient affichés avec plus de détails. ```-v``` donne un niveau de détail, ```-vv``` en donne 2.

Cependant, bien qu'il s'agisse de l'approche la plus simple, elle n'est pas toujours fiable. De nombreux pare-feu bloquent l'écho ICMP ; les nouvelles versions de MS Windows sont configurées avec un pare-feu hôte qui bloque par défaut les demandes d'écho ICMP. N'oubliez pas qu'une requête ARP précédera la requête ICMP si votre cible se trouve sur le même sous-réseau.

```bash
nmap -PE -sn MACHINE_IP/24
```
Comme les requêtes ICMP echo ont tendance à être bloquées, vous pouvez aussi considérer les requêtes ICMP Timestamp ou ICMP Address Mask pour savoir si un système est en ligne. Nmap utilise les requêtes d'horodatage (ICMP Type 13) et vérifie s'il obtiendra une réponse d'horodatage (ICMP Type 14). L'ajout de l'option `-PP` indique à Nmap d'utiliser les requêtes ICMP d'horodatage.

```bash
nmap -PP -sn MACHINE_IP/24
```

De la même manière, Nmap utilise des requêtes de masque d'adresse (ICMP Type 17) et vérifie s'il reçoit une réponse de masque d'adresse (ICMP Type 18). Ce scan peut être activé avec l'option `-PM`. 

```bash
nmap -PM -sn MACHINE_IP/24
```
Si vous voulez que Nmap utilise le **TCP SYN ping**, vous pouvez le faire via l'option `-PS` suivie du numéro de port, de la plage, de la liste, ou d'une combinaison de ceux-ci. Par exemple, `-PS21` ciblera le port 21, tandis que `-PS21-25` ciblera les ports 21, 22, 23, 24 et 25. Enfin, `-PS80,443,8080` ciblera les trois ports 80, 443 et 8080.

```bash
nmap -PS -sn MACHINE_IP/24
```

Pour faire un **TCP ACK ping** c'est le même principe mais il faut mettre `-PA`. Ça enverra un packet avec le flag 'ACK' d'activé.

```bash 
sudo nmap -PA -sn MACHINE_IP/24
```

Enfin, nous pouvons utiliser UDP pour savoir si l'hôte est en ligne `-PU`. Contrairement au TCP SYN ping, l'envoi d'un paquet UDP à un port ouvert ne devrait pas donner lieu à une réponse. En revanche, si nous envoyons un paquet UDP à un port UDP fermé, nous nous attendons à recevoir un paquet ICMP de port inaccessible, ce qui indique que le système cible est en ligne et disponible.

```bash
sudo nmap -PU -sn MACHINE_IP/24
```

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

## Scan TCP ACK `-sA`

Simplement, comme son nom l'indique ça envoie un paquet avec le flag ACK. La cible devrait répondre avec un paquet RST

## scan TCP Windows `-sW`

Une autre analyse similaire est l'analyse de la fenêtre TCP. L'analyse de la fenêtre TCP est presque identique à l'analyse ACK, mais elle examine le champ TCP Window des paquets RST renvoyés. Sur certains systèmes, cela peut révéler que le port est ouvert. Vous pouvez sélectionner ce type d'analyse avec l'option -sW. 

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

### Scan Maimon `-sM`

Dans ce scan les bits FIN et ACK sont activés. La cible devrait renvoyer un packet RST. **Ce scan ne devrait pas fonctionner sur les réseaux récents**.

### Scan personnalisés

On peut customiser les flags TCP envoyés avec `--scanflags`. Il suffit de rajouter les flags que l'on souhaite avoir activé simultanément : SYN, RST, FIN ou autre.

```bash
sudo nmap --scanflags SYNRSTFIN
```

#### Rappel sur la structure d'un flag TCP/IP :

3 octets composés de : 
* 3 bits réservés - 1 bit "Nonce"
* 1 bit CWR (Congestion Windows Reduced) - 1 bit  ECN (Echo) - 1 bit URG (Urgent) - 1 bit ACK (Acknoledgement)
* 1 bit PSH (Push) - 1 bit RST (Reset) - 1 bit SYN (Synchronisation) - 1 bit FIN 

## Amélioration des performances 

Vous pouvez spécifier les ports que vous souhaitez analyser au lieu des 1000 ports par défaut. La spécification des ports est maintenant intuitive. Voyons quelques exemples :

* liste des ports : `-p22,80,443` pour scanner les ports 22, 80 et 443.
* plage de ports : `-p1-1023` analysera tous les ports compris entre 1 et 1023 inclus, tandis que `-p20-25` analysera les ports compris entre 20 et 25 inclus.

Vous pouvez demander l'analyse de tous les ports en utilisant `-p-`, qui analysera les **65535** ports. Si vous souhaitez analyser les 100 ports les plus courants, ajoutez `-F`. L'utilisation de `--top-ports 10` vérifiera les dix ports les plus courants.

Vous pouvez contrôler la durée de l'analyse en utilisant `-T<0-5>`. `-T0` est le plus lent (paranoïaque), tandis que `-T5` est le plus rapide. Selon la page de manuel de Nmap, il existe six modèles :

* paranoïaque (0)
* sournois (1)
* poli (2)
* normal (3)
* agressif (4)
* fou (5)

Pour éviter les alertes IDS, vous pouvez opter pour `-T0` ou `-T1`. Par exemple, `-T0` scanne un port à la fois et **attend 5 minutes** entre l'envoi de chaque sonde, ce qui vous permet de deviner combien de temps le scan d'une cible prendra pour se terminer. Si vous ne spécifiez pas de timing, **Nmap utilise la méthode normale** `-T3`. Notez que `-T5` est le plus agressif en termes de vitesse ; cependant, cela peut affecter la précision des résultats de l'analyse en raison de la probabilité accrue de perte de paquets. Notez que `-T4` est souvent **utilisé pendant les CTF** et lors de l'apprentissage du balayage sur des cibles d'entraînement, alors que `-T1` est souvent **utilisé pendant les engagements réels** où la furtivité est plus importante.

Vous pouvez également choisir de contrôler le taux de paquets en utilisant `--min-rate <nombre>` et `--max-rate <nombre>`. Par exemple,` --max-rate 10` ou `--max-rate=10` garantit que votre scanner n'envoie pas plus de dix paquets par seconde.

De plus, vous pouvez contrôler la parallélisation des sondages en utilisant` --min-parallelism <numprobes>` et `--max-parallelism <numprobes>`. Nmap sonde les cibles pour découvrir quels hôtes sont actifs et quels ports sont ouverts ; le parallélisme des sondes spécifie le nombre de sondes qui peuvent être exécutées en parallèle. Par exemple, -`-min-parallelism=512` pousse Nmap à maintenir au moins 512 sondes en parallèle ; ces 512 sondes sont liées à la découverte des hôtes et des ports ouverts.