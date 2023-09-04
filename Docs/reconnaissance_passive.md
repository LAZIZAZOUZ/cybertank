# Outils de reconnaissance passive

La reconnaissance passive consiste à rassembler des informations sur la cible sans directement se connecter avec elle. Cela se fait principalement en se connectant à des bases de données publiques. 

## Whois

WHOIS est un protocole de demande et de réponse qui suit la spécification RFC 3912. Un serveur WHOIS écoute les demandes entrantes sur le port TCP 43. L'agent d'enregistrement du domaine est responsable de la mise à jour des enregistrements WHOIS pour les noms de domaine qu'il loue. Le serveur WHOIS répond en fournissant diverses informations relatives au domaine demandé. En particulier, nous pouvons apprendre ce qui suit :

* Registrar : Par l'intermédiaire de quel bureau d'enregistrement le nom de domaine a-t-il été enregistré ?
* Contact info of registrant : Nom, organisation, adresse, téléphone, entre autres. (sauf si elles sont cachées par un service de protection de la vie privée)
* Creation, update and expiration : Quand le nom de domaine a-t-il été enregistré pour la première fois ? Quand a-t-il été mis à jour pour la dernière fois ? Et quand doit-il être renouvelé ?
* Name Server : Quel est le serveur auquel il faut s'adresser pour résoudre le nom de domaine ?

```bash
whois google.com
```

Grâce à `whois` on s'intéresse le plus souvent au serveur DNS utilisé. 


## nslookup et dig

`nslookup`  signifie Name Server Look Up, ça permet de retrouver l'adresse IP d'un nom de domaine.

La synthaxe est :

```bash
nslookup [option] [nom de domaine] [serveur DNS à interroger]
```
Les serveurs DNS les plus connus sont : 
*  Cloudflare avec `1.1.1.1` et `1.0.0.1`
*  Google avec `8.8.8.8` et `8.8.4.4`
*  Quad9 avec `9.9.9.9` et `149.112.112.112`

Dans la partie `[option]` on peut choisir le type de résultat que l'on veut :

option | effet
------- | ------
-type=A | adresse IPv4
-type=AAAA| adresse IPv6
-type=CNAME | Nom Canonique
-type=MX | serveur mail
-type=SOA | "Start of Authority"  
-type=TXT | TXT record

## dig

Pour des requêtes DNS plus avancées et des fonctionnalités supplémentaires, on peut utiliser `dig` (Domain Information Groper)

Comme pour `nslookup` on peut préciser un type mais pas besoin de mettre "-type=", par exemple :

```bash
dig test.com MX
```

Pour préciser le serveur DNS, on ajoute '@' :

```bash
dig @8.8.8.8 test.com MX
```

## DNSDumpster

[DNSDumpster](https://dnsdumpster.com/) est un site qui permet de lister un certains nombre de sous-domaines. Et de faire un rendu visuel du mappage du site.

Les outils de recherche DNS, tels que nslookup et dig, ne peuvent pas trouver les sous-domaines seuls. Le domaine que vous inspectez peut inclure un sous-domaine différent qui peut révéler beaucoup d'informations sur la cible. Par exemple, si tryhackme.com possède les sous-domaines wiki.tryhackme.com et webmail.tryhackme.com, vous voudrez en savoir plus sur ces deux sous-domaines, car ils peuvent contenir une mine d'informations sur votre cible. Il est possible que l'un de ces sous-domaines ait été créé et ne soit pas mis à jour régulièrement. L'absence de mises à jour régulières entraîne généralement la vulnérabilité des services. 

## Shodan

[Shodan](https://www.shodan.io/) est un site internet qui peut être utile pour obtenir diverses informations sur le réseau du client, sans s'y connecter activement. En outre, du côté défensif, vous pouvez utiliser différents services de Shodan.io pour connaître les appareils connectés et exposés appartenant à votre organisation.

Shodan.io essaie de se connecter à chaque appareil accessible en ligne pour construire un moteur de recherche de "choses" connectées, contrairement à un moteur de recherche de pages web. Dès qu'il obtient une réponse, il collecte toutes les informations relatives au service et les enregistre dans la base de données pour les rendre consultables.