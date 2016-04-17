+++
date = "2016-03-15"
draft = false
title = "Powershell #3"
+++

## Les fichiers XML

Il est important de savoir que Powershell peut générer à la volée du fichier XML et traiter ces mêmes fichiers très facilement en utilisant le type de fichier XML.

La génération de fichier XML
Pour générer un fichier XML, il faut entrer une série de commandes permettant de définir le nom du fichier, de créer le document, d'écrire dans le document les informations nécessaires et, enfin, de le clôturer : 
	
	Définir le nom du fichier 			$filePath = "c:\command.xml"
	Créer le document 					New-Object System.XMl.XmlTextWriter($filePath,$Null)
	Ecrire dans le document 			$xmlWriter.WriteRaw (Cette commande permet de créer du contenu totalement personnalisé comme du XML SOAP)
	Finaliser le document 				$xmlWriter.Finalize
										$xmlWriter.Close()

Par exemple : 

	$filePath = "c:\command.xml"
	$XmlWriter = New-Object System.XMl.XmlTextWriter($filePath,$Null)
	$xmlWriter.WriteRaw("$XMLheader")
	$XmlWriter.WriteRaw("$XMLBody")
	$XmlWriter.WriteRaw("</soapenv:Envelope>")
 	$xmlWriter.Finalize
	$xmlWriter.Close()

Dans cet exemple, deux variables contenant le header et le body du fichier xml, cela permet de n'avoir qu'un seul fichier "command.xml" à créer et, donc, avoir une plus grand flxibilité et un code un peu plus léger que si le fichier xml devait être généré a chaque fois que c'était nécessaire.

Cependant, il est tout à fait possible de stocker le fichier XML dans une variable typée, de cette façon : 

	[xml]$variable = "contenu du fichier XML"

Le parsage de fichier xml
Pour parser un fichier xml et récupérer une donnée bien précise, il faut d'abord commencer par définir une valeur et utiliser la commande "Get-Content" : 
	
	[xml]$[variable] = Get-Content [nom du fichier].xml
		
Il faut faire attention à bien préciser le type de données à extraire et utiliser [xml] dans ce cas là. Une fois cette commande lancée, on procède par étape avant d'arriver au resultat donné : 
	
	$[variable].[étape_1].[étape_2].[étape3].etc...

par exemple : $serverlist.Envelope.Body.GetServersResponse.GetServersResult.Value.Server 	Permet d'obtenir la liste des serveurs sous forme de tableau.
		
De plus, il est possible d'affiner le résultat en n'affichante que certaines données de la sélection à l'aide de la commande "Select-Object" : 
	
	$[variable].[étape_1].[étape_2].[étape3].etc... | Select-Object [Objet_1], [Objet_2], [Objet_3], etc...
		
Dans le cas où l'on sélectionne un élément de la liste pour, par exemple, obtenir plus de détails, il faut procéder avec la commande "Where-Object" : 
	
	$[variable].[étape_1].[étape_2].[étape3].etc... | Where-Object {$_.element -match $[valeur]}

Il est possible aussi de combiner Where-Object et Select-Object comme dans l'exemple ci-dessous : 
	
	$serverlist.Envelope.Body.GetServersResponse.GetServersResult.Value.Server | Where-Object {$_.ServerId -match $serverid} | Select-Object -Property Name, ServerId, HypervisorServerType 

Insérer des données dans le fichier XML
Pour insérer des données dans un fichier XML, il faut procéder ainsi : 
	
	Créer la variable contenant le chemin des données XML 													$[variable_A] = @($xml.chemin.vers.les.données)[0]
	Créer une nouvelle variable qui contiendra la nouvelle donnée et utiliser la fonction ".Clone"			$[Variable_B] = $[Variable_A].Clone()
	Affecter une donnée à la variable nouvellement créée 													$[Variable_B].chemin.vers.les.données = "Donnée_A"
	Recommencer l'opération si nécessaire
	Ajouter les nouvelles données au fichier XML avec ".AppendChild"										$xml.chemin.vers.les.données.AppendChild([Variable_B])
	Sauvegarder le fichier XML avec ".Save"
