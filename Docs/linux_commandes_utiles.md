# Les commandes utiles de Linux

## Recherche de fichiers

### Rechercher un fichier

```bash
find / -name <filename>
```

### Rechercher un fichier avec un nom partiel

```bash
find / -name "*<filename>*"
```

## Recherche de logs

### Rechercher un mot dans les logs

```bash
grep -R "<word>" /var/log
```

## Recherche de user

On peut rechercher un user dans le fichier `/etc/passwd` :

```bash
cat /etc/passwd | grep <username>
```

ou alors on peut regarder dans le dossier `/home` :

```bash
ls /home
```


### Trouver les mots de passe des utilisateurs

```bash
cat /etc/shadow
```

### Trouver l'utilisateur actuel

```bash
whoami
```

### Trouver le nom de l'utilisateur du shell

On peut faire un `echo` de la variable `$SHELL` :

```bash
echo $SHELL
```

Si ça ne marche pas, on peut aller regarder dans le fichier `/etc/passwd` :

```bash
cat /etc/passwd | grep nom_utilistaur
``` 

Le retour ressemblera à ça (exemple avec l'utilisateur `www-data`):

```bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

Le nom de l'utilisateur est le premier champ, ici `www-data`. Ensuite il y a le mot de passe, mais il est chiffré donc on ne peut pas le lire. Ensuite il y a l'ID de l'utilisateur, ici `33`. Ensuite il y a l'ID du groupe de l'utilisateur, ici `33`. Ensuite il y a le nom complet de l'utilisateur, ici `www-data`. Ensuite il y a le chemin vers le dossier de l'utilisateur, ici `/var/www`. Enfin il y a le chemin vers le shell de l'utilisateur, ici `/usr/sbin/nologin`.

### Trouver la version du système d'exploitation

```bash
cat /etc/os-release
```
ou 
```bash
lsb_release -a
```
### Trouver la version du noyau

```bash
uname -a
```

## Trouver les informations concernant SSH

### Trouver la version de SSH

```bash
ssh -V
```

### Trouver la configuration de SSH

```bash
cat /etc/ssh/sshd_config
```

### Trouver les clés SSH

Affiche la clé privée de l'utilisateur username

```bash
cat /home/<username>/.ssh/id_rsa
```

Affiche la clé publique de l'utilisateur username

```bash
cat /home/<username>/.ssh/id_rsa.pub
```

### Trouver les clés autorisées

```bash
cat /home/<username>/.ssh/authorized_keys
```

### Trouver les clés connues

```bash
cat /home/<username>/.ssh/known_hosts
```

## Trouver les informations concernant Bash

### Trouver la version de Bash

```bash
bash --version
```

ou 

```bash
echo $BASH_VERSION
```


### Trouver les alias

```bash
alias
```

### Trouver les variables d'environnement

```bash
env
```

### Trouver les variables d'environnement globales

```bash 
printenv
```
### Trouver les variables d'environnement locales

```bash
set
```

### Trouver l'historique des commandes

```bash
history
```

ou afficher l'historique des commandes de l'utilisateur username

```bash
cat /home/<username>/.bash_history
```

### Trouver le fichier de configuration bash

```bash
cat /home/<username>/.bashrc
```

### Trouver le profile de l'utilisateur

```bash
cat /home/<username>/.profile
```

### Trouver le fichier de déconnexion bash

```bash
cat /home/<username>/.bash_logout
```

## Autres historiques

* `cat /home/username/.zsh_history` : affiche l'historique des commandes zsh de l'utilisateur username
* `cat /home/username/.history` : affiche l'historique des commandes history de l'utilisateur username
* `cat /home/username/.mysql_history` : affiche l'historique des commandes mysql de l'utilisateur username
* `cat /home/username/.psql_history` : affiche l'historique des commandes psql de l'utilisateur username
* `cat /home/username/.bash_sessions/*` : affiche l'historique des commandes bash de l'utilisateur username
* `cat /home/username/.bash_eternal_history` : affiche l'historique des commandes bash de l'utilisateur username
* `cat /home/username/.viminfo` : affiche l'historique des commandes vim de l'utilisateur username
* `cat /home/username/.mysql_history` : affiche l'historique des commandes mysql de l'utilisateur username
* `cat /home/username/.nano_history` : affiche l'historique des commandes nano de l'utilisateur username
* `cat /home/username/.lesshst` : affiche l'historique des commandes less de l'utilisateur username
* `cat /home/username/.wget-hsts` : affiche l'historique des commandes wget de l'utilisateur username
  