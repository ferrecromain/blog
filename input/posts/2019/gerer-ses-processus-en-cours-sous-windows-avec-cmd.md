Title: Gérer ses processus en cours sous Windows avec CMD
Published: 28/03/20
Tags: [Administration, Windows]
---

Nous allons voir quelques commandes indispensables pour gérer les différentes applications et processus en cours sous Windows avec l'invite de commande, au lieu de passer par le traditionnel *Gestionnaire des tâches*.

### Lister, filtrer et trier les tâches en cours d'exécution

La commande ```tasklist``` nous permet d'obtenir la plupart des informations sur les tâches en cours d'exécutions. 

```console
C:\>tasklist

Nom de l’image                 PID Nom de la sessio Numéro de s Utilisation
========================= ======== ================ =========== ============
System Idle Process              0 Services                   0         8 Ko
System                           4 Services                   0     6 736 Ko
Registry                       104 Services                   0    21 608 Ko
smss.exe                       388 Services                   0       352 Ko
csrss.exe                      580 Services                   0     2 304 Ko
wininit.exe                    656 Services                   0     1 004 Ko
services.exe                   728 Services                   0     6 564 Ko
lsass.exe                      820 Services                   0    14 440 Ko
svchost.exe                    932 Services                   0     1 124 Ko
fontdrvhost.exe                952 Services                   0       460 Ko
svchost.exe                    968 Services                   0    20 636 Ko
...
```

Il est possible d'y coupler la commande ```find``` afin d'y filtrer les éléments qui nous intéresses :

```console
C:\>tasklist | find "msedge"
msedge.exe                   12984 Console                    5   117 536 Ko
msedge.exe                   10576 Console                    5     2 140 Ko
msedge.exe                   11988 Console                    5     1 344 Ko
msedge.exe                   12832 Console                    5   108 476 Ko
...
```

De plus avec la commande ```sort```, nous pouvons trier la sortie, ci-dessous selon la mémoire occupée, par ordre décroissant.

Petit rappel sur les paramètres de la commande sort :
 - ```/r``` : permet d'avoir un ordre de trie descendant.
 - ```/+n``` : où n spécifie la position du caractère où effectuer le tri. Ici **64** correspond, mais cela peut différer selon votre sortie.

```console
C:\>tasklist | find "msedge" | sort /r /+64
Nom de l'image                 PID Nom de la sessio Numéro de s Utilisation
========================= ======== ================ =========== ============
msedge.exe                    1924 Console                    5   147 532 Ko
msedge.exe                   12832 Console                    5   127 128 Ko
msedge.exe                   12984 Console                    5   121 928 Ko
msedge.exe                    6692 Console                    5    81 464 Ko
msedge.exe                   11832 Console                    5    61 856 Ko
...
```

La commande ```tasklist``` comporte de nombreux *filtres* que vous pouvez utiliser, en fonction de vos besoins. Ici nous n'allons retenir que les instances de Microsoft Edge occupant plus de 100 Mo de mémoire vive.

```console
C:\>tasklist /fi "memusage gt 100000" | find "msedge" | sort /r /+58
msedge.exe                    1924 Console                    5   146 712 Ko
msedge.exe                   12832 Console                    5   131 840 Ko
msedge.exe                   12984 Console                    5   121 980 Ko
```

Notez que nous préférons utiliser la commande ```find``` plutôt que le filtre *imagename*. Ce dernier implique d'écrire le nom complet du processus, n'ayant que l'égalité comme opérateur de comparaison.

### Arrêter les tâches en cours d'exécution

Maintenant que nous connaissons les fondamentaux concernant l'affichage des informations sur les processus, voyons comment arrêter l'exécution de l'un ou plusieurs d'entre eux, avec la commande ```taskkill```.


#### Arreter un processus particulier à l'aide de son identifiant (PID).

```console
C:\>tasklist /fi "pid eq 1360"

Nom de l’image                 PID Nom de la sessio Numéro de s Utilisation
========================= ======== ================ =========== ============
msedge.exe                    1360 Console                    2   115 164 Ko

C:\>taskkill /pid 5380
Opération réussie : un signal de fin a été envoyé au processus de PID 5380.

C:\>tasklist /fi "pid eq 5380"
Information : aucune tâche en service ne correspond aux critères spécifiés.
```

Notez que le paramètre ```/pid``` peut être repété autant de fois que vous avez de processus particulier a fermer.

#### Arrêter un ensemble de processus à l'aide de leur nom commun.

```console
C:\>tasklist | find "notepad"
notepad.exe                    468 Console                    2    13 528 Ko
notepad.exe                   2640 Console                    2    13 524 Ko

C:\>taskkill /im "notepad*"
Opération réussie : un signal de fin a été envoyé au processus "notepad.exe" de PID 468.
Opération réussie : un signal de fin a été envoyé au processus "notepad.exe" de PID 2640.

C:\>tasklist | find "notepad"
notepad.exe                    468 Console                    2    13 528 Ko
notepad.exe                   2640 Console                    2    13 524 Ko
```

Les deux instances de notepad que nous avons souhaité fermer sont toujours actif, car seul un **signal de fin** a été envoyé au processus.

Si nous savons ce que nous faisons, et que nous souhaitons passer outre ce comportement, vous pouvez **forcer** l'extinction (on parle également de tuer un processus), à l'aide du paramètre ```/f```.

```console
C:\>tasklist | find "notepad"
notepad.exe                    468 Console                    2    13 528 Ko
notepad.exe                   2640 Console                    2    13 524 Ko

C:\>taskkill /im "notepad*" /f
Opération réussie : le processus "notepad.exe" de PID 468 a été arrêté.
Opération réussie : le processus "notepad.exe" de PID 2640 a été arrêté.

C:\>tasklist | find "notepad"
notepad.exe                    468 Console                    2    13 528 Ko
notepad.exe                   2640 Console                    2    13 524 Ko
```