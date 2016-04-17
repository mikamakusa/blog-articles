+++
date = "2016-03-15"
draft = false
title = "Powershell #4"
+++

## Les boucles
Ces blocs de codes sont destinés à réaliser plusieurs itérations d’un même ensemble d’instructions.  
Ils ont toutefois besoin d’une directive stipulant les conditions de sortie, sous peine d’écrire une boucle sans fin.

La boucle **for** est une instruction relativement basique et rarement utilisée en raison de son caractère arbitraire. Sa structure est la suivante :

	for ( valeur de départ ; test ou condition ; incrément / nombre d’itération }

par exemple, le code suivant affiche les chiffres de 1 à 5 :

	for ($i=1; $i -le 5; $i++)
	{Write-Host $i}	

Obtenez plus d’informations via :

	help about_for

La boucle **foreach** est une instruction très fréquemment utilisée en langage Powershell.  
Contrairement à la directive **for**, cette instruction permet de parcourir un tableau ou une collection composée de 0 à n éléments, sans se préoccuper du nombre. Très proche, pour ne pas dire “similaire”, à l’applet de commande “foreach-object”, alias “%”, sa structure syntaxique est la suivante :

	$Collection = 1..10
	foreach ($item in $Collection) {
	write-host $item
	}

Obtenez plus d’informations via :

	help about_foreach

La boucle **do** nécessite une condition de sortie définie dans la directive “while” ou “Until” (c’est à dire, “tant que la condition n’est pas” ou “jusqu’à ce que la condition soit” remplie ou vérifiée) – Il est également possible d’utiliser la directive “while” sans stipuler le “do”.

Structure d’une boucle “do” :

	do {
	  $valeur++
	  Write-Host $valeur
	} 
	while ($valeur –ge | –le 10)

ou

	do {
	  $valeur++
	  Write-Host $valeur
	} 
	until ($valeur –ge | –le 10)


Il existe de nombreuses variantes d’écriture, selon que la condition soit traitée par “until” ou “while”, selon l’opération de comparaison et l’évaluation éventuellement inversée via un “not”… Bref, ce genre de code reste donc à la discrétion et préférence de l’auteur, l’essentiel, c’est que ça marche !…

A la différence de la directive suivante, les boucles **do** sont toujours exécuté au moins une fois. C’est à dire que la condition est évaluée après l’exécution du bloc de script. Dans le cas d’une boucle **while** uniquement, la condition est évaluée avant l’exécution du bloc de script.

Structure d’une boucle “while” seulement :
	
	while($valeur -ne 10) {
	  $valeur++
	  Write-Host $valeur
	}

Obtenez plus d’informations via :
	
	help about_do
	help about_while



