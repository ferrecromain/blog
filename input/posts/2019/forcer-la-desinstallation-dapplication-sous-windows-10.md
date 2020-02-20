Title: Forcer la désinstallation d'application sous Windows 10
Published: 19/02/20
Tags: Windows
---

L'arrivée de Windows 10 a permis de profier des applications universelles que l'ont peut
télécharger et installer depuis une logithèque officielle, le fameux Windows Store.

Une gamme d'application nous est proposée par défaut tel que Cartes, Enregistreur Vocal ou
encore Photos.
Cela dit, quel intêret de disposer d'un logiciel d'édition vidéo si vous n'avez aucun
besoin particulier en la matière ? Comment faire pour se séparer des applications inutiles ?

Nous allons commencer par obtenir la liste des applications installées sur le système. Ouvrez
pour cela une session Powershell, et saisissez la commande suivante :

<?# highlight powershell ?>
Get-AppxPackage | select Name
<?#/ highlight ?>