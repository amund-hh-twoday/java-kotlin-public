# java-kotlin-workshop

## Oppgave: Bygg en Todo-app med Spring Boot

### Oppgave 1 - Basic web-app

#### 1.1

Last ned Spring Boot ved hjelp av Spring Initializr.</br>
Legg til avhengighetene Spring Web og Spring JDBC. Åpne prosjektet i Intellij, kjør Maven "clean install" og</br>
start deretter applikasjonen. Når du ser dette har du klart oppgaven: <img src="/src/main/resources/img/1.png">

#### 1.2

Opprett en ny mappe med navn controller. Lag en Controller-klasse med et POST-endepunkt som tar imot en streng i
request</br>
parameter, og returnerer den samme strengen i en 200 OK respons. Dette skal være todo-innslaget.</br>
Test at det fungerer med Postman.

#### 1.3

Lag en "service"-mappe og putt inn en ny Service-klasse (Tips: @Service). Den skal ha en add()-metode som tar inn
en</br>
streng. Bruk autowiring til å hente inn Servicen i Controlleren og kall add()-metoden derfra.</br>
Passer inn strengen og lagre den i en lokal variabel i Service-klassen.

#### 1.4

Lag et nytt GET-endepunkt i controlleren. Den skal kalle en get()-metode i Servicen som returnerer den lagrede
strengen,</br>
og returnere denne til klienten.

### Oppgave 2 - Dataklasser

#### 2.1

Lag en mappe som heter "model" og opprett en dataklasse med navn TodoEntry. Den skal inneholde feltene id (UUID),</br>
date (LocalDate) og todo (String), som alle er påkrevd.

#### 2.2

Endre add()-metoden i Servicen til å bygge en TodoEntry og returnere iden til objektet.</br>
Tips: Bruk UUID.randomUUID() for å generere id.

#### 2.3

Test at add()-metoden i Servicen returnerer id ved å koble den på controlleren og kalle POST-endepunktet med Postman

### Oppgave 3 - Lagre i minne

#### 3.1

Lag en "repository"-mappe og plasser en repository-klasse der (bruk @Repository). Den skal ha en funksjon som tar
imot</br>
en TodoEntry og lagrer den i et map.</br>
Tips: Bruk mutableMapOf<TodoEntry>() for å lage mappet

#### 3.2

Bruk autowiring for å ta repository-klassen inn i Service-klassen, og bruk den nye metoden til å lagre Todo-en der</br>
isteden.

#### 3.3

Legg til en metode i repository-klassen som henter ut igjen det lagrede innslaget.</br>
Ta i bruk funksjonen til å hente ut todo-en ved å kalle GET-endepunktet</br>
Test med postman at du får sendt inn og hentet ut et todo-innslag.

### Oppgave 4 - Utvidet funksjonalitet

#### 4.1

Legg til funksjonalitet for å kunne markere en Todo som fullført

#### 4.2

Legg til funksjonalitet for å hente ut alle Todoer i en liste

#### 4.3

Legg til funksjonalitet for å slette Todo-innslag

#### 4.4

Test at det virker ved hjelp av Postman

### Oppgave 5 - Exceptionhåndtering

Vi ønsker å kunne sende ulik respons tilbake til konsumenten avhengig av resultatet på operasjonen. </br>
Dette kan vi gjøre ved å returnere ulike former for ResponseEntity objekter. </br>
F.eks bygger vi en 200 OK respons fra controller ved å skrive `return ResponseEntity.ok().build()`

Den enkleste måten å gjøre dette på er å lage logikk i Controller-klassen og returnere passende respons der.</br>
Men vi kan også lage dedikerte ExceptionHandler-klasser for å håndtere feil på en ryddigere måte, på et samlet sted.

#### 5.1

Dersom vi prøver å hente ut en Todo som ikke eksisterer, skal konsumenten få 404 tilbake.</br>
Legg til logikk for å få dette til å skje på GET-endepunktet, og test med Postman.

#### 5.2

Lag en mappe "exception" og opprett en klasse med navn ExceptionHandler. </br>
Marker klassen med @ControllerAdvice

#### 5.3

Lag en egen Exception-klasse som utvider RuntimeException. Kall den for TodoNotFoundException

#### 5.4

Lag en handler-metode i ExceptionHandler som plukker opp denne nye exceptionen og returnerer 404 Not Found

#### 5.5

Tilpass logikken fra 5.1 til å bruke denne handleren istedenfor å håndtere det direkte i controller-klassen

#### 5.6

Legg til logikk på resten av endepunktene for å bruke denne handleren, f.eks dersom en ikke finner en todo ved
sletting.</br>
Tips: Kast exception i Service-klassen.

### Oppgave 6 - Automatiserte tester

Det mest brukte testautomatiseringsverktøyet for Java er JUnit. </br>
Vi skal her bruke en kombinasjon av JUnit og MockK for å skrive automatiserte tester til programmet vårt.

#### 6.1

Importer MockK som avhengighet i prosjektet:

```    
<dependency>
<groupId>io.mockk</groupId>
<artifactId>mockk-jvm</artifactId>
<version>1.13.12</version>
<scope>test</scope>
</dependency>
```

#### 6.2

Under test-mappen: opprett en ny klasse med navn TodoServiceTest.</br>
Marker klassen med @ExtendWith(MockKExtension::class)</br>
Bruk MockK til å lage tester som sjekker at repositoryklassen blir kalt med riktige parametere og gir riktig svar
tilbake.</br>

Kjør testene og se at alt går grønt.

Tips: Slik ville jeg gjort det for add()-metoden:</br>

```
    @Test
    fun `legg til ny todo`() {
        val service = TodoService(todoRepository)
        every { todoRepository.insertTodo(any()) } just runs
        service.add("todo")
        verify(exactly = 1) { todoRepository.insertTodo(any()) }
    }
```

### Oppgave 7 - Koble på database

I denne oppgaven skal vi koble på en in-memory H2 SQL database, og lagre Todo-innslagene våre dit isteden.</br>
Her bruker vi Spring-data sin innebygde namedParameterJdbcTemplate for å gjøre SQL-kall mot databasen.</br>
Vi bruker Flyway for å sette opp databasestrukturen og håndtere versjonsmigrering.

#### 7.1

Vi begynner med å konfigurere opp H2-databasen. For å bruke H2 trenger du denne avhengigheten i prosjektet:

```
    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <version>2.3.230</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>org.flywaydb</groupId>
      <artifactId>flyway-core</artifactId>
      <version>10.17.0</version>
    </dependency>
```

Du må også legge til noe ekstern Spring-konfigurasjon.