+++
date = "2016-04-08T11:37:49+02:00"
draft = false
title = "Powershell #6 | Intéractions avec VirtualBox"

+++

Powershell...c'est bon, mangez-en !

Concernant Powershell, je ne suis plus vraiment un novice en la matière...je vous invite à consulter mon Github et découvrir mon projet actuel.
Mais, je n'avais pas encoré tenté d'intéragir avec VirtualBox.

J'ai trouvé quelques commandes qui peuvent être utiles, sans passer par vboxmanage (sinon, c'est presque de la triche).

On commence par démarrer VirtualBox avec la commande suivante : <code>$vbox = New-Object -ComObject "VirtualBox.virtualBox"</code>  
Pour lister les VM existantes : <code>$vbox.Machines</code>  
Cette commande va lister toutes les VM, qu'elles soient allumées ou non. L'attribut *SessionState* est relatif à l'état du serveur, parmis les suivants : 

	1 : Stopped
	2 : Saved
	3 : Teleported
	4 : Aborted
	5 : Running
	6 : Paused
	7 : Stuck
	8 : Snapshotting
	9 : Starting
	10 : Stopping
	11 : Restoring
	12 : TeleportingPausedVM
	13 : TeleportingIn
	14 : FaultTolerantSync
	15 : DeletingSnapshotOnline
	16 : DeletingSnapshot
	17 : SettingUp

Si vous souhaitez intéragir avec une VM en particulier, il est préférable d'utiliser l'id de la VM. Vous pourrez l'obtenir à l'aide de la commande suivante :  
<code>$vmachine = $vbox.FindMachine(($vbox.Machines | where {$_.Name -match "VMName"}).id</code>

Les actions possibles :  
Pour démarrer une VM : <code>$vmachine.LaunchVMProcess(New-Object -ComObject "VirtualBox.Session","headless","")</code>  
Pour suspendre une VM : <code>$vmachine.LockMachine(New-Object -ComObject "VirtualBox.Session",1);((New-Object -ComObject "VirtualBox.Session").Console.SaveState())</code>  
Pour eteindre une VM : <code>$vmachine.LockMachine(New-Object -ComObject "VirtualBox.Session",1);((New-Object -ComObject "VirtualBox.Session").console.PowerButton())</code>  

En bonus...parce que j'ai eu un peu de mal : Récupérer l'adresse IP et le port SSH d'un VM.

	foreach ($i in (& 'C:\Program Files\Oracle\VirtualBox\VBoxManage.exe' showvminfo "default" --machinereadable)) {
        if ($i -match "ssh") {
            return $i.split(",")
            $IP = $i.Split(",")[2]
            $Port = $i.Split(",")[3]
        }
    }
