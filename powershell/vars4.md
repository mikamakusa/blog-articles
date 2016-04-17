+++
date = "2016-03-21"
draft = false
title = "Powershell #5 - Silent Download and Install"
+++

Suite de la série d'articles sur Powershell publiés la semaine dernière (et parce que, à la base, je suis un Windowsien), voici un petit trick bien utile pour ceux qui veulent télécharger un fichier exécutable sans avoir à ouvrir de navigateur...en gros : pour faire comme nos amis linuxiens en ligne de commande.  
Par contre, il faut connaître l'URL de la ressource, sinon ça ne sert à rien (Captain Obvious...)

Il nous faudra, tout d'abord, le script suivant à intégrer dans le vôtre (Ce n'est pas le mien, je l'ai juste trouvé sur le web et je le trouve bien pratique) : 

	function Install-MSIFile {
	    [CmdletBinding()]
	    Param(
	        [parameter(mandatory=$true,ValueFromPipeline=$true,ValueFromPipelinebyPropertyName=$true)][ValidateNotNullorEmpty()][string]$msiFile,
	        [parameter()][ValidateNotNullorEmpty()][string]$targetDir
	        )
	    if (!(Test-Path $msiFile)){
	        throw "Path to the MSI File $($msiFile) is invalid. Please supply a valid MSI file"
	    }
	    $arguments = @(
	        "/i"
	        "`"$msiFile`""
	        "/qn"
	        "/norestart"
	    )
	    if ($targetDir){
	        if (!(Test-Path $targetDir)){
	            throw "Path to the Installation Directory $($targetDir) is invalid. Please supply a valid installation directory"
	        }
	        $arguments += "INSTALLDIR=`"$targetDir`""
	    }
	    Write-Verbose "Installing $msiFile....."
	    $process = Start-Process -FilePath msiexec.exe -ArgumentList $arguments -Wait -PassThru
	    if ($process.ExitCode -eq 0){
	        Write-Verbose "$msiFile has been successfully installed"
	    }
	    else {
	        Write-Verbose "installer exit code  $($process.ExitCode) for file  $($msifile)"
	    }
	}

Ensuite, on téléchage le fichier à l'aide de la commande suivante : 

<code>Invoke-WebRequest -Uri *http://Chemin-du-fichier* -Outfile *Chemin du fichier*</code>

Une fois le fichier téléchargé, on lance la commande suivante : 

- Si c'est un MSI : 

	Install-MSIFile *Chemin du fichier*; Remove-Item *Chemin du fichier*

- Si c'est un EXE : 

	Start-Process *Chemin du fichier* -ArgumentList "/s" -Wait; Remove-Item *Chemin du fichier*

Une fois installé, le programme d'installation sera supprimé...tout simplement.
