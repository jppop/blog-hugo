---
title: "Quick Tips -- Logging"
date: "2019-04-28T11:30:00+02:00"
url: "/slides/logging/"
---

class: center, middle

# logger.info("Be a better logger");

## Bien utiliser la journalisation Java

---

# La journalisation

## A quoi ça sert ?

- Obtenir des informations sur l'exécution de l'application.
- Enregistrer les circonstances inhabituelles ou erreurs qui pourraient survenir.
- Auditer l'application

--

## Différents acteurs, différents besoins

Une application est développée, testée et exploitée. Par des acteurs différents.
Les journaux doivent prendre en compte les besoins de chacun.

Les différent niveaux, de DEBUG à ERROR, doivent cibler ces besoins différents.

La journalisation doit être configurable en fonction des circonstances.

---

# ERROR, WARN, INFO, DEBUG, TRACE ?

### ERROR

Quelque chose d'inattendue est arrivé, je ne peux plus continuer.

```java
logger.error("Database unavailable");
```

<img src="./arrow.png" alt="purpose" style="float: left; width: 20px"/>.focus[
Pour les opérateurs systèmes et les systèmes de supervision
]

### WARN

C'est pas normal mais prévu, on peut faire sans (le système continue à fonctionner).

```java
logger.warn("This resource is unavailable. Use the default value");
logger.warn("The mail service is mocked. No mails will be sent.");
logger.warn("Customer has placed no orders. Cannot process this order.");
logger.warn("Duplicate key. Ignoring the new one.");
```

<img src="./arrow.png" alt="purpose" style="float: left; width: 20px"/>.focus[
Pour les opérateurs systèmes, les systèmes de supervision et les administrateurs.
]

---

# ERROR, WARN, INFO, DEBUG, TRACE ?

### INFO

Un évènement, changement d'état s'est produit.

```java
logger.info("Starting server");
logger.info("Shutting down");
logger.info("Starting in dry run mode");
logger.info("Interrupt signal caught. Terminating the process");
```

<img src="./arrow.png" alt="purpose" style="float: left; width: 20px"/>.focus[
Pour les opérateurs systèmes et les systèmes de supervision, et les administrateurs. En général, permet de confirmer une action.
]

---

# ERROR, WARN, INFO, DEBUG, TRACE ?

### DEBUG

Détaille les changements internes et les traitements.
Les évènements de niveau DEBUG ne sont généralement pas reportés sauf pour des besoins de diagnostic.

```java
logger.debug("processing item {}", item);
logger.info("deleting file {}", filename);
logger.info("read {} items(s)", count);
```

<img src="./arrow.png" alt="purpose" style="float: left; width: 20px"/>.focus[
Pour les administrateurs et le support.
]

--

### TRACE

Idem que DEBUG mais plus verbeux.

<img src="./arrow.png" alt="purpose" style="float: left; width: 20px"/>.focus[
Pour les développeurs, les administrateurs et le support pour diagnostiquer un problème.
]

---

# SLF4J pour mettre tout le monde d'accord

Plusieurs bibliothèques de journalisation existent :

- Apache Log4j 2
- Logback
- Java Util Logging (JDK)

SLF4J, comme Apache Commons Logging (Jakarta Commons Logging, JCL), sont des facades au dessus des bibliothèque.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

.focus[
Très utile lorsque vous utilisez des frameworks ayant chacun opté pour un logger différent
(et n'utilisant pas SLF4J)
]

---

# Instances logger

Chaque classe émettant des logs, .focus[DOIT] déclarer sa propre instance de logger :

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

class Foo {
    private static final Logger LOGGER = LoggerFactory.getLogger(Foo.class);

    ...
}
```

---

# Message paramétré

```java
class Foo {
    private static final Logger LOG = LoggerFactory.getLogger(Foo.class);

    // GOOD: string literal, no dynamic objects
    public void good_method(Object arg) {
        LOG.debug("Method called with arg {}", arg);
    }

    // BAD: string varies with argument
    public bad_method1(Object arg) {
        LOG.debug("Method called with arg " + arg);
    }

    // BAD: code clutter
    public void bad_method2(Object arg) {
        if (LOG.isDebugEnabled()) {
            LOG.debug("Method called with arg {}", arg);
        }
    }
}
```

---

# Soyez sûr de ce que vous tracer

Attention à ne pas provoquer d'erreur :

```java
log.debug("Processing request with id: {}", request.getId());
```

`Request` n'est jamais nul ?

Un autre piège, les collections :

```java
log.debug("Returning users: {}", users);
```

On peut utiliser [Commons BeanUtils](http://commons.apache.org/beanutils) :

```java
log.debug("Returning user ids: {}", collect(users, "id"));
```

Avec :

```java
public static Collection collect(Collection collection, String propertyName) {
  return CollectionUtils.collect(collection,
    new BeanToPropertyValueTransformer(propertyName));
}
```

---

# Fournissez un contexte utile

```java
class Foo {
    private static final Logger LOG = LoggerFactory.getLogger(Foo.class);

    // VERY BAD:
    // - no context provided
    // - non-constant message string
    // - assumes useful toString()
    public bad_method1(Object arg) {
        LOG.debug(arg.toString());
    }

    // VERY BAD:
    // - no context provided
    public bad_method2(Object arg) {
        LOG.debug("{}", arg);
    }

    // COMPLETELY BAD:
    // - silently ignoring errors!!!
    public bad_method3(Object arg) {
        try {
            doSomething(arg);
            ...
        } catch (SomeException ex) {
        }
    }
}
```

---

# Fournissez un contexte utile

```java
class Foo {
    private static final Logger LOG = LoggerFactory.getLogger(Foo.class);

    // EXTREMELY BAD:
    // - message is not constant
    // - no context is provided
    // - ex.getCause() is lost
    // - call stack is lost
    public void bad_method4(Object arg) {
        try {
            doSomething(arg);
            ...
        } catch (SomeException ex) {
            LOG.warn(ex.getMessage());
        }
    }

    // EXTREMELY BAD:
    // - message is not constant
    // - no context is provided
    // - ex.getCause() is probably lost
    // - call stack is probably lost
    // - assumes useful toString()
    public void bad_method5(Object arg) {
        try {
            doSomething(arg);
            ...
        } catch (SomeException ex) {
            LOG.warn(ex.toString());
        }
    }
}
```

# Fournissez un contexte utile

```java
class Foo {
    private static final Logger LOG = LoggerFactory.getLogger(Foo.class);

    // VERY BAD:
    // - no useful context is provided
    // - ex.getCause() is probably lost
    // - call stack is probably lost
    // - administrators don't know what an Exception is!
    public void bad_method6(Object arg) {
        try {
            doSomething(arg);
            ...
        } catch (SomeException ex) {
            LOG.warn("Exception {}", ex);
        }
    }
}
```

---

# Tracer les exceptions

```java
class Foo {
    private static final Logger LOG = LoggerFactory.getLogger(Foo.class);

    // GOOD:
    // - string literal
    // - we explain what we tried to do
    // - we pass along information we have about the failure
    // - we escalate the failure to our caller
    // - we also 'chain' the exception so it is not lost and can be
    // correlated
    public void good_method1(Object arg) {
        try {
            doSomething(arg);
            ...
        } catch (SomeException e) {
            LOG.debug("Failed to do something with {}, got error {}", arg, e);
            String msg = String.Format("Failed to do something with {}", arg);
            throw new RuntimeException(msg, e);
        }
    }
}
```

---

# Tracer les exceptions ignorées

```java
class Foo {
    private static final Logger LOG = LoggerFactory.getLogger(Foo.class);

    // GOOD:
    // - string literal
    // - we explain what we tried to do
    // - we pass along information we have about the failure
    // - we explain that we recovered from the failure
    public void good_method1(Object arg) {
        try {
            doSomething(arg);
            ...
        } catch (SomeException ignore) {
            LOG.warn("Failed to do something with {}, ignoring it", arg, ignore);
        }
    }
    // BAD:
    // - exception is interpreted as an object
    // - exception chaining cause is lost
    // - stack trace is lost
    public void bad_method(Object arg) {
        try {
            doSomething(arg);
            ...
        } catch (SomeException ex) {
            LOG.warn("Failed to do something with {} because {}, continuing",
                      arg, ex);
        }
    }
}
```

---

# is{Trace|Debug|Info|Warn|Error}Enabled ?

Ne pas l'utiliser. Sauf :

```java
for (int i = 0; i < 100000; i++) {
    if (LOG.isDebugEnabled()) {
        LOG.debug("The size is: {}", expensiveMethodToCalculateSize());
    }
}
```

Passer une référence, jamais toString()

Par exemple, ceci est mal :

```java
List<Interface> interfaces;
if (LOG.isDebugEnabled()) {
    LOG.info("Interfaces: {}", interfaces.toString());
}
```

Plus simplement :

```java
LOG.debug("Interfaces: {}", interfaces); // no need to guard this with isDebugEnabled!
```

---

# Logback -- configuration

Logback trouve son fichier de configuration (XML ou groovy) dans le `classpath` :

1. cherche le fichier `logback-test.xml`.
2. cherche le fichier `logback.groovy`, puis `logback.xml`

Vous pouvez également fournir explicitement le chemin du fichier de configuration :

```
java -Dlogback.configurationFile=/path/to/config.xml ...
```

Pour les projets Maven, placez `logback-test.xml` dans `src/test/resources`.
Et `logback.xml`, dans `src/main/resources`.

.focus-high[
La configuration _DOIT_ être modifiable en production sans nécessiter une relivraison.
]

--

.see-also[
Si l'application peut être démarrée en fournissant l'option `logback.configurationFile`,
alors le fichier `logback.xml` peut être empaqueté dans l'exécutable.

Et l'installation de l'application permettra de prendre en compte un fichier de configuration
externe.
]

---

# Logback -- configuration : les variables.

La **Production** accueille votre application chez elle, respectez leurs règles.

.focus-high[
La configuration _DOIT_ s'adapter au besoin d'exploitation.
]

Utilisez des variables (pour éviter de maintenir un fichier par environnement).

```xml
<configuration>

  <property name="LOGDIR" value="${logging.dir:-./logs}" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>${LOGDIR}/myApp.log</file>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <root level="${logging.level:-ERROR}">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

.focus[
<img src="./arrow.png" alt="RTFM" style="float: left; width: 20px"/>Plus d'information : [Chapter 3: Logback configuration](https://logback.qos.ch/manual/configuration.html)
]

---

# Logback -- Appenders

Logback délègue l'écriture des logs aux [**Appenders**](https://logback.qos.ch/manual/appenders.html).

```xml
  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
      <charset>UTF-8</charset>
      <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>%d{ISO8601}|%-5level|%-20thread|%X{uuid}|%logger{36}|%msg%n%xEx</pattern>
      </layout>
    </encoder>
  </appender>

  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <File>${LOG_FILE}</File>
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
      <!-- daily and size rollover -->
      <fileNamePattern>${LOGDIR}/${LOG_FILE}.%d{yyyy-MM-dd}.%i.zip</fileNamePattern>
      <maxHistory>31</maxHistory>
      <maxFileSize>500MB</maxFileSize>
      <totalSizeCap>20GB</totalSizeCap>
    </rollingPolicy>

    <append>true</append>
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
      <charset>UTF-8</charset>
      <pattern>%d{ISO8601}|%-5level|%-20thread|%X{uuid}|%logger{36}|%msg%n%xEx</pattern>
    </encoder>
  </appender>
```

---

# Logback -- Loggers

```xml
<logger name="my.package" level="${logging.level:-info}" />

<logger name="my.package.audit" additivity="false">
  <level value="trace" />
  <appender-ref ref="AUDIT" />
</logger>

<!-- Show SQL statements (set logging.sql.level to debug to enable them) -->
<logger name="org.hibernate.SQL" level="${logging.sql.level:-warn}" />

<root level="INFO">
  <appender-ref ref="FILE"/>
  <appender-ref ref="CONSOLE"/>
</root>
```

---

# Pour la route

Quelques règles :

- .focus-high[Pas de log sur la console]  
  En production, ni en dév. (ne vous mentez pas, vous ne les regardez jamais).

- .focus-high[Pas de chemin en dur]  
  Utilisez des variables. En test, appuyez vous sur Maven pour substituer les chemins.

  ```xml
  <property name="logDir" value="${project.build.directory}/logs}"/>
  ```

- .focus-high[Gardez toujours en tête à quoi servent les logs]  
  LA principale utilisation reste l'analyse de problème en production.

- .focus-high[Aidez vous des autres pour améliorer la qualité des logs]  
  Les équipes de support, de production peuvent vous indiquer quelles informations pourraient
  faciliter leurs tâches (exploitation, analyse). Profitez d'une correction, d'une évolution pour
  améliorer la journalisation.

Bonus : on vous demandera surement un jour [un truc comme ça](https://moelholm.com/2016/08/16/spring-boot-enhance-your-logging/).

.note[
Crédits : [OpenDayLight, Logging Best Practices](https://wiki.opendaylight.org/view/BestPractices/Logging_Best_Practices)
]
