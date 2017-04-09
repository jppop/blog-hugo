+++
banner = "/images/spring-modularity.png"
menu = ""
description = "Spring config et modularité"
date = "2017-04-10T00:56:56+02:00"
title = "Spring config et modularité"
draft = false
disable_profile = true
categories = ["développement"]
tags = ["spring", "java"]
images = []
+++

Comment éviter que l'injection de dépendances deviennent un plat de spaghettis.

<!--more-->

## Le problème

Il y a quelques temps, je suis intervenu sur un projet Java composé de plusieurs modules Maven.
Nous développions une application de bureau en JavaFX embarquant une base données synchronisée avec des données serveurs via des API REST.
Dans ce projet, **Spring Configuration** est utilisé pour injecter les dépendances.

<div class="note">
L'injection de dépendances est utilisée dans ce projet pour pouvoir remplacer simplement certaines implémentations par des implementations de tests ("bouchons", _mockup_). Comme dans la plupart des cas.
</div>

Le module Maven de l'application principale inclut les autres modules et configure l'injection des dépendances. De _tous_ les modules. Plusieurs fichiers de configuration Spring ``applicationContext*.xml`` ainsi que plusieurs fichiers ``persistence.xml``.

Dans les autres modules, des tests unitaires et d'intégration sont implémentés. Là encore, les développeurs ont inclus des fichiers de configuration Spring. Les mêmes ou presque que ceux du module de l'application principale.

Finalement, le projet s'est retrouvé avec une multitude de fichiers de configuration : 28 fichiers pour une demi douzaine de modules utilisant l'injection.
Pire, certains développeurs, pour éviter certainement de déclarer à nouveau les injections, ont développé des tests dans des modules qui les déclaraient déjà mais qui n'avait rien à voir avec la nature des tests.

## La solution
<img src="/images/dry-soc-low.jpg" alt="DRY - Separation of Concerns" style="width: 50%">

Evidemment, il s'agit d'un problème couvert par les patterns **_Don't Repeat Yourself_** et, surtout, **_Separation of concerns_**. Séparation des responsabilités parce que l'injection des dépendances d'un module ne devrait être que de sa responsabilité. Un module implémentant une couche d'accès aux données, par exemple, a, _lui seul_, la responsabilité des bibliothèques qu'il utilise.

La plupart des moteurs d'injection (mais pas ceux à injection), Guice, HK2 et Spring Configuration, permettent facilement de résoudre ce problème.
Pour Spring, il suffit d'utiliser la possibilité d'importer des définitions.

Par exemple, dans un module "client" :

```java
@Configuration
@Import({ DataConfig.class, CommonConfig.class, SyncConfig.class })
public class AppConfig {

  // ... autres définitions propres à l'application
}
```

Les classes de configuration importées, ``DataConfig``, ``CommonConfig`` et ``SyncConfig``, proviennent d'autres modules (d'autres jars). Chacune de ces classes définissent leurs dépendances.
Et ``SyncConfig`` définit des dépendances (d'autres modules du projet) :

```java
@Configuration
@Import({ CommonConfig.class, RepositoryConfig.class,  NomadeJaxRsServiceConfig.class })
@Lazy
public class SyncConfig {
  // ...
}
```

Et si on a besoin de remplacer une implémentation dans un module client ? Il suffit de surcharger la défition du bean, avec Spring, c'est celui qui parle en dernier qui a raison.

Par exemple, j'ai besoin d'une configuration où je veux tous les services sauf la base de données (utilisation de PostgreSQL embarquée de test) et un service qui ne synchronise pas les données modifiées localement (le bean Uploader).

```java
@Configuration
@Profile("mock")
@Import({AppConfig.class, EmbeddedPgConfig.class})
@Lazy
public class MockConfig {

	@Bean
	public Uploader uploader() {
		return new MockUploaderImpl();
	}

}
```

La classe de configuration ``MockConfig`` définit tous les services "réels" en important la classe ``AppConfig``, importe une classe de configuration, ``EmbeddedPgConfig`` qui va modifier la configuration de PostgreSQL et redéfinit le bean Uploader en le remplaçant par un _mock_.

## Conclusion

Voila. Le but est atteint :

- La configuration des dépendances n'est pas répétée deux fois.
- Chaque module définit ses propres dépendances dans une classe qu'il expose.
