+++

date = "2016-04-14T11:37:49+02:00"
draft = false
title = "Powershell vs Python | Parser du JSON"

+++

Je me suis lancé sur Powershell il y a un peu moins de 3 ans pour un projet en rapport avec du provisionning d'instances Cloud avec l'API d'ArubaCloud...et par la suite, je me suis demandé si il n'était pas possible de faire de même avec d'autres API...tel que celle de DigitalOcean, par exemple.  
Mais dans le même laps de temps, j'ai joué un peu avec Python.  

Certes, Python et Powershell sont complètement différents.  
Le premier ne tourne que sur Windows tandis que le second tourne aussi bien sur Windows que sur Linux et à l'avantage d'être maintenu et agrémenté de modules par une communauté toujours grandissante (Pendant que Powershell évolue grâce à Microsoft, principalement).  

Toujours est-il que comparer ces deux languages de scripting pourrait paraître totalement futile si le but final n'était pas d'observer une méthode de travail sur un type d'objet : le JSON

Je ne ferais pas de cours sur le JSON, vous trouverez ça sur wikipedia ou sur d'autres sites.  
A la place, je me contenterai de vous décrire les méthodes pour extraire les données dont vous pourrez avoir besoin par la suite...et pour cela, j'ai choisi la fonction "VersionsList" de l'API de numergy.

Le fichier de réponse (en JSON) que l'on peut obtenir à l'aide d'une requête GET est ci-dessous : 

	{
	  "versions": [
	    {
	      "id": "V0.1",
	      "links": [
	        {
	          "href": "http://www.api.com/documentation",
	          "rel": "self",
	          "type": "application/json" 
	        } 
	      ],
	      "status": "DEPRECATED",
	      "updated": "2012-11-24T10:50:25+0200" 
	    },
	    {
	      "id": "V1.1",
	      "links": [
	        {
	          "href": "http://www.api.com/documentation",
	          "rel": "self",
	          "type": "application/json" 
	        } 
	      ],
	      "status": "DEPRECATED",
	      "updated": "2013-11-24T10:50:25+0200" 
	    }, 
	    {
	      "id": "V1.2",
	      "links": [
	        {
	          "href": "http://www.api.com/documentation",
	          "rel": "self",
	          "type": "application/json" 
	        } 
	      ],
	      "status": "CURRENT",
	      "updated": "2014-11-24T10:50:25+0200" 
	    }
	  ] 
	}

En Powershell, ça donne ça : 

	$Nversion = ((Invoke-WebRequest -Uri "https://api2.numergy.com/" -ContentType "application/json; charset=utf-8" -Method Get | ConvertFrom-Json).versions | where "status" -Match "CURRENT").id

Par contre, en Python, ça donnera ça : 

	import requests
	for i in (requests.get("https://api2.numergy.com/")).json():if "CURRENT" in i['status']:version = i['id']

De là à affirmer que Powershell est plus simple d'accès que Python, il n'y a qu'un pas que je ne m'amuserai pas à franchir...car de nombreux développeur python me tomberaient dessus (et ça peut faire peur).
