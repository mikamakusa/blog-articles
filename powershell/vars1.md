+++
date = "2016-03-15"
draft = false
title = "Powershell #2"
+++

## Les commandes

Les commandes sont composées de deux éléments : le verbe et le nom
Exemple : Get-Service 
Pour obtenir la liste des commandes disponible pour Powershell, tapez "Get-Command" et la liste de toutes les commandes, incluant les alias s'affichera.
Chaque commandes dispose de paramètres obligatoires ou non, ceux permettant d'expliciter le resultat recherché à l'aide de la commande, par exemple : Get-Command -name "*ssh*"
Les commandes suivantes seront très utile pour obtenir des informations sur les commandes recherchées : 
	
	Cmdlet				Description														Alias
	Get-Command			Informations de base sur les commandes							gcm
	Get-Help			Aide de base (utiliser -full ou -example)						help, man
	Get-Member			Informations sur les méthodes et propriétés des objets			gm
	Get-PSDrive			Informations sur les “lecteurs” PowerShell						gdr
	Get-Module			Liste les “modules” actuellement chargés						gmo
	Get-PSSnapin		Liste les “snapins” actuellement chargés						gsnp

Certaines commandes seront extremement utiles :
	
	Write-Host			Affiche en sortie standard le contenu de la commande (Write-Host "Information")
	Read-Host			Lit l'entrée de l'utilise et la stocke dans une variable
	Get-Content			Récupère le contenu d'un fichier (xml par exemple)
	Import-Csv			Récupère le contenu d'un fichier CSV, dans le but de le parser
	ConvertFrom-Json	Récupère le contenu d'un fichier JSON afin de le rendre exploitable par Powershell
	Invoke-WebRequest	Effectue une requête vers un serveur WEB ou vers une API REST
