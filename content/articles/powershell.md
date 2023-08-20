# Tutoriel avancé sur l'utilisation de la fonctionnalité "Runspace" de PowerShell

Le **Runspace** est une fonctionnalité avancée de PowerShell qui vous permet d'exécuter des commandes en parallèle, ce qui peut considérablement améliorer les performances de vos scripts et de vos tâches. Cette fonctionnalité est particulièrement utile lorsque vous avez besoin de traiter de grandes quantités de données ou d'effectuer des opérations intensives en calcul.

## Qu'est-ce qu'un Runspace ?

Un **Runspace** est un environnement d'exécution isolé dans PowerShell. Il permet d'exécuter des commandes en parallèle tout en maintenant la séparation entre elles. Cela signifie que chaque runspace dispose de son propre espace mémoire et d'une exécution indépendante, ce qui peut conduire à des gains significatifs de performance.

## Cas d'usage : Traitement parallèle de fichiers

Prenons l'exemple concret de devoir traiter un grand nombre de fichiers, comme la conversion d'images ou l'analyse de logs. Sans l'utilisation de Runspaces, cela pourrait prendre beaucoup de temps. Avec Runspaces, vous pouvez diviser le traitement en parallèle et profiter de l'utilisation efficace des ressources de votre système.

### Étape 1 : Importation du module Runspace

Avant d'utiliser des Runspaces, assurez-vous d'importer le module approprié en exécutant la commande suivante :

```powershell
Import-Module PSSession


# Importation du module
Import-Module PSSession

# Chemin vers les fichiers à traiter
$cheminFichiers = "C:\Chemin\Vers\Fichiers\"

# Création d'une liste pour stocker les résultats
$resultats = @()

# Création des Runspaces
$runspaces = @()
$nombreRunspaces = 5 # Nombre de Runspaces à utiliser

# Fonction pour le traitement individuel de chaque fichier
function TraitementFichier {
    param($chemin)

    # Opérations à effectuer sur le fichier (exemple : conversion d'image)
    $resultat = "Traitement effectué pour $chemin"
    return $resultat
}

# Création des Runspaces avec les opérations à effectuer
for ($i = 0; $i -lt $nombreRunspaces; $i++) {
    $runspace = [powershell]::Create().AddScript({
        param($chemin)
        TraitementFichier -chemin $chemin
    })
    $runspace.RunspacePool = $pool
    $runspaces += [PSCustomObject]@{ Pipe = $runspace; Status = $runspace.BeginInvoke($cheminFichiers[$i]) }
}

# Attente de la fin de tous les Runspaces et récupération des résultats
$runspaces | ForEach-Object {
    $_.Pipe.EndInvoke($_.Status)
    $_.Pipe.Dispose()
}

# Fermeture du pool de Runspaces
$pool.Close()
$pool.Dispose()

# Affichage des résultats
$resultats
