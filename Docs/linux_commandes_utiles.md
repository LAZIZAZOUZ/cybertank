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

