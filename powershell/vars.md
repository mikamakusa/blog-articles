+++
date = "2016-03-15"
draft = false
title = "Powershell #1"
+++

## Les variables
Une variable est un objet permettant de stocker des données.
Une variable peut tout aussi bien contenir des données textes, chiffres, xml, tableau, etc.

Une variable commence toujours par le signe "$".


les types de données 

	Types 					Description
	[int] 					Chiffre (32 bits)
	[long] 					Chiffre (64 bits)
	[string]				Texte
	[char]					Caractère signé [8 bits]
	[byte]					Caractère non-signé (8 bits)
	[bool]					Booléen (oui/non)
	[decimal]				Valeur décimale (128 bits)
	[single]				Nombre à virgule flottante (32 bits)
	[double]				Nombre à virgule flottante (64 bits)
	[xml]					Objet XML
	[array]					Tableau
	[hashtable]				Tableeau associatif


Les opérateurs

	Types 					Action
	= 						Assigne une valeur à une variable
	+ ou += 				Addition
	- ou -= 				Soustaction
	* ou *=					Multiplication
	/ ou /= 				Division
	% ou %= 				Modulo
	++ 						Incrementation
	--						Décrémentation


Pour associer une valeur à une variable, il faut utiliser l'opérateur "=" de cette manière : 
	
	$ma_variable = "ma_valeur"

Par exemple : 
	
	$diskoperation = "Create"

Il est aussi possible d'associer un type de données à une variable : 
	
	[string]$diskoperation = "Create"
	[int]$choice = "3"
	[xml]$xmlIPpurchased = Get-Content ippurchased.xml

Pour concaténer plusieurs valeurs, la commande est la suivante : 
	
	$valeur_A = "ValeurA"
	$Valeur_B = "Valeur B"
	$Valeur_C = $Valeur_A += $Valeur_B

Pour remplacer une valeur dans une chaîne, il faut utiliser la commande "-replace" : 
	
	$valeur_A = "Il y a plusieurs valeurs"
	$valeur_B = $valeur_A -replace "[valeur à remplacer]","[nouvelle valeur]"

L'interpolation permet d'inclure la valeur d'une variable dans un texte : 
	
	$valeur_A = "valeur A"
	Write-Host "il y a plusieurs $valeur_A"
