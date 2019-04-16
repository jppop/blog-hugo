---
title: "Quick Tips -- Refactoring"
date: "2019-04-13T16:43:02-04:00"
url: "/slides/refactoring/"
---

class: center, middle

# Refactoring

## Ou comment éviter le copier/coller (et faciliter la maintenance)

---

# T’es sûr, c’est mal copier/coller ?

- .focus[Rapide], sûr (ca marche déjà)
- Quand utiliser le copier/coller ?
  - .focus[Forking]
  - .focus[Templating]
  - .focus[Tests, exploration]

--

![alt text](./copy-own.png "Copy and own")

.see-also[
Pour approfondir : [Is Copy and Paste Programming Really a Problem?](https://dzone.com/articles/copy-and-paste-programming)]

---

# Mais

Fais quelque chose si on est on est un jour ouvré :

```java
public void doSomeThingAt(LocalDate when) {
  if (isBusinessDay(when)) {
    // ...
  }
}
private boolean isBusinessDay(LocalDate when) {
  return !isHoliday(when) && !isWeekEnd(when);
}
private boolean isHoliday(LocalDate when) {
  // ...
}
```

---

# Mais

Puis ça aussi mais toujours si on est on est un jour ouvré,
mais dépêche-toi, on doit livrer demain :

```java
public void doSomeThingElseAt(LocalDate when) {
  if (isBusinessDay(when)) {
    // ...
  }
}
private boolean isBusinessDay(LocalDate when) {
  return !isHoliday(when) && !isWeekEnd(when);
}
private boolean isHoliday(LocalDate when) {
  // ...
}
```

---

# Mais

J'avais oublié, ça encore :

```java
public void doSomeThingCompletlyDifferentAt(LocalDate when) {
  if (isBusinessDay(when)) {
    // ...
  }
}
private boolean isBusinessDay(LocalDate when) {
  return !isHoliday(when) && !isWeekEnd(when);
}
private boolean isHoliday(LocalDate when) {
  // ...
}
```

--
<img src="./stop.png" alt="stop" width="160px"/>

.focus-high[T'aurais pas oublié le 11 novembre ? C'est férié, le 11 novembre.]

---

# Refactoring

- Règle de trois (three strikes and you refactor)
- Don’t Repeat Yourself
- Separation of Concern
- Low coupling
- Penser global puis local : trop dur

--

- .focus-high[Codez local en pensant global]

--
.see-also[

#### .focus[Design Patterns].

<img src="/images/quotation-open.png" width="16px" height="16px" style="transform: translate(0px, -8px);"/>
In software engineering, a .focus[design pattern] is a general repeatable solution to a commonly occurring problem in software design.
<img src="/images/quotation-close.png" width="16px" height="16px" style="transform: translate(0px, 8px);" />  
<cite>[Source Making](https://sourcemaking.com/design_patterns)</cite>

De Martin Fowler, un [Catalog of Patterns of Enterprise Application Architecture](https://martinfowler.com/eaaCatalog/)
]

---

# Three strikes and you refactor

Déplacez les méthodes dans une autre classe.

```java
public class DateUtil {
  public static boolean isHoliday(LocalDate when) {
    boolean isHoliday = false;
    // ...
    return isHoliday;
  }

  static boolean isWeekEnd(LocalDate when) {
    return when.getDayOfWeek().equals(DayOfWeek.SATURDAY)
      || when.getDayOfWeek().equals(DayOfWeek.SUNDAY);
  }

  static boolean isBusinessDay(LocalDate when) {
    return !isHoliday(when) && !isWeekEnd(when);
  }
}
  // Further ...

  public void doSomeThingAt(LocalDate when) {
    if (DateUtil.isBusinessDay(when)) {
      // ...
    }
  }
```

---

layout:true
name: dry

<img src="./dry.png" alt="drawing" width="400"/>

---

layout:true
name: none

---

layout:false
template: dry

At Reynholm Industries ([The IT Crowd](https://fr.wikipedia.org/wiki/The_IT_Crowd), série britanique):

```java
public class Receptionist {
  public String divisionFromFloor(int floor) {
    switch (floor) {
      case 0:
        return "reception";
      case 1:
        return "accounting";
      case 2:
        return "marketing";
      case -1:
        return "IT";
      default:
        throw new IllegalArgumentException("unknown floor");
    }
  }
  public int floorFromDivision(String division) {
    if ("reception".equals(division)) {
      return 0;
    } else if ("accounting".equals(division)) {
      return 1;
    } else if ("marketing".equals(division)) {
      return 1;
    } else if ("IT".equals(division)) {
      return -1;
    } else {
      throw new IllegalArgumentException("unknown division");
    }
  }
}
```

---

template: dry

.focus-high[Dry or wet ?]

--

<dl>
  <dt>Wet !</dt>
  <dd>Les noms des divisions, les étages sont dupliqués.</dd>
  <dt>Boggué</dt>
  <dd>La compta est annoncée au 1<sup>er</sup> alors qu'elle est au 2<sup>ème</sup>.</dd>
</dl>

---

template: dry

Une solution : utiliser des constantes :

```java
  public static final String ACCOUNTING_DIVISION = "accounting";
  public static final int ACCOUNTING_FLOOR = 1;

  //...

  if (ACCOUNTING_DIVISION.equals(division)) {

  }
```

--
Mais on duplique toujours l'association division/étage dans 2 méthodes.

Toujours un risque d'erreur.

---

template: dry

```java
package me.sample.refactoring.dry;

import java.util.Arrays;
import java.util.Optional;

public class DryReceptionist {

  private static Pair[] divisions = new Pair[]{
    new Pair<>(0, "reception"),
    new Pair<>(1, "accounting"),
    new Pair<>(2, "marketing"),
    new Pair<>(3, "direction"),
    new Pair<>(-1, "IT"),
  };

  public String divisionFromFloor(final int floor) {
    Optional<Pair> pair = Arrays.stream(divisions)
      .filter(p -> p.partA().equals(floor))
      .findFirst();
    if (pair.isPresent()) {
      return (String) pair.get().partB();
    }
    throw new IllegalArgumentException("unknown floor");
  }

  public int floorFromDivision(String division) {
    Optional<Pair> pair = Arrays.stream(divisions)
      .filter(p -> p.partB().equals(division))
      .findFirst();
    if (pair.isPresent()) {
      return (int) pair.get().partA();
    }
    throw new IllegalArgumentException("unknown floor");
  }
```

---

template: dry

```java
  public static class Pair<A, B> {
    private final A partA;
    private final B partB;

    public Pair(A partA, B partB) {
      this.partA = partA;
      this.partB = partB;
    }

    public A partA() {
      return partA;
    }
    public B partB() {
      return partB;
    }
  }
}
```

---

template: dry

### Qu'est-ce qu'on a fait ?

.see-also[
L'association division/étage est codée à un seul endroit : le tableau `divisions`.

Les méthodes `divisionFromFloor` et `floorFromDivision` sont devenues des "algorithmes"
exploitant le tableau pour fournir l'information demandée. Elles ne détiennent plus
de responsabilité.

Ajouter un étage, une division se fera à un seul endroit.
Le risque de coder une erreur d'association division/étage
est fortement réduit (le développeur a une vue d'ensemble de l'immeuble).
]

---

template: none

# Là, je répète

Une exception au DRY : les contrôles de surface vs les contrôles dans la couche métier.

Par exemple, une application WEB propose un formulaire en ligne.
Des contrôles de surface sont effectués : un montant supérieur à 0, une adresse mail correctement
formatée, etc.

Est-ce qu'il faut également répéter ces contrôles dans la couche métier (l'API qu'appellera le frontend) ?

Oui.

La couche métier est une "frontière" du système, un point d'entrée. Les "consommateurs" de la couche métier ne sont pas toujours "clean". La douane doit les contrôler. Une fois à l'intérieur, on peut être plus souple.

.see-also[
Les contrôles aussi dans la couche données ? Le moins possible. La couche données n'est pas censée être exposée directement. On "tolère" souvent des contraintes d'intégrité dans le SGBD mais on pourrait (devrait) s'en passer.
]

---

template: none

# Abstraction

Jeff Bolos se lance dans la vente en ligne de livres. Il automatise l'emballage.

```java
public class Book {

  private String title;
  // constructors, getters, setters omitted for brevity
  public Package pack() {
    return new Package(this);
  }
}
```

Ca marche bien, il vend aussi des lave-linges.

```java
public class Washer {

  private String brand;
  // constructors, getters, setters omitted for brevity
  public Package pack() {
    return new Package(this);
  }
}
```

---

# Abstraction : héritage

Jeff Bolos se souvient qu'il a hérité :

```java
public abstract class Item {

  public Package pack() {
    return new Package(this);
  }
}

// ...

public class Book extends Item {
}

public class Washer extends Item {
}
```

--

Oui mais,

- Si je dois vendre des objets fournis par des partenaires et qui sont déjà emballés ?
- Ce serait pas plus efficace de faire l'emballage par un service dédié ?

---

# Abstraction : les interfaces à la barre

```java
public class PackageService {

  public boolean addItem(Object item) {
    if (Measurable.class.isAssignableFrom(item.getClass())) {
      return addItem((Measurable)item);
    } else if (Packageable.class.isAssignableFrom(item.getClass())) {
      return addItem((Packageable)item);
    }
    return false;
  }
  public boolean addItem(Measurable item) {
    int weight = item.getWeight();
    if (weight < 2) {
      packageItem(item);
      return true;
    } else {
      return false;
    }
  }
  public boolean addItem(Packageable item) {
    packageItem(item);
    return true;
  }

  private void packageItem(Measurable item) {
  }

  private void packageItem(Packageable item) {
  }
}
```

---

# Abstraction : les interfaces à la barre

```java
public class Book extends Measurable {

}
public class Washer extends Measurable {

}
```

.see-also[
Les interfaces sont très largement utilisés dans le JDK : Serializable, Comparable, Runnable, ...

Pensez aussi aux génériques (pas les médicaments, les [types Java Generic](https://docs.oracle.com/javase/tutorial/java/generics/types.html)).
]

---

# Abstraction : le décorateur peut nous aider

Une classe qui ne supporte pas exactement ce dont nous avons besoin :

```java
public class MeasurableWeirdItem implements Measurable {

  private final WeirdItem item;

  @Override
  public int getWeight() {
    return item.getPieces() * 200; // each piece weighs 200g
  }
}
```

---

# Pensez global, codez local

- Découpez une méthode "longue" (pas de scroll) en plusieurs.
- Si une classe gère trop de concepts, divisez là.
- Préférez les variables locales aux champs de classe.

Mais surtout :

.focus-high[Une classe, une méthode, une variable a une seule responsabilité].
