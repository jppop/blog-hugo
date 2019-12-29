---
title: "Quick Tips -- Maven S01E02"
date: "2019-05-03T11:30:00+02:00"
url: "/slides/maven-part2/"
categories: ["tips"]
tags: ["dev", "java"]
---

class: center, middle

# S01E02 - Maven

## Les plugins, les dépendances, les repositories

---

# Maven - Saison 1

- S01E1 : Le coeur de Maven
- S01E2 : Les plugins, les dépendances, les repositories
- S01E3 : Multi-module
- S01E4 : Describe Once, Build Everywhere
- S01E5 : Packaging

---

# Les plugins du cycle de vie par défaut

Pour rappel, le _lifecyle build_ par défaut [associe des plugins aux différentes phases](http://maven.apache.org/ref/3.6.1/maven-core/default-bindings.html).

Par exemple, pour un _packaging_ de type `jar`

| **phase**              | **plugin goals**                                                  |
| ---------------------- | ----------------------------------------------------------------- |
| process-resources      | org.apache.maven.plugins:maven-resources-plugin:2.6:resources     |
| compile                | org.apache.maven.plugins:maven-compiler-plugin:3.1:compile        |
| process-test-resources | org.apache.maven.plugins:maven-resources-plugin:2.6:testResources |
| test-compile           | org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile    |
| test                   | org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test        |
| package                | org.apache.maven.plugins:maven-jar-plugin:2.4:jar                 |
| install                | org.apache.maven.plugins:maven-install-plugin:2.4:install         |
| deploy                 | org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy           |

---

# Configuration du plugin `compiler`

.smaller[Exemple : compiler en 1.8]

```xml
<project>
  [...]
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>
  [...]
</project>
```

.smaller[ou]

```xml
<project>
  [...]
  <build>
    [...]
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
    </plugins>
    [...]
  </build>
  [...]
</project>
```

---

# Help !

Comment se rappeler de toutes les options de tous les plugins ?

Tous les plugins les plus communs sont documentés en ligne.

Par exemple, le plugin [compiler](https://maven.apache.org/plugins/maven-compiler-plugin/index.html)

---

# Configuration d'un plugin

```xml
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-enforcer-plugin</artifactId>
        <version>3.0.0-M2</version>
        <executions>
          <execution>
            <id>enforce-property</id>
            <phase>validate</phase>
            <goals>
              <goal>enforce</goal>
            </goals>
            <configuration>
              <rules>
                <requireProperty>
                  <property>port</property>
                  <message>"port number must be specified."</message>
                  <regex>\d+</regex>
                  <regexMessage>"The port property must contain at least one digit."</regexMessage>
                </requireProperty>
              </rules>
              <fail>true</fail>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```

---

# Un privilégié : le plugin `resources`

Copie les ressources principales (goal resource) et de test (goal testResources) du projet.
Ils accèdent aux éléments du pom `project.build.resources` et `project.build.testResources`.

```xml
<project>
  ...
  <name>My Resources Plugin Practice Project</name>
  ...
  <build>
    ...
    <resources>
      <resource>
        <directory>src/main/resources</directory>
      </resource>
      ...
    </resources>
    ...
  </build>
  ...
</project>
```

---

# Resources filtering

Le plugin [`resources`](https://maven.apache.org/plugins/maven-resources-plugin/) peut substituer des variables dans les ressources :

```shell
$ cat src/main/resources/version.properties
project: ${project.name}
version: ${project.version}
```

```xml
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
```

```shell
$ cat target/classes/version.properties
project: My Resources Plugin Practice Project
version: 1.0.0-SNAPSHOT
```

--
Attention aux fichiers binaires !

```xml
    <resources>
      <resource>
        <directory>src/main/resources-non-filtered</directory>
        <filtering>false</filtering>
      </resource>
    </resources>
```

---

# Les dépendences

Les dépendances sont transitives.

Quelques règles pour contrôler quelles dépendances sont inclues :

- .focus[_Dependency mediation_]  
   Si une dépendance apparaît plusieurs fois, la "plus proche" est retenue.  
   Déclarer une dépendance explicitement évite des suprises.

--

- .focus[_Dependency management_]  
  Permet de fixer les versions des dépendances trouvées dans le graphe des dépendances transitives ou quand la version n'est pas spécifiée.

--

- .focus[_Dependency scope_]  
  Pour une dépendance spécifique à une étape du build (détaillé juste après).

--

- .focus[_Excluded dependencies_]  
  Exclusion explicite d'une dépendance transitive.

--

- .focus[_Optional dependencies_]  
  Les dépendances optionnelles sont exclues par défaut. Le projet doit l'inclure explicitement.

--
.focus[

```shell
mvn dependency:tree
```

]

---

# Dependency scope

- .focus-high[compile]  
  Nécessaire à la compilation.

- .focus-high[provided]  
  Idem mais non empaquetée car fournit par le JDK ou le "container" à l'exécution.

- .focus[runtime]  
  Pas nécessaire à la compilation mais doit être présent dans le _classpath_ à l'exécution.

- .focus-high[test]  
  Pour les tests uniquement.

- .focus[system]  
  Idem que `provided` mais le chemin du jar doit être fourni.

- .focus-high[import]  
  Pour les dépendances de type `pom` et inclues dans la section `<dependencyManagement>`. La dépendance importée est remplacée par la liste qu'elle définit.

---

# Déclaration d'un dépendance

```xml
<dependencies>

  <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.9</version>
  </dependency>

  <dependency>
    <groupId>my.project</groupId>
    <artifactId>configuration</artifactId>
    <version>${project.version}</version>
    <type>zip</type>
  </dependency>

  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
  </dependency>

</dependencies>
```

.focus[
Pour trouver une version, consultez le [catalogue en ligne](https://search.maven.org/)
]

---

# Les _repositories_

Une _repository_ est un catalogue de dépendance.  
Deux types de repo : _local_ et _remote_.
Elles sont structurées de la même manière (groupId/articfactId/version)

### Local repository.

Elle agit comme un cache local. Elle est stockée sous `%USERPROFILE%/.m2/repository` (`$HOME`)

### Remote repository.

Accédée par HTTP:// (ou FTP:// ou FILE://).  
La repo par défaut est la _central_ : [repo.maven.apache.org](http://repo.maven.apache.org/maven2/).

Généralement, d'autres repo sont nécessaires : internes à une entreprise ou à un projet.

---

# To Be Continued

Episode suivant : le multi-module.
