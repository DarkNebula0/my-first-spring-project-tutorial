# my-first-spring-project
Erstellt zusammen mit Umschülern in der Mischok Academy, Modul Software-Architektur 1. Hierunter eine vorab pro Session vorbereitete Anleitung, die wir über mehrere Sitzungen befolgt haben.

# My first Spring Backend (+Thymeleaf Frontend)

Das hier ist eine Schritt-für-Schritt-Anleitung für die Erstellung eines ersten [Spring](https://spring.io/) Backends. Entstanden im Rahmen vom Modul "Software-Architektur 1" in der Mischok Academy.

## 0) Was wir bauen
Eine funktionierende, kleine ***Spring***-Anwendung, mit folgenden Funktionalitäten
- Dependencies verwalten mit ***Maven*** in einer `pom.xml`
- **GET**- und **POST-Requests** annehmen
  - Anfrage
    - GET zuerst nur mit der URL-Zeile vom Browser
    - für POST dann mit `curl` aus dem Terminal
    - später alles über klickbare Links und **Formulare** im HTML-Frontend
  - Antwort
    - mit ***Thymeleaf*** direkt eine HTML-Seite als renderbares Frontend
    - oder einfach Daten als **JSON**
    - später auch mit selbst gesetztem *HTTP-Status-Code* und weiteren Informationen im *HTTP-Header*
- mit Code-Generator ***Lombok*** sparen wir uns Boilerplate-Code und lernen *Annotations* kennen, die nicht zu Spring gehören
- ganz normale HTML-Seiten anzeigen
- Datenbank zum Speichern von Daten
  - wir nutzen ***H2***, zuerst *In-Memory*, dann auch persistiert lokal als Datei
  - dank Spring ***JPA*** sparen wir uns eigene SQL-Queries
  - *Datenbank-Migrations* mit ***Flyway***, um initiale Queries und andere Datenbankänderungen reproduzierbar und automatisiert durchzuführen
- *Automatisierte Tests* (Coming Soon, im Modul Software-Architektur 2)

## 1) Voraussetzungen
- Vorwissen:
  - **Java** Grundlagen
  - **Terminal** in einem Ordner öffnen und keine Angst davor haben
  - optional: **git** installiert und Grundlagen dafür gelernt, um nach jedem Schritt (oder nach wenigen Schritten) einen schönen neuen Commit anzulegen
- Betriebssystem: **Ubuntu** wäre gut
- im Terminal installierte/verfügbare Programme:
  - **mvn**
  - **curl**
- IntelliJ oder anderer Editor / IDE
  - mit Java **17** oder höher


## 2) Start und ein erster Endpunkt

### Leeres Spring Projekt mit passenden Dependencies starten

Ausgangspunkt ist ein ausführliches [Spring Tutorial zu "*Serving Web Content with Spring MVC*"](https://spring.io/guides/gs/serving-web-content), falls man es selbst mal durchlesen möchte (lückenhaft und teilweise nicht so gut). In unserer Anleitung gehen wir darüber hinaus und nutzen u.a. Annotations von **Lombok**, bekommen auch **JSON** als Antwort von unseren Endpunkten, verschicken **POST-Requests** und binden eine **H2-Datenbank** ein.

Wir starten mit der Grundstruktur, die wir aus dem [Spring initializr](https://start.spring.io/) herausbekommen.
- entweder die Vorauswahl in [diesem pre-initialized project](https://start.spring.io/#!type=maven-project&language=java&packaging=jar&jvmVersion=11&groupId=com.example&artifactId=serving-web-content&name=serving-web-content&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.serving-web-content&dependencies=web,thymeleaf,devtools) herunterladen
- oder selbst im [Spring initializr](https://start.spring.io/) mit Auswahl:
  - Maven als Buildtool
  - Textfelder passend ausfüllen
  - Dependencies auswählen:
    - Spring Web
    - Thymeleaf
    - Spring Boot DevTools
- diese Dependencies liegen dann direkt in unserer Maven-Verwaltungs-Datei `pom.xml`:
  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
  ```
- dieses Startprojekt am besten direkt auch mit Hilfe von **git** versionieren, optional auch remote auf GitLab oder GitHub, und im Laufe der Anleitung passende Commits erstellen.

### in IntelliJ öffnen und einfachen GET-Endpunkt erstellen
- in IntelliJ öffnen, am besten direkt die `pom.xml` auswählen
- erstelle neben der main-Klasse auch einen GreetingController, dort zuerst nicht Code aus Tutorial einfügen
	- zuerst komplett ohne RequestParam
      ```java
      package com.example.simple;

      import org.springframework.stereotype.Controller;
      import org.springframework.ui.Model;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RequestMapping;

      @Controller
      public class GreetingController {

          @GetMapping("/greeting")
          public String greeting(Model model) {
              return "greeting";
          }
      }
      ```
- erstelle `greeting.html` in `src/main/resources/templates`
	- Code aus [Spring-Anleitung](https://spring.io/guides/gs/serving-web-content#initial) kopieren, aber hier noch `${name}` rausnehmen (evtl Zeile ersetzen und auskommentieren).
    - ACHTUNG!: die einzelnen Anführungszeichen um `Hello World!` im folgenden Beispielcode sind wichtig! Durch die doppelten Anführungszeichen wird erst der Input für Thymeleaf ermöglicht, aber dann muss man die einzelnen Anführungszeichen noch als String-Delimiter setzen. Das nicht zu tun, wäre so verwerflich, wie in Java `String text = Hello World!;` ohne String-Delimiter zu schreiben.
    ```html
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Hallo :-)</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    </head>
    <body>
      <p th:text="'Hello World!'"></p>
    </body>
    </html>
    ```
    - beachte beim Code-Beispiel von der [Spring-Anleitung](https://spring.io/guides/gs/serving-web-content#initial) die Pipe-Zeichen "`|`", durch die man in Spring Strings markiert, in denen man Platzhalter ähnlich wie bei Javascript Strings einsetzen kann. Für diese Anleitung lieber Strings konkatenieren wie in basic Java

### Backend lokal laufen lassen mit maven
Zum Ausführen des Projekts lernen wir zwei Möglichkeiten:
- in einer IDE: bei IntelliJ auf Play drücken, oder vorher Rechtsklick auf die `pom.xml` und dort "Ausführen" (mit grünem Play-Symbol daneben) oder sowas anklicken, oder auf die Java-Klasse mit der main Rechtsklick und "Ausführen". Nach dem ersten erfolgreichen Run sollte bei IntelliJ oben rechts ein grüner Play-Button verfügbar sein.
- ohne IntelliJ oder sonstige IDE: Terminal öffnen im Projektordner und ausführen:
  ```
  ./mvnw spring-boot:run
  ```

Bis wir die Ausführung im Terminal wieder stoppen, können wir jetzt unseren Endpunkt erreichen, zB in einem Webbrowser wie Firefox.
- aufrufen: http://localhost:8080/greeting
  - `localhost` ist der Name des virtuellen Servers, der auf dem eigenen Computer läuft. Der gleiche Ort ist erreichbar über die IP-Adresse 127.0.0.1 und wird oft verwendet von lokal laufenden Anwendungen, die eine Webseite/-server für die Entwicklung lokal simulieren wollen. Die Zahl dahinter steht für den verwendeten Port.

Jetzt ändern wir den Inhalt im Thymeleaf-Template durch einen Request-Parameter am Ende der URL. Dafür müssen wir unseren Controller anpassen:
- mit `@RequestParam` und Variable eingefügt in `model`, dafür auch das Thymeleaf Template (`greeting.html`) anpassen
  ```java
  @GetMapping("/greeting")
  public String greeting(@RequestParam(name = "name", required = false, defaultValue = "World") String someName, Model model) {
      model.addAttribute("name", someName);
      return "greeting";
  }
  ```
  ```html
  <p th:text="'Hello ' + ${name} + '!'"></p>
  ```
  ```
  localhost:8080/greeting
  localhost:8080/greeting?name=Alex
  ```
  - die Variable beim `@RequestParam` auch mal "`someName`" oder so nennen, um abzugrenzen vom RequestParam `name="name"` <- letzteren auch mal nur `"n"` nennen oder `"u"`. Alternativer Endpunkt und `<p>`-Tag in Thymeleaf und was man im Browser aufrufen muss:
    ```java
    @GetMapping
    public String greeting(@RequestParam(name = "n", required = false, defaultValue = "World") String someName, Model model) {
        model.addAttribute("inputName", someName);
        return "greeting";
    }
    ```
    ```html
    <p th:text="'Hello ' + ${inputName} + '!'"></p>
    ```
- beachte, dass man die ganzen Bezeichnungen hierüber (`someName`, `n`, `inputName`) auch alle gleich benennen könnte, zB "name".
  Die sind hier nur verschieden, um klar zu machen, welcher Name was tut.

## 3) JSON als Antwort, Lombok und statische Seiten

### JSON als Antwort vom Endpunkt
Als nächstes wollen wir vom Endpunkt aus json zurückgeben.
Dafür nicht mehr mit Thymeleaf und mit dem model und String-Rückgabe beim Endpunkt.
Sondern wir geben einfach ein POJO (=*Plain Old Java Object*) zurück und Spring-Web konvertiert das mit Jackson für uns direkt zu einem JSON-Objekt.

<!-- - bevor wir mehr Endpunkte aufrufen, verschieben wir das `"/greeting"` von `@GetMapping` (und löschen da die dann leere Klammer) und setzen stattdessen für den ganzen Controller oben direkt unter `@Controller` diese Zeile:`@RequestMapping("/greeting")` -->
- jetzt brauchen wir ein POJO, das wir dann zurückgeben können.
  Für diese Anleitung Arbeiten wir beispielhaft mit einer Klasse `Person`:
  ```java
  package com.example.simple;

  public class Person {
    private int id;
    private String name;

    public int getId() {
        return this.id;
    }
    public String getName() {
        return this.name;
    }

    public void setId(int id) {
        this.id = id;
    }
    public void setName(String name) {
        this.name = name;
    }
  }
  ```

- GET Endpunkt mit JSON-Antwort schreiben, hier brauchen wir auch noch die Annotation `@ResponseBody`:
  ```java
  @GetMapping("/greetingJson")
  @ResponseBody
  public Person greetingJson(@RequestParam(name = "name", defaultValue = "Max") String someName) {
      Person person = new Person();
      person.setId(1);
      person.setName(someName);

      return person;
  }
  ```
  - Achtung: hier auch `model` bei den Eingabeparametern weglassen!


### Lombok Annotations
Wir löschen in unserem POJO die Getter und Setter und nutzen stattdessen die passenden Annotations von Lombok.
Mehr Infos [bei Baeldung](https://www.baeldung.com/intro-to-project-lombok).
- So aktivieren wir Lombok in IntelliJ:
  - Dependency in die `pom.xml` packen.
    Am besten recherchieren wir dafür alle selbst, wie wir die Dependency kriegen.
    Suchmaschine: "maven lombok"
    ```xml
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.34</version>
    </dependency>
    ```
  - jetzt würde das schon compilen, aber damit IntelliJ das für die Unterstützung beim Code Schreiben auch versteht, müssen wir das Plugin Lombok noch installieren.
  Dann eventuell noch bei IntelliJ bei kleinem aufploppendem Dialog auf den Button mit "Enable Lombok Annotation Processing" klicken
- jetzt löschen wir bei unserem POJO die Getter und Setter und setzen stattdessen lombok-Annotations:
  ```java
  package com.example.simple;

  import lombok.Getter;
  import lombok.Setter;

  @Getter
  @Setter
  public class Person {
      private int id;
      private String name;
  }
  ```
- optional: `@Builder`-Annotation einsetzen, um unser POJO im Stile des dem Builder-Patterns zu erzeugen, statt mit Konstruktoren und hinterher gesetzten Werten zu arbeiten.
  - Vorsicht bei Lombok: Die `@Builder`-Annotation macht den leeren Konstrktur kaputt macht bei der POJO, d.h. wenn man den braucht, braucht man noch die Lombok-Annotation `@NoArgsConstructor`. Dann aber geht der Builder wieder kaputt, außer man setzt dann noch die Annotation `@AllArgsConstructor`.
  ```java
  import lombok.Builder;
  import lombok.Getter;
  import lombok.Setter;

  @Getter
  @Setter
  @Builder
  public class Person {
      private Long id;
      private String name;
  }
  ```
  ```java
  @GetMapping("/person")
  @ResponseBody
  public Person personJson() {
      // Objekt erstellen, zB aus Datenbank holen, sonstige Business-Logik
      Person person = Person.builder()
              .id(7L)
              .name("Paul")
              .build();

      return person;
  }
  ```

### Welcome Page
Damit wir uns nicht die Pfade merken müssen, basteln wir uns eine statische, stinknormale HTML-Startseite, ganz ohne Thymeleaf.
Spring (bzw. Spring Boot?) erkennt alle in `src/main/resources/static` abgelegten HTML-Dateien und antwortet auf GET-Requests mit einem Pfad, der sich mit dem Pfad von abgelegten Dateien auf `static/` deckt, mit der entsprechenden HTML-Datei.

In folgender Hauptseite (`static/index.html`) zeigen wir unsere bisherigen Links klickbar an.
Dafür legen wir eine Datei `index.html` in den Ordner `src/main/resources/static`:
  ```html
  <!DOCTYPE HTML>
  <html>
  <head>
    <title>Meine Seite</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>
  <body>
    <p>
      Gerade angekommen? <a href="/greeting">Hier entlang.</a>
    </p>
    <p>
      Bist du Alex? Dann <a href="/greeting?n=Alex">bitte hier</a>.
    </p>
  </body>
  </html>
  ```
  - wie wir die Unterseite mit Input öffnen können, ohne ihn selbst in die URL zu schreiben, sehen wir in Kürze. TODO ALEX check ob GET mit Form mit Request Params geht!!
    Zunächst reichen uns diese beiden fest gesetzten Links -->


## Thymeleaf mit Styling

Wie auch in HTML-Dateien, können wir oben im `<head>`-Tag Thymeleaf-Template eine CSS-Datei mithilfe des HTML-Tags `<link>` einbinden.
Damit das Backend, auch wenn es gebaut ist und auf irgendeinem Server läuft, noch den Pfad zur Datei findet, nutzen wir Thymeleafs URL-Syntax.
Dadurch können wir den Pfad ab dem Ordner `src/main/resources/static/` angeben.
Dafür schreiben wir im `<link>`-Tag: `th:href="@{/...}"`.

Mehr zu [Thymeleafs URL-Syntax in der Dokumentation](https://www.thymeleaf.org/doc/articles/standardurlsyntax.html).

Als Beispiel stylen wir unser Template für die Begrüßung.
- erstellen unter `src/main/resources/static/greeting-styles/boring.css` eine CSS-Datei:
  ```css
  p {
    font-family: monospace;
    font-weight: bold;
    font-size: 5rem;
    font-style: italic;
  }
  ```
- in der `greeting.html` fügen innterhalb vom `<head>`-Tag den folgenden Tag hinzu:
  ```html
  <link rel="stylesheet" th:href="@{/greeting-styles/boring.css}">
  ```

### CSS je nach RequestParam
Jetzt wollen wir eine weitere CSS-Datei erstellen und diese nutzen, wenn ein passender RequestParam beim GET-Request mitgegeben wurde.
- Endpunkt anpassen:
  ```java
  @GetMapping("/greeting")
  public String greeting(
          @RequestParam(name = "n", required = false, defaultValue = "World") String someName,
          @RequestParam(name = "css", required = false, defaultValue = "") String cssFileName,
          Model model
  ) {
      model.addAttribute("inputName", someName);
      model.addAttribute("cssFileName", cssFileName);
      return "greeting";
  }
  ```
- diese für Thymeleaf bereitgelegte Variable `cssFileName` setzen wir ein im Template.
  Als Default verwenden wir aber weiterhin unsere alte CSS-Datei, falls der RequestParam leer geblieben ist.
  Dafür basteln wir uns den String für den fertigen Pfad zusammen, u.a. mit der aus Java bekannten String-Methode `.isEmpty()`:
  ```html
  <!DOCTYPE HTML>
  <html xmlns:th="http://www.thymeleaf.org">
  <head>
      <title>Hallo :-)</title>
      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  <!--    <link rel="stylesheet" th:href="@{/greeting-styles/boring.css}">-->
      <link
        rel="stylesheet"
        th:href="@{'/greeting-styles/' + ${cssFileName.isEmpty() ? 'boring' : cssFileName} + '.css'}"
      >
  </head>
  <body>
      <p th:text="'Hello ' + ${inputName} + '!'"></p>
  </body>
  </html>
  ```
- neue CSS-Datei könnte unter `src/main/resources/static/greeting-styles/style-alex.css` erstellt werden und so aussehen:
  ```css
  p {
    color: darkcyan;
    background-color: white;
    padding: 0.5rem 4rem;

    transition: background-color 0.5s, scale 0.5s, rotate 1s;

    font-weight: bold;
    font-family: Arial;
    font-size: 5rem;
  }
  p:hover {
    background-color: black;
    scale: -1.2 1.2;
    rotate: 4deg;
  }

  body {
    background-color: darkcyan;

    display: flex;
    justify-content: center;
    align-items: center;

    height: 100vh;
    width: 100vw;
  }
  ```
- in unserer Hauptseite unter `src/main/resources/static/index.html` passen wir den zweiten `<p>`-Tag an, sodass er die Begrüßung mit dem richtigen Style aufruft:
  ```html
  <p>
      Bist du Alex? Dann <a href="/greeting?n=Alex&css=style-alex">bitte hier</a>.
  </p>
  ```

Die alte, jetzt auskommentierte Zeile `<link rel="stylesheet" th:href="@{/greeting-styles/boring.css}">` in `greeting.html` könnten wir auch drinnen lassen, damit dieses Styling auch dann greift, wenn der in `cssFileName` mitgegebene Wert nicht zu einer existierenden CSS-Datei passt.
In der ausgewählten CSS-Datei können alle Werte aus `boring.css` überschrieben werden, weil bei CSS neuere Regeln alte überschreiben.



<!--
## Optionaler Abschweifer: Kollaborativ arbeiten mit git (hier: Github)
Gemeinsam erweitern wir unsere Hauptseite um weitere Begrüßungslinks.
Folgende konzeptionellen Schritte müssen wir dafür durchgehen:
- wir ziehen uns mit `git clone` den aktuellen Stand vom Dozenten
- erstellen dann jeweils einen neuen Branch
- implementieren unseren Teil und erstellen einen Commit
- diesen neuen Branch laden wir hoch
- bei Github erstellen wir einen *Pull Request* (bei Gitlab anderer Name: *Merge Request*), sodass der Dozent die Änderungen sehen und automatisiert bei sich einpflegen kann

Dafür werden wir folgende Befehle brauchen:
1. `git clone`
    - `git clone <URL_TO_REPO>` lädt ein Projekt von Github oder Gitlab runter auf unseren Laptop, inklusive aller git-Sachen, die dran hängen
2. `git branch`
    - Branch erstellen: `git branch <BRANCH_NAME>`, dann müssen wir noch wechseln zum gerade erstellten Branch
3. `git checkout`
    - zu einem Branch wechseln: `git checkout <BRANCH_NAME>`
    - einen Branch erstellen und direkt dahin wechseln: `git checkout -b <BRANCH_NAME>`
    - alle lokalen Änderungen einer Datei rückgängig machen, die noch nicht commitet wurden (Vorsicht, die Änderungen sind für immer weg!): `git checkout <FILE_NAME>`
4. `git merge`
    - `git merge <BRANCH_NAME>` versucht, die Änderungen des angegebenen Branches in den rein zu ziehen, in dem wir uns aktuell lokal befinden
    - `git mergetool` zum Nutzen einer optional eingerichteten Software, mit der mögliche Merge Konflikte mit einer GUI behandelt werden können

Nach Ausführen von Schritten 1 bis 3, können wir lokal das Projekt in unserer IDE öffnen und unsere Begrüßung ergänzen.
- erstelle neue CSS-Datei unter `static/greeting-styles/`
  - möglichst Namenskollisionen mit anderen Leuten im Kurs vermeiden, damit es später leichter ist, die Pull Requests ohne Konflikte zu vereinen
- ergänze Link mit passenden RequestParams für Name und CSS-Datei in der `index.html`
- erstelle Commit, wenn alles funktioniert
- mit `git push` wird der branch mit dem neuen Commit gepusht
- am Beamer geht es weiter.

Schritt 4 führen wir nur aus, um lokal zwei Branches zu mergen.
Wenn es dabei zu Konflikten kommt, kann ein unter `git mergetool` eingerichtetes Programm helfen, übersichtlich Änderungen manuell zu vereinen.

Stattdessen nutzen wir aber die GUI von Github, um einen Pull Request zu erstellen, den der Repository-Besitzer (oder jemand mit passenden Rechten) auch in der GUI von Github annehmen kann.



## `curl` und verschiedene Rückgabe dank Request-Headers

### Im Webbrowser (zB Firefox)

Die Headers eines HTTP-Requests können dem empfangenden Backend weitere Informationen mitgeben, wie gewünschtes Dateiformat oder Sprache.
Beispielhaft wollen wir unsere bisherigen zwei Endpunkte mit jeweils HTML- und JSON-Rückgabe abändern, sodass sie unter dem gleichen Pfad erreichbar sind.

<b style="color: pink; background-color: black; padding: 0.1rem;">
Achtung:
In unserem Beispiel machen diese zwei Endpunkte ganz verschiedene Dinge. Einer gibt eine Begrüßung zurück, der andere gibt ein Dummy POJO zurück.
Zwei Endpunkte sollten nur unter dem gleichen Pfad erreichbar sein, wenn sie konzeptionell die gleiche Sache machen und zurückgeben.
Damit unser Beispielcode schlank und einfach bleibt, ändern wir die JSON-Rückgabe aber nicht ab.
</b>

Wir öffnen unsere Hauptseite im Firefox Web Browser, öffnen die Dev-Tools (zB mit `F12`), klicken dort auf den **Network**-Tab und klicken dann auf unserer Seite auf den Link zur Begrüßung.
Im Network-Tab sehen wir, wenn wir den entsprechenden GET-Request anklicken, dass in den Headers `Accept: text/html,[...]` steht.

![1. Screenshot der Dev-Tools mit markierten Stellen, die man anklicken sollte.](images/ff-dev-accept.png "1. Screenshot Klickpfad Dev-Tools")

Mit einem Rechtsklick auf den Request oben und dann Klick auf `Edit and Resend`, können wir die Headers ändern und schreiben bei Accept rein: `application/json` und schicken den Request ab.

![2. Screenshot der Dev-Tools mit markierten Stellen, die man anklicken sollte.](images/ff-dev-new-accept.png "2. Screenshot Klickpfad Dev-Tools")

Als Antwort bekommen wir jetzt den Status-Code `406 Not Acceptable`, denn unser Backend liefert hinter diesem Pfad noch keine JSON.
Das ändern wir jetzt und probieren das gleiche nochmal.

### Endpunkte mit gleichem Pfad, aber verschiedenen Rückgabetypen

Wenn wir bei unseren beiden bisherigen Endpunkten in der Annotation `@GetMapping("...")` den gleichen String als Pfad übergeben und dann unser Programm starten, stürzt es direkt ab, u.a. mit Fehler 
`java.lang.IllegalStateException: Ambiguous mapping. Cannot map 'greetingController' method`

Wir müssen die Endpunkte noch nach Rückgabetyp unterscheiden.
Bei Spring reicht es, in der HTTP-Verb-Annotation die Variable `produces` zu setzen mit dem korrekten Wert für die entsprechenden Rückgabetypen.
Anstatt standardisierte Strings wie `"text/html"` oder `"application/json"` auswendig zu kennen, können wir hier den Enum `MediaType` verwenden.
Wir tauschen also `@GetMapping("/greeting")` beim ersten Endpunkt aus durch:
  ```java
  @GetMapping(value = "/greeting", produces = MediaType.TEXT_HTML_VALUE)
  ```
und beim JSON-Endpunkt schreiben wir:
  ```java
  @GetMapping(value = "/greeting", produces = MediaType.APPLICATION_JSON_VALUE)
  ```
Ansonsten ändern wir nichts an den Endpunkten.

Jetzt können wir unseren Request mit angepassten Headern erneut in den Dev-Tools von Firefox abschicken und die JSON-Antwort sehen.


### Requests mit `curl`

Die beiden GET-Requests können wir folgendermaßen im Terminal mit `curl` ausführen, um uns für spätere POST-Requests aufzuwärmen:

```
curl localhost:8080

curl localhost:8080/greeting -H "accept: text/html"
curl localhost:8080/greeting -H "accept: application/json"

curl -H "accept: text/html" localhost:8080/greeting
curl -H "accept: application/json" localhost:8080/greeting

curl localhost:8080/greeting -H "accept: text/html"
curl localhost:8080/greeting -H "accept: application/json"

curl 'localhost:8080/greeting?name=Alex&css=style-alex' -H "accept: text/html"
curl localhost:8080/greeting?name=Paul -H "accept: application/json"
```

Beim vorletzten `curl`-Befehl sind die String-Delimiter beim Pfad wichtig, da sonst das `&`-Zeichen im Pfad zu Problemen führt.



## POST-Requests und HTML-forms

### POST-Request basteln im Backend
Laut HTTP ist das POST-Verb dafür gedacht, Datensätze anzulegen.
Praktisch wird es häufig einfach dafür verwendet, bei einem HTTP-Request auch einen Request Body mitzuschicken -- normalerweise mit einem JSON-Objekt als Datenträger.

Wir basteln zuerst auch nur einen POST-Request, der keine Daten anlegt.
Unser erstes Ziel ist es, die Daten im Body richtig zu empfangen und zB über `System.out.println(...)` im Terminal anzuzeigen, oder sie in einer Antwort mit JSON oder Thymeleaf zurückzugeben.

Dafür bereiten wir einen POST-Endpunkt vor, der im Body die Daten enthält, mit denen wir unser vorher verwendetes POJO basteln können.
Später werden wir dann dieses Objekt in einer Datenbank persistieren wollen.

- mit curl Testen, Infos im JSON-Body mitschicken
  - zuerst mit System.out.println(...)
  - jetzt auch mit Thymeleaf als Rückgabe, zuerst mit curl und die HTML-Antwort im Terminal bestaunen, aber nichts weiter damit anfangen

TODO ALEX

### POST-Request mit HTML-Form
Die Daten, die wir im Body mitschicken, wollen aus einer Eingabe vom User nehmen.

Beachte, dass wir Strings theoretisch auch als Request-Params bei einem GET-Request schicken könnten.
Diese sind für Einstellungen/Optionen gedacht, und nicht für zu speichernde Informationen.
Entsprechend gibt es bei HTML auch keine einfache Möglichkeit (ohne JavaScript) Request-Params mitzuschicken.
Wir können aber einen Request-Body mitschicken.

#### ohne Thymeleaf, nur HTML
Wir basteln eine HTML-Form mit ein paar Textfeldern, die wir dann per POST-Request an unseren Endpunkt schicken.

TODO ALEX


#### mit Thymeleaf
Jetzt basteln wir wieder eine HTML-Form, aber direkt mit Thymeleaf. Wir wollen den gleichen POST-Rquest abschicken, wie direkt davor ohne Thymeleaf.

TODO ALEX, dabei auch Vorteile im Vergleich zu normalen HTML-Forms herausstellen!




## Daten speichern in einer Datenbank
Unsere Daten wollen wir jetzt auch mal abspeichern, sodass sie nach einem Request noch verfügbar sind.

Es gibt viele Lösungen für Datenbanken, die man anbinden kann.
Manche laufen als "In-Memory"-Datenbank und werden, wie der Arbeitsspeicher bei einem Laptop, komplett gelöscht beim Ausschalten des Programms.
In solchen Datenbanken werden vor allem Dateien gecached, nicht auf Dauer zur Nutzung über mehrere Sitzungen persistiert.

Wir verwenden hier H2 ("Hypersonic 2") als Datenbank.
H2 ist in Java geschrieben und bietet bei Spring eine einfache Einbindung mit tollen Features zum Herumprobieren. 

Mehr Infos zu Spring mit H2: https://www.baeldung.com/spring-boot-h2-database


### Spring JPA und H2-Datenbank (In-Memory mode: temporär)
Mit JPA (und für Feinschmecker dabei: JPQL) sparen wir uns das Schreiben der SQL-Queries.
Damit JPA irgendwelche Daten ansprechen kann, brauchen wir dann auch eine Datenbank, mit der sich unser Programm verbinden kann.

#### JPA im Projekt einbinden
ENTWEDER ganz am Anfang im Spring-Initializr eine weitere Dependency auswählen: **Spring Data JPA**.
ODER einfach in der `pom.xml` folgendes einfügen in dem `<dependencies>`-Tag:
  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  ```

Wenn wir jetzt Starten (im Terminal mit `./mvnw spring-boot:run` oder über Play-Button in IntelliJ) gibt es einen Fehler:
  ```
  Error creating bean with name 'entityManagerFactory' [...]
  ```
  - kurz gesagt: Spring findet keine Datenbank, also erstellen wir eine

#### H2-Datenbank erstellen
Mit Maven binden wir ganz einfach H2 ein.
Am besten selbst selbst mit einer Suchmaschine zB nach "maven h2" suchen und bei der Seite von `mvnrepository.com` zu H2 landen.
Dort auf eine neue Version klicken, den `<dependency>`-Tag rauskopieren und dann einfügen in die `pom.xml`.
Sieht dann zum Beispiel so aus:
```xml
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <version>2.3.232</version>
</dependency>
```
- falls bisher noch nicht erwähnt: online wird direkt unter dem `<version>`-Tag wahrscheinlich noch ein `<scope>test</scope>` stehen.
  Hier wird festgelegt, in welchem Scope diese Dependency zur Verfügung stehen soll.
  Damit wollen wir uns aktuell nicht auseinander setzen, also können wir die Zeile auch weglassen.
  Mehr Infos: [Baeldung -- Maven Dependency Scopes](https://www.baeldung.com/maven-dependency-scopes)

Die Einbindung der Dependency in Maven sollte bereits ausreichen.
Spring setzt bei H2 automatisch Default-Werte.
Diese Setzen wir aber trotzdem noch manuell in den `application.properties`, weil wir sie später noch anpassen möchten.
Und damit wir ein Gefühl für solche Konfigurationen bekommen:
  - TODO ALEX checken, ob es die default-Konfig für H2 wirklich nicht braucht
  ```
  spring.datasource.url=jdbc:h2:mem:testdb
  spring.datasource.driverClassName=org.h2.Driver
  spring.datasource.username=sa
  spring.datasource.password=
  ```
  - die letzte Zeile hat einen leeren Wert, damit das Passwort leer bleibt und wir kein Passwort eingeben müssen.
    Für die lokale Entwicklung ist das praktisch.


### Initiale Tabellen und Datensätze anlegen

TODO ALEX
- wir könnten mit POST-Requests (curl) selbst Daten basteln, aber wir haben noch keine Speicherung der Daten über Endpunkte implementiert, d.h. das schauen wir uns erst später an
- zuerst mit dem CommandLineRunner Bean nahe der main ein paar JPA-Queries abfeuern
- dann evtl die Lösung in irgendeinem Firefox-Tab mit der .sql-Datei, die Spring wohl automatisch aufsammelt? -> mehr recherchieren
- dann ersetzen wir das aber durch Flyway-Migration-Skripte



### Datenbank persisteren: H2-Datenbank, als File lokal abgespeichert

Als file anlegen (damit wir über den Browser mal reinschauen können):
  ```
  spring.datasource.url=jdbc:h2:file:./data/mynewdb
  ``` 

Manueller Zugriff auf persistierte H2-Datenbank im Browser unter `localhost:8080/h2-console`
	- dort passenden Pfad zur Datei eingeben, sollte danach immer gespeichert bleiben im Browser. Eventuell auch Passwort und Username, falls man die in den `application.properties` geändert hat

TODO ALEX VERSCHIEBEN:
Schreibe SQL-Queries, um eine neue Tabelle zu erstellen mit 2-3 Einträgen:
```sql
CREATE TABLE journal_entry (
    id int NOT NULL,
    name varchar(50) NOT NULL,
    description varchar(255),
    content text,
    PRIMARY KEY (id)
);
INSERT INTO journal_entry VALUES (1, 'Start', 'es geht los', 'So fing alles an. Was ist schon lange her, dass ...');
INSERT INTO journal_entry VALUES (2, 'Abend', 'hammer', 'Was ein Tag es war, was ich alles gelernt habe ...');
```


************

## TODO ALEX Ideen:
- CHECKEN: Endpunkt gleicher Pfad aber durch Request-Header (wie stellen; wie verarbeiten bei Annahme?) kriegen wir JSON oder HTML zurück
- Endpunktantwort: mit selbst gesetztem HTTP-Status-Code
  - und evtl weiteren Informationen im HTTP-Header
- Datenbank initialisieren:
  - In-Memory zuerst in 
- Entity in Java als Objekt anlegen, außerdem Repository (ohne Service), per Controller zurückgeben (siehe Projekt)

- am Ende noch User-Authentication und Rollen einbauen???
-->

