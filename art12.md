+++
date = "2020-04-28T15:29:49+02:00"
draft = false
title = "les opérateurs ternaires"
+++

## Intro
**Mais qu'est-ce qu'il dit le monsieur ?**  
Cela doit certainement être la question à 10000 € que vous vous posez après avoir lu le titre de l'article...Et je vais voir avouer qu'après en avoir vu ça dans une conversation sur **Slack** je me suis dis la même chose.  
Dit comme ça **Opérateurs Ternaire**, ça à l'air étrange, ça sonne un peu comme du chinois pour ceux qui, un peu comme moi, ne sont pas très familiarisés avec le développement applicatif.  
Et pourtant, si vous avez joué assez intensivement avec **Terraform** vous en avez déjà vu: *les boucles if/else*.  

`var.a != "" ? var.a : "default-a"` 
Ou plus concrètement : `source_security_group_id = lookup(var.security_group_rule[count.index], "cidr_blocks") == "" ? var.source_security_group_id : null` (dans le cas d'une création d'une règle de security group sur **AWS**).  

Après avoir vu ça sur du **groovy** et sur **terraform**, j'ai décidé de creuser un peu plus en direction des langages sur lesquels j'avais déjà eu l'occasion de travailler, à savoir **Powershell** (je suis un ancien Windowsien totalement assumé), **Python** et **Ruby**.

## Python
Sur **Python** il y a plusieurs manières d'implémenter des *opérateurs ternaires*: 

- `condition_if_true if condition else condition_if_false`
Ou bien  
- `(if_test_is_false, if_test_is_true)[test]`

En conditions réelles, ça donnerait ça : 
```python
is_nice = True
state = "nice" if is_nice else "not nice"
```

ou  
```python
nice = True
personality = ("mean", "nice")[nice]
print("the cat is ", personality)
```

## Ruby
Sur **Ruby** c'est plus simple, il n'y a qu'une seule syntaxe : `test_expression ? if_true : if_false` (cela ressemble à la syntaxe de **Terraform**)

En conditions réelles, ça donnerait ça :  
```ruby
var = 5
is_greater = (var > 3 ? true : false)
```

## Powershell
Sur **Powershell**, étrangement cela ressemble un peu à la syntaxe du **Ruby**. Rassurez-vous, je ne fais allusion qu'aux opérateurs ternaires, voici la syntaxe : `(test_expression) ? $true : $false`

En conditions réelles, cela donnerait ça :  
```
(Test-Path $Path) ? "Path Exist" : "Path not found"
```

## Groovy
Au départ, c'est pour ce langage que j'avais effectué des recherches sur ce type de syntaxe (dans l'optique des pipelines sur **Jenkins**), la syntaxe: `(condition_test) ? if_true: if false` ou si vous n'avez pas de value pour *if_false* : `(condition_test) ?: if_true`.

