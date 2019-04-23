+++
date = "2019-04-24T15:29:49+02:00"
draft = "false"
title = "Tester son infrastructure avec Inspec"

+++

Inspec est un outil de test venant de la même équipe à l'origine de **Chef** et **Habitat** et partage au moins un point commun : c'est en Ruby.
N'ayant jamais essayé les autres outils, je ne me lancerai sur de la comparaison entre **Chef** et **Ansible**/**Salstack** (je l'ai fais avec **Puppet**, mais parce que j'ai déjà joué avec).
Pour en revenir à **Inspec**, celui-ci permet d'effectuer des tests de compliance sur des infrastructures cloud, bien qu'il permette également d'effectuer des tests sur des serveurs classique (sur des conteneurs, ça fonctionne aussi).

# Cloud ou Classique ?
Tout dépend ce que vous souhaitez tester, mais le site officiel propose une documentation très complète et les sections **Resources** et **Matchers** suffisent dans de nombreux cas.  
Cependant si vous l'utilisez dans le cadre d'une stratégie de tests à l'aide de junit, vous pouvez également consulter la section **Reporters** qui traite du sujet.  

## Classique
Dans le cas de tests infrastructure sur un serveur classique, la liste des resources Inspec **OS RESOURCES** couvre de nombreux cas d'utilisation tels que :  
- apache  
- apt  
- bash  
- directory  
- docker  
- npm  
- package  
- postgresql

Par exemple :
```ruby
describe powershell(script) do
  its('property') { should eq 'output' }
end

describe bash('command') do
  it { should exist }
  its('property') { should eq 'expected value' }
end

describe directory('path') do
  its('property') { should cmp 'value' }
end

describe user('root') do
  it { should exist }
  its('uid') { should eq 1234 }
  its('gid') { should eq 1234 }
  its('group') { should eq 'root' }
  its('groups') { should eq ['root', 'other']}
  its('home') { should eq '/root' }
  its('shell') { should eq '/bin/bash' }
  its('mindays') { should eq 0 }
  its('maxdays') { should eq 90 }
  its('warndays') { should eq 8 }
end
```

## Cloud
Certes, le nombre de fournisseur d'infrastructure Cloud est...conséquent, mais **Inspec** n'en propose que trois, les plus courants :  
- Azure  
- GCP  
- AWS  
Avec ces trois là, je pense que l'on peut couvrir une grande partie du marché.  
Dans l'idée, tester son infrastructure après l'avoir montée à l'aide de **Terraform** n'a absolument aucun sens ni aucun intérêt (je suis tout à fait d'accord avec vous) étant donné que **Terraform** fait tellement bien son job qu'il crée uniquement ce qu'on lui demande (même si on se trompe, il y a toujours un moyen de le vérifier et de supprimer la ressource en question).  
Mais là ou ça devient intéressant, c'est lorsque l'on a des différences entre le contenu du fichier *tfstate* sur trouvant sur le backend du cloud provider et ce qui est toujours présent sur notre compte cloud. Evidemment, Le fichier *tfstate* étant un fichier json relativement classique, il est possible de le parser dans **Inspec**...mais pas directement depuis le backend (sinon, ce serait bien trop simple), c'est à ce moment que l'on se rend compte que deux commande **Terraform**, à priori inutile, montrent toute leur utilité :  

- terraform refresh  
- terraform state pull  

Le premier permet de rétablir la connexion avec le provider en lui fournissant les fichiers de variables qui vous ont permit de monter votre infrastructure cloud, tandis que le second télécharge le fichier tfstate du backend pour le stocker dans un dossier.

### ATTENTION
**Inspec** est très...*tatillon* et ne cherchera le tfstate qu'a un seul endroit pour le parser lors des tests d'infrastructure : dans le dossier *files* du profil que vous avez créé au préalable.

J'avoue n'avoir testé Inspec qu'avec le cloud **Azure**...mais je ferais un update lorsque je ferais de même avec **Google Cloud Engine**.  

Revenons notre fichier *tfstate*...et l'idée de le parser...Comment faire ?  
Essayez de ne pas pleurer devant tant de simplicité (comme j'ai pu le faire en installant une appli sur un mac) :  
```ruby
params = JSON.parse(inspec.profile.file('terraform.tfstate'))
```

Certes, ce n'est pas terminé...il faut maintenant *décortiquer* chaque ressources à tester avec **Inspec**, mais je vous rassure ce n'est pas non plus très compliqué :  
```ruby
control 'virtual servers' do
  title 'Azure Resources tests'
  desc 'Azure Resources Tests'
  (params['modules'][-1]['resources']).each do | tests |
    if tests[1]['type'] == 'azurerm_virtual_machine'
      name      = tests[1]['primary']['attributes']['name']
      rg_name   = tests[1]['primary']['attributes']['resource_group_name']
      location  = tests[1]['primary']['attributes']['location']
      id        = tests[1]['primary']['attributes']['id']

      describe azurerm_virtual_machine(resource_group: rg_name, name: name) do
        it                { should exist }
        its('name')       { should eq name}
        its('location')   { should eq location }
        its('id')         { should eq id }
      end

      describe azurerm_virtual_machine(resource_group: 'test', name: name) do
        it                { should_not exist }
      end

      describe azurerm_virtual_machine(resource_group: rg_name, name: 'test') do
        it                { should_not exist }
      end
    end
  end
end
```


Et ceci n'est qu'un exemple de la simplicité offerte par l'outil qui, étrangement, ne permet pas de tester tout les types de PaaS autre que **SQL Server** (exit donc les MySQL ou PostgreSQL pour le moment), mais il y a tellement d'autres ressources testables sur **Azure**.  
Je ferais un update sur cette article une fois **Inspec/GOOGLE CLOUD** testé...mais **Inspec** est très séduisant et très puissant (et accessoirement, à part **Kitchen**, je ne crois pas qu'il y ait d'autre outil dans le même genre).
