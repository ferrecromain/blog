Title: Forcer la désinstallation d'applications sous Windows 10
Published: 19/02/20
Tags: Windows
---

Depuis l'arrivée de Windows 10, nous pouvons profiter des **applications universelles**,
téléchargeables depuis le [Microsoft Store](https://www.microsoft.com/fr-fr/store/apps/windows).

Une gamme d'application nous est proposée par défaut tel que *Cartes*, *Enregistreur Vocal* ou
encore *Photos*.
Cela dit, quel intérêt de disposer d'un logiciel d'édition vidéo si nous n'avons aucun
besoin particulier en la matière ? Comment faire pour se séparer des applications inutiles ?

Tout d'abord, nous allons commencer par récupérer la liste des applications installées sur le 
système. Ouvrons une session **Powershell**, et saisissons la commande suivante :

```powershell
Get-AppxPackage | select Name
```

Dans la liste des résultats, nous cherchons le **nom de code** de l'application à désinstaller
(dans notre exemple ```Microsoft.ZuneVideo``` qui correspond à *Films et TV*).

Il nous suffit ensuite d'exécuter la commande :

```powershell
Get-AppxPackage Microsoft.ZuneVIdeo | Remove-AppxPackage
```

Une barre de progression apparait en haut du terminal, nous indiquant la progression de la désinstallation.

Si nous souhaitons la réinstaller ultérieurement, cela est possible en recherchant le nom de l'application depuis **Microsoft Store**.