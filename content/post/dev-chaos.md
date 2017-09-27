+++
draft = true
images = []
banner = ""
menu = ""
description = ""
date = "2017-09-27T17:18:55+02:00"
title = "Chauchemard en DEV-TU"
categories = ["internal"]
tags = ["dev", "java", "spring"]

+++
<br/>
<div class="note">
<strong>Avertissement : post à usage interne. Cet article s'inscrit dans un contexte particulier, privé.</strong>
</div>

La fabrication du SI Nice pose de nombreux problèmes aux développeurs dès que, et c’est toujours le cas, plusieurs équipes sont impliquées. Par exemple, une équipe pour développer le front, une pour le back. Et on ajoute une troisième, les utilisateurs qui testent le tout. Comment éviter que ces trois-là ne se fassent pas la guerre ?
Le développement en parallèle n’est pas une nouveauté. Des solutions éprouvées existent. Comment les adapter à la fabrication du SI Nice ? Comment, pratiquement, éviter de se gêner les uns les autres pendant le build et le run ?

<!--more-->
## Problème

Le front consomme des ressources produites par le back. Dans la phase de développement, les ressources évoluent très vite (jusqu’à plusieurs fois par jour) alors que les consommateurs ont besoin de stabilité.
L’objectif est donc de proposer au front des ressources stables tout en permettant aux développeurs du back de les faire évoluer rapidement. En d’autres termes, il faut que chaque ressource exposée dispose de deux cycles de vie : celui des développeurs et celui des consommateurs.

## Des solutions

Une solution simple pour proposer deux cycles de vie est de disposer de deux environnements de développement : un pour le front et un pour le back. La plateforme de fabrication distingue déjà deux environnements : les branches A et B. Il me semble difficile d’en introduire facilement d’autres. Par contre, il existe déjà un environnement justement prévu pour intégrer un consommateur et un producteur : la plateforme d’intégration, la VMOE.

La première piste consiste donc à utiliser la VMOE pour pousser les ressources en garantissant aux consommateurs une certaine stabilité. Le problème est résolu coté développeur du back (il est quand même nécessaire d’adopter les bonnes pratiques de gestions des sources, voir plus loin, la livraison). Mais il est transféré coté front : comment, en DEV-TU, consommer des ressources de la VMOE ?

Une deuxième piste, permettant à tous de rester dans l’environnement DEV-TU, consiste à mettre à disposition des consommateurs une autre ressource, stable dans le temps : un bouchon. Là encore, il faut résoudre le problème « comment consommer ce bouchon ».

Avant de voir comment résoudre pratiquement l’utilisation d’une ressource différente (bouchon ou VMOE), il est important d’examiner comment les ressources sont consommées par le front (on se limitera au PUC pour l’instant).

### Consommation des services SOA

Une UA consomme un service en utilisant un clieant SOAP : les classes Java générées par un plugin Eclipse. Le _endpoint_ du service est configuré dans le fichier de contexte Spring (applicationContext.xml) :

```xml
<todo/>
```
