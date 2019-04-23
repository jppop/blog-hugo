---
title: "Quick Tips -- Spring Batch Unit Testing"
date: "2019-04-13T16:43:02-04:00"
url: "/slides/spring-batch-test/"
---

class: center, middle

# Spring Batch Unit Testing

## Tester les applications basées sur Spring Batch

---

# Le batch

Basé sur le guide [Creating a Batch Service](https://spring.io/guides/gs/batch-processing/).

```java
    @Bean
    public Step step1(JdbcBatchItemWriter<Person> writer) {
        return stepBuilderFactory.get("step1")
            .<Person, Person> chunk(10)
            .reader(reader())
            .processor(processor())
            .writer(writer)
            .build();
    }
```

Et quelques "vrais" exemples.

---

# Nos outils

La batch est testé avec :

- JUnit, Mockito, AssertJ
- Le _JUnit runner_ .focus[SpringJUnit4ClassRunner]
- .focus-high[spring-batch-test]

.note[
La documentation en ligne :
[Spring Batch Unit Testing](https://docs.spring.io/spring-batch/4.0.x/reference/html/testing.html)
]

---

# Test unitaire de la phase _read_

```java
  private List<Person> readAll(File inputFile) throws Exception {

    // build step execution context
    StepExecution stepExecution = MetaDataInstanceFactory.createStepExecution();
    stepExecution.getExecutionContext().putString("input.file", inputFile.getAbsolutePath());

    return StepScopeTestUtils.doInStepScope(
      stepExecution,
      () -> {

        // init reader
        reader.open(stepExecution.getExecutionContext());

        List<Person> items = new ArrayList<>();

        Person person;
        while ((person = reader.read()) != null) {
          items.add(person);
        }
        return items;
      });
  }
```

---

# Pas à pas

### Créez la classe de test

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = {BatchConfiguration.class, TestConfig.class})
public class ReaderTest {
}
```

`BatchConfiguration` définit le job ; `TestConfig` définit des ressources pour Spring Batch.

--

### Préparez le test (_Arrange_)

```java
    // Arrange
    File dataFile = folder.newFile("data.csv");

    CsvFaker<Person> csvFaker = new CsvFaker<>(Person.class);
    csvFaker
      .with("001", personOf("john", "doe", 23))
      .build(dataFile);

```

---

# Pas à pas

### Exécuter la méthode `read` (_Act_)

```java
    // Act
    List<Person> people = readAll(dataFile);
```

### Vérifiez (_Assert_)

```java
    // Assert
    assertThat(people).hasSize(1);
    assertThat(people)
      .extracting("firstName", "lastName", "age")
      .contains(tuple("john", "doe", 23));
```

--
Assert that you can read more on [AssertJ](https://assertj.github.io/doc/)

---

# Les cas d'erreur aussi

```java
  @Test
  public void failWhenInputIsNotValid() throws Exception {

    // Arrange
    File dataFile = folder.newFile("data.csv");

    CsvFaker<Person> csvFaker = new CsvFaker<>(Person.class);
    csvFaker
      .with("001", personOf("john", "doe", 23))
      .withFaultyRecord("002", "donald", "duck", "xx")
      .build(dataFile);

    // Act and Assert
    assertThatThrownBy(() -> readAll(dataFile))
      .isInstanceOf(FlatFileParseException.class)
      .hasCauseInstanceOf(BindException.class);
  }
```

--
.note[Un focus sur CsvFaker et CsvNameExtractor maintenant, non ?]

---

# Process, sans les mains

```java
  @Test
  public void process() throws Exception {

    // Arrange

    // create John
    Person john = personOf("john", "doe", 34);
    // John has 123 as ID
    Mockito.when(
      nationalService.findNationalIdentifier(
        eq("John"),
        eq(john.getLastName().toUpperCase())))
      .thenReturn(Optional.of("123"));

    // Act
    Person actualPerson = processor.process(john);

    // Assert

    assertThat(actualPerson.getFirstName()).isEqualTo("John");
    assertThat(actualPerson.getLastName()).isEqualTo("DOE");
    assertThat(actualPerson.getAge()).isEqualTo(34);

    assertThat(actualPerson.getNationalId()).isEqualTo("123");
  }
```

---

# End-To-End test

Les tests de bout en bout testent l'exécution complète du job.

```java
JobExecution jobExecution = jobLauncherTestUtils.launchJob(
    new JobParametersBuilder()
        .addString(
          CollectGIPMDSJobConfiguration.FILENAME_PARAMETER,
          inputFile.getAbsolutePath())
        .toJobParameters());
```

La méthode retourne un objet `JobExecution` permettant de vérifier l'exécution du job.

```java
    assertEquals(BatchStatus.COMPLETED, jobExecution.getStatus());

    // assert read/written items
    final int expectedGlobalReads = fileCount * itemCount;
    Optional<StepExecution> executionOpt =
      stepExecutions.stream().filter(e -> "partitionStep".equals(e.getStepName())).findFirst();
    assertTrue(executionOpt.isPresent());
    StepExecution execution = executionOpt.get();
    assertEquals(expectedGlobalReads, execution.getReadCount());
```

---

# Derniers conseils pour la route

- Utilisez une base de données en mémoire (H2 ou HSQL) pour tester les `JdbcCursorItemReader`
  ou `JdbcBatchItemWriter`.

- Appuyez vous sur Spring Configuration pour remplacer certaines implémentations par des mocks.
  -> Un exemple, `CriticalErrorTest`.

- Gardez vos tests .focus[simples].  
  Compréhensibles en un coup d'oeil. N'hésitez pas à cacher la "plomberie".

Les sources de l'exemple sont sur [GitHub](https://github.com/jppop/spring-batch).
