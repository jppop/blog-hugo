+++
title = "good to great"
draft = true
banner = "/images/lechat-megalo.gif"
date = "2017-04-08T11:09:35+02:00"
disable_profile = true
disable_widgets = false
categories = ["idées"]
tags = ["quality"]
+++

Pourquoi ce blog.
<!--more-->

# Améliorer la qualité

Cela fait quelques temps que j'ai envie d'écrire un blog pour partager quelques trucs simples et pratiques essentiellement destinés aux développeurs débutant.
J'ai choisi [Hugo](http://gohugo.io/) pour l'écrire. Son tuto commence par un post "_Good to Great_" qui est le debut d'une critique du livre du même nom. Je n'ai pas lu le livre mais il m'a donné envie de réfléchir comment, à notre niveau de développeur (le livre semble s'adresser à des managers), nous pouvons **améliorer la qualité** de ce que nous produisons.

## La qualité ?

La qualité dont il est question ici est bien sûr la qualité logicielle. La [page Wikipedia sur le sujet](https://fr.wikipedia.org/wiki/Qualit%C3%A9_logicielle) définit très clairement ce qu'est la qualité logicielle et les critères qui permettent de la mesurer (définis par des normes dont ISO/IEC 25010).

Ce qui peut paraître étrange, c'est que la "mauvaise" qualité logicielle est un problème décrit depuis 1960 :

> Un phénomène de baisse des prix du matériel informatique et d'augmentation des prix du logiciel, accompagné d'une baisse de la qualité des logiciels a été identifié à la fin des années 1960.

> -- <cite>[Wikipedia, La qualité logicielle][1]</cite>
[1]: https://fr.wikipedia.org/wiki/Qualit%C3%A9_logicielle

## Difficile de faire de la qualité

La mauvaise qualité logicielle est donc un problème connu et assez généralisé. Certains facteurs de ce problème sont inhérents au métier même de la production logicielle : difficulté d'exprimer et de comprendre un besoin, impossibilité de ne pas faire d'erreur en programmation et de le vérifier, erreur de conception, etc.

Il existe cependant des moyens (des méthodes de productions logicielle, de contrôle de qualité, ...), connus de tous, qui permettent de diminuer les cas où un logiciel n'atteint pas ses objectifs de qualité.

D'autres moyens "émergent" comme le mouvement **_DevOps_**. Certaines compagnies arrivent, par exemple, à déployer leur logiciel en production très fréquemment. Par exemple [Facebook](http://swreflections.blogspot.fr/2013/09/this-is-how-facebook-develops-and.html) s'appuie, en autre, sur la revue de code, les tests automatisés et responsabilise fortement chaque développeur pour la mise en production de ce qu'il a développé. Le [Guardian](https://www.theguardian.com/info/developer-blog/2015/jan/05/delivering-continuous-delivery-continuously) décrit aussi comment ils ont amélioré la qualité de leur site en "responsabilisant" leurs développeurs.

Bien. Que ce soit des moyens anciens (tests automatisés, revue de code, intégration continue) ou nouveau, la réalité est quelque peu différente dans nos projets de tous les jours.
Parce que nous constatons que tous ces "principes" ne sont souvent pas mis en oeuvre ou que partiellement. Et dans la plupart des cas, par manque de temps.

## Prix tirés vers le bas

Notre métier, spécialement en SSII, est soumis à une très grande concurrence, nous devons constamment tirer les prix vers le bas.

A titre d'exemple, je prendrai le taux que nous appliquons sur les coûts "Développement + Tests" pour obtenir le coût total du développement d'un projet Ce taux permet d'obtenir le coût total en fonction d'abaques. Par expérience, nous savons que pour produire 1 jour de code, il faut avoir produit 0,6 jours de spécifications, il faudra produire 0,4 jour de tests d'intégration. Et d'autres taux encore pour la qualification, le packaging, la VABF, VSR et le pilotage. Bref, ce taux est aujourd'hui (dans ma boîte en tout cas) proche de 2. Il y a 15 ans, il était de 3.

Un gain de productivité de 66% ! Qu'est ce qui a pu nous faire gagner autant ? De meilleurs outils ? Des meilleurs frameworks ? Certes, il y eu des amelioration mais jamais dans de telle proportions.

C'est tout simplement que nous avons baissé nos prix de ventes mais en prétendant faire toujours la même qualité.

## Rester positif

Je pourrais continuer à developper ma vision de ce qu'est devenu notre métier mais je préfère réfléchir à comment continuer à nous améliorer (c'est moins déprimant). Proposer des idées simples, avec les moyens à notre disposition, pour tenter de produire des logiciels de qualité.

## Quelques idées (à développer)

Je continuerai à proposer des idées et à les poster ici.

Des premières pistes :
<dl>
  <dt>Développement ouvert</dt>
	<dd>
	Le modèle courant des développements en SSII (et chez nos clients) est le modèle fermé. Je suis convaincu que si nous développions tout ou partie en mode ouvert (_Open source_), nous pourrions sans doute améliorer la qualité de nos développements mais, aussi, réduire nos coûts en favorisant la réutilisation. Pour se convaincre, on peut penser aux [projets open source](https://github.com/explore) de Google (Guice, Guava et beaucoup d'autres), [Netflix](https://github.com/Netflix/), à [React](https://github.com/facebook/react), [React Native](https://github.com/facebook/react-native) de Facebook, [VS Code](https://github.com/Microsoft/vscode), [TypeScript](https://github.com/Microsoft/TypeScript) de Microsoft. Et aussi Backbone.js (développé par DocumentCloud, un regroupement de journaux, dont le New York Times).</br>
	_à developer dans un autre post_
	</dd>
	<dt>Documentation</dt>
	<dd>
		Une piste sur les outils de documentations techniques (MS Word nous fait perdre du temps, AcsiiDoctor permet d'être beaucoup plus productif).</br>
		_à developer dans un autre post_
	</dd>
</dl>

To be continued.
