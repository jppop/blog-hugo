+++
draft = true
banner = ""
menu = ""
description = ""
categories = ["développement"]
date = "2017-10-18T22:57:00+02:00"
title = "unit testing"
tags = ["dev", "java", "test", "mock"]
images = []

+++

# Cas pratique : test unitaire avec Mockito

La bibliothèque [Mockito](http://site.mockito.org/) permet de simuler des objets réels (service web, DAO, etc).
Nous n'allons pas voir l'ensemble des possibilités offertes par Mockito mais présenter concrètement comment ce framework a été utilisé dans un cas réél.

<div class="note">
Certains noms ont été changés. Tout d'abord pour respecter des clauses de confidentialité (secteur bancaire). Mais aussi parce que, comme souvent, le code original nommait mal les choses. J'attache une grande importance aux noms. Un nom mal choisi entraîne de la confusion et parfois de la suspicion (le développeur a-t-il réellement compris ?).
</div>

> Il y a seulement deux choses de difficile en informatique : invalider un cache et nommer les choses.

> -- Phil Karlton (cf un [post de Martin Fowler](https://martinfowler.com/bliki/TwoHardThings.html))

Il s'agit de tester une classe permettant d'envoyer un mail de confirmation d'inscription à un service. La méthode principale est `sendConfirmation`. Cette méthode construit le mail à partir d'un modèle contenant des parties variables (civilité, nom, date, etc.). Elle s'appuie sur trois services. `ProfileService` pour obtenir des informations sur le destinataire, `TemplateEngine` pour construire le mail (html) et `MailService`pour envoyer le mail :
```java
public class ConfirmMailService {

  @Inject
  private MailService mailService;

  @Inject
  private UserProfileService profileService;

  @Inject
  private TemplateEngine templateEngine;

  public String sendConfirmation(final UserInfo userInfo) {
    ...
  }

}
```
## Dépendances
Pour commencer il faut ajouter la dépendance à Mockito :
```xml
TBD: insert pom
```

##

<!--more-->
