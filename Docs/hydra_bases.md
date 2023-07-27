# Les bases de l'outil Hydra

## Introduction

Hydra est un outil puissant de bruteforce de mots de passe en ligne. Il peut être utilisé pour attaquer de nombreux protocoles différents !

## Utilisation

### Syntaxe

```bash
hydra [[[-l LOGIN|-L FILE] [-p PASS|-P FILE]] | [-C FILE]] [-e nsr] [-o FILE] [-t TASKS] [-M FILE [-T TASKS]] [-w TIME] [-W TIME] [-f] [-s PORT] [-x MIN:MAX:CHARSET] [-SuvVd46] [service://server[:PORT][/OPT]]
```

### Options

TAG | Description
--- | ---
`-l LOGIN` ou `-L FILE` | identifiant ou fichier contenant une liste d'identifiants
`-p PASS` ou `-P FILE` | mot de passe ou fichier contenant une liste de mots de passe
`-C FILE` | fichier contenant des identifiants et des mots de passe séparés par des espaces
`-e nsr` | arrêter sur le premier succès, afficher les réponses négatives, afficher les réponses positives
`-o FILE` | enregistrer les résultats dans le fichier FILE
`-t TASKS` | exécuter TASKS nombre de tâches simultanément (valeur par défaut : 16)
`-M FILE` | serveurs multiples (fichier de liste)
`-T TASKS` | exécuter TASKS nombre de tâches simultanément par serveur (valeur par défaut : 16)
`-w TIME` | délai d'attente pour les réponses (valeur par défaut : 30)
`-W TIME` | délai d'attente global pour les réponses (valeur par défaut : 30)
`-f` | quitter après la première réponse positive
`-s PORT` | si le service est sur un autre port que le port par défaut, spécifiez-le ici
`-x MIN:MAX:CHARSET` | exécuter un bruteforce, avec des longueurs de mot de passe de MIN à MAX avec le jeu de caractères CHARSET
`-S` | connexion SSL
`-u` | connexion TLS
`-vV` | niveau de verbosité, -v : afficher les tentatives, -V : afficher les tentatives et les réponses
`-d` | afficher les informations de débogage
`-4` | force IPv4
`-6` | force IPv6

### Exemples

#### SSH

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt -t 4 -vV
```

