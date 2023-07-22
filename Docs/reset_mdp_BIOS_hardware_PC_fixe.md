# Supprimer le mdp maitre BIOS d'un PC fixe avec accès au hardware 

### Etape 1 :

Débrancher la carte mère et essayer d'allumer l'ordinateur à vide afin de vider les condensateurs (on est jamais trop prudent !)

### Etape 2 : 

1. Retirer la pile de la carte mère attendre 2 min. 
2. Démarrer à vide. 
3. Remettre le pile. 
4. Rallumer l'ordinateur et regarder si le mdp a disparu.

### Etape 3 : si ça n'a pas fonctionné

Regarder si la carte mère a un cavalier au niveau du module BIOS, c'est ce qu'on appelle un NUC. Il devrait y avoir 3 pin avec marqué "BIOS sec" et le chevalet est sur 2 d'entre eux (normalement sur le pin 1 et 2)

![NUC_bios_sec]( \Images\NUC_bios_sec.png "NUC.png")

### Etape 4 : Si c'est un NUC de nouvelle génération

1. Retirer le cavalier (avant prendre une photo est conseillé pour retenir sa position)
2. Rebrancher et rallumer
3. L'ordinateur va entrer dans un menu de configuration
4. Sélectionner "Supprimer le mot de passe de l'utilisateur et du superviseur du BIOS
5. Eteindre et tout débrancher 
6. Replacer le NUC sur les broches 1 et 2 (dans le bon sens Attention)
7. Rebrancher et voilà

### Etape 5 : Si c'est un NUC de première génération

Liste des modèles de 1<sup>ère</sup> génération :

* D33217CK
* D33217GKE
* DCP847SKE
* D53427RKE
* DC3217BY
* DC3217IYE
* DCCP847DYE
* DC53427HYE  

Etapes à suivre :  
1. Mettre le NUC sur les pins 2-3 (au lieu de 1-2)  
2. Reprendre à l'étape 2 de la partie précédente

