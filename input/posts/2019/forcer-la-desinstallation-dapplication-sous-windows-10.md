Title: Forcer la désinstallation d'applications sous Windows 10
Published: 19/02/20
Tags: Windows
---

De puis l'arrivée de Windows 10, nous pouvons profiter des **applications universelles**,
telechargeables depuis une logithèque unique, le fameux 
[Microsoft Store](https://www.microsoft.com/fr-fr/store/apps/windows).

Une gamme d'application nous est proposée par défaut tel que *Cartes*, *Enregistreur Vocal* ou
encore *Photos*.
Cela dit, quel intêret de disposer d'un logiciel d'édition vidéo si vous n'avez aucun
besoin particulier en la matière ? Comment faire pour se séparer des applications inutiles ?

Tout d'abord, nous allons commencer par obtenir la liste des applications installées sur le 
système. Ouvrez pour cela une session **Powershell**, et saisissez la commande suivante :

<?# highlight powershell ?>
Get-AppxPackage | select Name
<?#/ highlight ?>

Dans la liste des resultats, cherchez le nom de l'application que vous souhaitez voir disparaitre
(dans notre exemple *Microsoft.ZuneVideo* qui correspond à *Films et TV*).
Muni de celui ci, la commande suivante permet de la désinstaller :

<?# highlight powershell ?>
Get-AppxPackage Microsoft.ZuneVIdeo | Remove-AppxPackage
<?#/ highlight ?>

Patientez jusqu'a ce que Powershell vous rende la main afin de saisir une nouvelle commande.

A présent vous pouvez constater que votre application a bel et bien été désinstallée, et pas
de panique, si vous souhaitez la réinstaller vous pouvez toujours le faire au moyen du Windows Store !