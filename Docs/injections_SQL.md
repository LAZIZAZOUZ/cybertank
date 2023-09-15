# Les injections SQL

## Introduction

Il y a beaucoup à dire sur les injections SQL, je ne reviens pas sur le principe de base mais chaque injection est un peu unique. Il y a beaucoup de façons d'exploiter une injection SQL, et il y a beaucoup de façons de la trouver. Je vais vous montrer quelques techniques pour trouver des injections SQL et comment les exploiter en fonction de ce que j'ai rencontré.

## Injection SQL dans un formulaire de connexion

### Contexte de l'injection

Imaginons que l'on remarque qu'un code php exécute une requête SQL du type :

```php
<?php
$requete = "SELECT * FROM users WHERE username = '" . $_POST['username'] . "' AND password = '" . $_POST['password'] . "'";
?>
```

On peut voir que le code php récupère les variables POST username et password et les concatène dans une requête SQL. On peut donc essayer d'injecter du code SQL dans ces variables.

Même sans avoir le code PHP, on peut essayer d'injecter du code SQL dans les champs de connexion. Si on obtient une erreur SQL, c'est qu'il y a une injection SQL possible. 

### Injection SQL

Maintenant que nous savons comment l'application fonctionne, nous allons tirer parti d'une fonctionnalité de SQL appelée "commentaires" pour abuser de la requête SQL et exécuter n'importe quelle commande arbitraire. SQL vous permet d'ajouter des commentaires à la fin de vos requêtes. Cette fonctionnalité est utile pour de nombreuses raisons, mais dans notre cas, elle sera utilisée pour injecter une commande dans la requête SQL afin de la faire exécuter.

Pour ajouter un commentaire à la fin d'une requête SQL, il suffit de l'enfermer dans le format suivant `-- commentaire`. Si la requête SQL détecte un commentaire, elle **l'ignorera** et exécutera la requête normalement.

Voici la ligne que l'on peut rentrer dans le formulaire de connexion pour essayer de bypasser l'authentification :

```
' OR 1=1 -- 
```

On peut voir que la requête SQL devient :

```sql
SELECT * FROM users WHERE username = '' OR 1=1 -- ' AND password = ''
```

On peut voir que la requête SQL est maintenant valide et que l'on peut se connecter sans connaître le mot de passe.

## Injection SQL via requête GET (HTTP)

### Contexte de l'injection

Imaginons que l'on soit sur un site internet qui donne des informations sur une personne ou un produit ou autre et que l'on remarque que l'URL est du type :

```
https://www.example.com/index.php?id=1

ou 

https://www.example.com/about/1

```
On peut supposer que le site utilise une requête SQL du type :

```sql
SELECT infos FROM users WHERE id = $_GET['id']
```

En utilisant **Burpsuite** par exemple, on peut intercepter la requête HTTP et la modifier pour essayer d'injecter du code SQL. (Dans ce cas particulier on peut le faire directement dans l'URL)

### Requête Union

Imaginons que la requête HTTP d'erreur du site renvoie la requête SQL (c'est improbable et ce serait une grosse faille mais c'est pour comprendre le principe) :

```sql
SELECT firstName, lastName, pfpLink, role, bio FROM people WHERE id = 2
```

Ainsi, on voit que la requête sélectionne cinq colonnes du tableau : firstName, lastName, pfpLink, role et bio. Nous pouvons deviner l'emplacement de ces colonnes dans la page, ce qui nous aidera à choisir l'endroit où placer nos demandes.

Comme nous connaissons le nom de la table et le nombre de lignes, nous pouvons utiliser une requête d'union pour sélectionner les noms des colonnes de la table ```people``` dans la table columns de la base de données par défaut ```information_schema```.

Si on injecte : 
    
```sql
0 UNION ALL SELECT column_name,null,null,null,null FROM information_schema.columns WHERE table_name="people"
```
Cela crée une requête d'union et sélectionne notre cible, puis quatre colonnes nulles (pour éviter les erreurs de la requête). Remarquez que nous avons également changé l'ID que nous sélectionnons de 2 à 0. En définissant l'ID à un nombre invalide, nous nous assurons que nous ne récupérons rien avec la requête originale (légitime) ; cela signifie que la première ligne renvoyée par la base de données sera notre réponse souhaitée à partir de la requête injectée.

#### Récupérer plusieurs colonnes 

Supposons que la réponse à la requête précédente ai été :
    
```sql  
id
```
Cela signifie que seule la première colonne a été renvoyée. Il faut donc modifier la requête pour récupérer les autres colonnes dans une seule réponse. 

Heureusement, nous pouvons utiliser notre SQLi pour regrouper les résultats. Nous ne pouvons toujours récupérer qu'un seul résultat à la fois, mais en utilisant la fonction `group_concat()`, nous pouvons regrouper tous les noms de colonnes en un seul résultat :
    
```sql
0 UNION ALL SELECT group_concat(column_name),null,null,null,null FROM information_schema.columns WHERE table_name="people"
```

La réponse à cette requête est :
    
```sql
id, firstName, lastName, pfpLink, role, shortRole, bio, notes
```
On a ainsi récupéré le nom de toutes les colonnes présentes dans la table ```people```.

