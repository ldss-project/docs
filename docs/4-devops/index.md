---
title: DevOps
layout: default
nav_order: 6
---

# DevOps
{: .no_toc}

In questo capitolo si approfondiranno le buone pratiche di _DevOps_ che sono
state adottate dal team durante lo sviluppo del sistema, definendo un processo
e un ambiente di sviluppo standard per i membri del team, automatizzando i
processi d'installazione, verifica, controllo qualità e pubblicazione del sistema.

Lo scopo di questo processo è di promuovere l'autonomia dei membri del team e
l'incrementalità dello sviluppo del sistema, di automatizzare il più possibile le
attività ripetitive del processo di sviluppo e di garantire la riproducibilità dei
fallimenti e la robustezza del sistema.

## Contenuti
{: .no_toc}

- TOC
{:toc}

---

## Workflow Organization

In questa sezione, si descriverà il processo di sviluppo standard adottato dal team
per realizzare questo sistema.

### GitHub Organization

Il sistema da realizzare è stato suddiviso in diversi componenti software indipendenti
tra di loro, ma comunque parte dello stesso sistema.

Per questo, si è deciso di separare lo sviluppo di ciascun componente in un _repository_
corrispondente, ma anche di raggruppare tutti i _repository_ all'interno di una unica
_GitHub Organization_, corrispondente al sistema nella sua interezza.

I vantaggi di utilizzare una _GitHub Organization_ in questo progetto sono stati quelli
di poter accedere facilmente ai _repository_ che vi appartengono, separandoli da altri
progetti, e di condividere delle impostazioni, delle variabili e dei segreti che sarebbero
stati altrimenti ripetuti in ogni _repository_.

Di seguito, si riporta la struttura della _GitHub Organization_ creata:
- [ldss-project](https://github.com/ldss-project): la _GitHub Organization_ relativa al progetto
  per il corso di [Laboratorio di Sistemi Software 2022-2023](https://www.unibo.it/it/didattica/insegnamenti/insegnamento/2022/412677).
  Contiene i seguenti _repository_:
  - [hexarc](https://github.com/ldss-project/hexarc): framework per la definizione ed il deployment di
    servizi secondo le buone pratiche della _Clean Architecture_.
  - [wartremover-gradle-plugin](https://github.com/ldss-project/wartremover-gradle-plugin): plugin gradle
    per configurare [WartRemover](https://www.wartremover.org/) come _code linter_ in un progetto Scala 3.
  - [scala3-project-template](https://github.com/ldss-project/scala3-project-template): _template repository_
    per la creazione di progetti Scala 3.
  - [frontend](https://github.com/ldss-project/frontend): implementazione del **Frontend
    Service**, come descritto nei capitoli precedenti.
  - [authentication-service](https://github.com/ldss-project/authentication-service): implementazione
    dell'**Authentication Service**, come descritto nei capitoli precedenti.
  - [statistics-service](https://github.com/ldss-project/statistics-service): implementazione
    dello **Statistics Service**, come descritto nei capitoli precedenti.
  - [chess-game-service](https://github.com/ldss-project/chess-game-service): implementazione
    del **Chess Game Service**, come descritto nei capitoli precedenti.
  - [swagger-apis](https://github.com/ldss-project/swagger-apis): sito web per la pubblicazione
    delle API dei servizi del sistema (creato per non essere vincolati dai limiti sulle pubblicazioni
    imposti dalla licenza gratuita di [SwaggerHub](https://swagger.io/tools/swaggerhub/)).

### DVCS Workflow

Durante lo sviluppo del sistema, si è cercato il più possibile di suddividere il lavoro tra i membri del team,
in modo da ridurre al minimo i possibili conflitti di pubblicazione del codice sui _repository_. In particolare,
nella maggior parte dei casi, a ogni _repository_ ha lavorato principalmente un unico membro del team, con piccole
eccezioni durante l'integrazione tra i vari moduli del sistema.

L'approccio adottato per lo sviluppo all'interno dei singoli _repository_ è una versione semplificata di _Git Flow_
e, in generale, prevede i seguenti branch:
- **master** (o **main**): branch utilizzato per la pubblicazione del software.
- **feature/\<topic>**: branch utilizzato per aggiungere una feature nel software. Nasce dal **master** e ritorna
  sul **master**.
- **fix/\<topic>**: branch utilizzato per rimuovere un difetto nel software. Nasce dal **master** e ritorna
  sul **master**.
- **\<module>/\<topic>**: branch utilizzato per modificare uno specifico modulo del software, utile per identificare
  velocemente la parte del software modificata maggiormente nel branch, quando il software è composto da numerosi
  moduli, come ad esempio il **Frontend** (logica, pagine, componenti grafici...). Nasce dal **master** e
  ritorna sul **master**.

All'interno di un branch, i commit aderiscono allo standard [Conventional Commit](https://www.conventionalcommits.org/en/v1.0.0/)
in modo da supportare il versionamento automatico del software. I principali tipi di commit utilizzati sono i seguenti:
- **feat**: indica che il commit ha aggiunto una nuova funzionalità al software visibile ai suoi utilizzatori.
- **fix**: indica che il commit ha rimosso un difetto dal software visibile ai suoi utilizzatori.
- **docs**: indica che il commit ha aggiornato la documentazione del software.
- **build**: indica che il commit ha modificato uno script di _build automation_ (ad esempio, ha aggiunto una dipendenza
  al software o ha modificato il processo di creazione degli artefatti software...).
- **ci**: indica che il commit ha modificato uno script di _continuous integration_ (ad esempio, ha aggiunto un _GitHub
  Workflow_...).
- **test**: indica che il commit ha aggiunto un test per una parte del software.
- **chore**: indica che il commit ha modificato una parte del software non visibile ai suoi utilizzatori (ad esempio,
  inizializzazione del _repository_, refactoring, aggiunta di un componente non ancora utilizzato...).
- **\<module>**: equivalente a **chore(\<module>)**. Utilizzato per identificare velocemente la parte del software
  modificata maggiormente nel commit, quando il software è composto da numerosi moduli, come ad esempio il **Frontend**
  (**logic**, **page**, **component**, **style**...).

### Versioning

Per quanto riguarda il versionamento del software, si è deciso di adottare lo standard [Semantic Versioning](https://semver.org/)
in modo da garantire che una modifica sulla versione del software rifletti un effettivo cambiamento del software dal
punto di vista dei suoi utilizzatori.

Il vantaggio di adottare tale standard è la disponibilità di alcuni strumenti per automatizzare il versionamento e la
pubblicazione del software, come [Semantic Release](https://github.com/semantic-release/semantic-release), che permette
di assegnare una versione al software sulla base della sua storia dei commit. Questo stesso strumento sarà anche
utilizzato per automatizzare la pubblicazione degli artefatti software del sistema.

In maggior dettaglio, la tecnica di versionamento adottata parte dalla versione **0.1.0**, come prima versione instabile
di un _repository_. Quindi, ad ogni aggiornamento del branch **master**, la storia dei commit del branch viene controllata,
eventualmente producendo una nuova versione del software che viene associata come tag all'ultimo commit del branch.

In particolare, la versione del software viene modificata come segue:
- Se la storia dei commit dall'ultima versione contiene un _Breaking Change_, la versione del _repository_
  viene incrementata di una _Major_;
- Altrimenti, se la storia dei commit dall'ultima versione contiene un commit di tipo **feat**, la versione
  del _repository_ viene incrementata di una _Minor_;
- Altrimenti, se la storia dei commit dall'ultima versione contiene un commit di tipo **fix**, la versione
  del _repository_ viene incrementata di una _Patch_;
- Altrimenti, la versione non viene modificata.

In questo progetto, non si è mai fatto uso di _Breaking Change_ esplicitamente, infatti i moduli realizzati
sono ancora in una versione instabile, per cui ogni modifica del software potrebbe comportare delle modifiche
sostanziali alla sua API.

### GitHub Template Repositories

Per evitare di dover definire la stessa configurazione per ogni _repository_ quando creato, sono stati utilizzati
dei _template repository_.

In particolare, per la documentazione dei moduli è stato applicato un template di terze parti, chiamato
[just-the-docs](https://just-the-docs.com/), mentre per l'inizializzazione di progetti Scala 3 è stato
creato un template interno, chiamato [scala3-project-template](https://github.com/ldss-project/scala3-project-template).

Un altro metodo utilizzato agli stessi scopi, è quello di creare delle _fork_ da progetti pre-configurati. Ad
esempio, questa stessa documentazione è stata creata a partire da una _fork_ della documentazione del progetto
realizzato per il corso di [Project Management 2022-2023](https://www.unibo.it/it/didattica/insegnamenti/insegnamento/2022/412683),
disponibile al seguente [link](https://github.com/jahrim/pm).

## Build Automation

In questa sezione, si descriveranno gli strumenti e gli script utilizzati per la generazione e pubblicazione 
degli artefatti del sistema.

> _**NOTA**_: siccome ogni modulo del sistema può avere dei requisiti diversi per quanto riguarda la
> _build automation_, per semplicità si riporterà solo l'approccio più comunemente utilizzato.

### Build Automation Tools

I principali strumenti di _build automation_ utilizzati in questo progetto sono:
- [Gradle](https://gradle.org/): utilizzato per generare e pubblicare gli artefatti dei progetti Scala 3.
- [npm](https://www.npmjs.com/): utilizzato per generare gli artefatti del **Frontend Service** e per
  l'integrazione di [Semantic Release](https://github.com/semantic-release/semantic-release) nei moduli del sistema.

Per quanto riguarda il **Frontend Service**, si eviterà di scendere nel dettaglio poiché gli script utilizzati sono
quelli forniti di default dall'installazione di [Vue + Vite](https://vuejs.org/guide/quick-start.html), solo leggermente
modificati. Pertanto, ci si concentrerà sulla descrizione della _build automation_ con [Gradle](https://gradle.org/).

### Gradle Catalog

Per la gestione delle dipendenze dei moduli del sistema, si è fatto uso del _Gradle Catalog_, che permette
di raggruppare gli identificatori delle dipendenze e le loro versioni in un unico file, facilitandone la
manutenzione e l'aggiornamento tramite strumenti automatici.

In particolare, per automatizzare l'aggiornamento delle dipendenze dei moduli del sistema, è stato abilitato
[Renovate](https://github.com/renovatebot/renovate) sui _repository_ del progetto. In questo modo, le
_repository_ sono monitorate periodicamente alla ricerca di dipendenze da aggiornare e in tal caso vengono
aperte automaticamente delle pull request sul _repository_ per aggiornare tali dipendenze.

[Renovate](https://github.com/renovatebot/renovate) funziona non solo sui _Gradle Catalog_, ma anche su
concetti equivalenti di altri strumenti di _build automation_, come i _package.json_ di [npm](https://www.npmjs.com/),
per cui è stato possibile abilitarlo anche per il **Frontend Service**.

### Module Information

Le informazioni e i metadati relativi ai moduli del sistema sono racchiusi nel _build.gradle.kts_ in
una classe creata appositamente. Tali informazioni sono necessarie alla generazione degli artefatti e
alla loro pubblicazione.

Di seguito, si riporta un esempio delle informazioni utilizzate per descrivere un modulo del sistema.

```kotlin
private class ProjectInfo {
    val longName: String = "Scala 3 Project Template"
    val description: String = "A template for configuring Scala 3 projects."
  
    val repositoryOwner: String = "jahrim"
    val repositoryName: String = "scala3-project-template"
  
    val artifactGroup: String = "io.github.jahrim"
    val artifactId: String = project.name
    val implementationClass: String = "main.main"
  
    val license = "The MIT License"
    val licenseUrl = "https://opensource.org/licenses/MIT"
  
    val website = "https://github.com/$repositoryOwner/$repositoryName"
    val tags = listOf("scala3", "project template")
}
private val projectInfo: ProjectInfo = ProjectInfo()
```

### Quality Assurance

Per il controllo di qualità dei moduli del sistema, sono stati utilizzati diverse categorie di strumenti:
- **Linting Tools**: strumenti per il controllo dello stile del codice. Permettono agli sviluppatori di
  adottare uno stile di programmazione standard, facilitando la comprensione del codice altrui.
  - `Scala 3`:
    - [WartRemover](https://www.wartremover.org/)
  - `Kotlin`:
    - [Ktlint](https://github.com/pinterest/ktlint)
  - `Javascript` / `Typescript` / `Vue`:
    - [ESLint](https://eslint.org/)
- **Static Code Analysis**: strumenti per la ricerca di possibili difetti del codice. Permettono di prevenire
  alcuni problemi dovuti al cattivo utilizzo dei costrutti fragili nei linguaggi di programmazione.
  - `Scala 3`:
    - [WartRemover](https://www.wartremover.org/)
  - `Kotlin`:
    - [Detekt](https://github.com/detekt/detekt)
- **Code Formatter**: strumenti per la formattazione del codice. Permettono agli sviluppatori di adottare uno
  stile di formattazione del codice standard, migliorando la leggibilità e la manutenibilità del codice.
  - `Scala 3`:
    - [Scala Formatter](https://scalameta.org/scalafmt/)
  - `Kotlin`:
    - [Ktlint](https://github.com/pinterest/ktlint)
  - `Javascript` / `Typescript` / `Vue`:
    - [Prettier](https://prettier.io/)

Per applicare questi controlli sulla qualità del codice, sono stati utilizzati i seguenti strumenti di
appoggio:
- [spotless/plugin-gradle](https://github.com/diffplug/spotless/tree/main/plugin-gradle): plugin che 
  permette l'applicazione e la configurazione di numerosi formattatori per linguaggi di programmazione
  diversi. La tipica configurazione di tale plugin nei moduli è la seguente:
  ```kotlin
  spotless {
    isEnforceCheck = false
    scala { scalafmt(libs.versions.scalafmt.version.get()).configFile(".scalafmt.conf") }
    tasks.compileScala.get().dependsOn(tasks.spotlessApply)
  }
  ```
  
  Come si può osservare, il plugin è stato utilizzato per configurare
  [Scala Formatter](https://scalameta.org/scalafmt/) in modo che esegua ogni volta che il
  codice del modulo viene compilato e non solo durante la fase di _check_.
  Così facendo, la formattazione viene applicata anche quando il codice viene eseguito o testato
  all'interno di [IntelliJ Idea](https://www.jetbrains.com/idea/).
- [wartremover-gradle-plugin](https://github.com/ldss-project/wartremover-gradle-plugin): plugin
  che permette la configurazione di [WartRemover](https://www.wartremover.org/) come _code linter_
  per un progetto Scala 3.

  Il plugin era stato realizzato come un'_API Extension_ all'interno del progetto per il corso di
  [Paradigmi di Programmazione e Sviluppo 2022-2023](https://www.unibo.it/it/didattica/insegnamenti/insegnamento/2022/412597)
  (reperibile al seguente [link](https://github.com/jahrim/PPS-22-chess)), dal quale è stato estratto, quindi 
  refattorizzato, testato e pubblicato su [Gradle Plugin Portal](https://plugins.gradle.org/).
  Il nuovo _repository_ del plugin è disponibile al seguente [link](https://github.com/ldss-project/wartremover-gradle-plugin).
- [kotlin-qa](https://github.com/DanySK/gradle-kotlin-qa): plugin che permette la configurazione di
  [Detekt](https://github.com/detekt/detekt) e [Ktlint](https://github.com/pinterest/ktlint) per un progetto Kotlin.
- [create-vue](https://github.com/vuejs/create-vue): lo script ufficiale per eseguire il _Project Scaffolding_
  per un progetto Vue, che include la configurazione di [ESLint](https://eslint.org/) e [Prettier](https://prettier.io/).

### Artifacts

Per la generazione degli artefatti di ogni modulo, sono state predisposte tre task principali:
- **sourcesJar**: produce il jar contenente i sorgenti del modulo.
- **javadocJar**: produce il jar contenente la documentazione del modulo.
- **jar**: produce il jar eseguibile di un modulo che può essere eseguito.

Di seguito, si riporta la configurazione relativa a tali task.

```kotlin
group = projectInfo.artifactGroup
gitSemVer {
    buildMetadataSeparator.set("-")
    assignGitSemanticVersion()
}

tasks.jar {
    dependsOn(configurations.runtimeClasspath)
    from(sourceSets.main.get().output)
    from(configurations.runtimeClasspath.get().filter { it.name.endsWith("jar") }.map { zipTree(it) })

    manifest.attributes["Main-Class"] = projectInfo.implementationClass
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
}

tasks.javadocJar {
    dependsOn(tasks.scaladoc)
    from(tasks.scaladoc.get().destinationDir)
}

// tasks.sourcesJar imported from "Publish On Central Gradle Plugin"
```

Come si può notare, non è stata definita una task relativa ai **sourcesJar**, infatti i moduli dipendono
dal plugin [Publish On Central](https://github.com/DanySK/publish-on-central), che già configura tale
task in maniera adeguata per la pubblicazione su [Maven Central](https://central.sonatype.com/).

Inoltre, per determinare la versione degli artefatti generati, si è utilizzato il plugin 
[Git Sensitive Semantic Versioning](https://github.com/DanySK/git-sensitive-semantic-versioning-gradle-plugin), che
configura come versione degli artefatti l'ultimo tag conosciuto nella storia dei commit relativa al branch attuale.
In sinergia con [Semantic Release](https://github.com/semantic-release/semantic-release), questo plugin permette di
generare degli artefatti la cui versione aderisce allo standard [Semantic Versioning](https://semver.org/).

### Publishing on Maven

Per la pubblicazione degli artefatti su [Maven Central](https://central.sonatype.com/), come già anticipato,
si è utilizzato il plugin [Publish On Central](https://github.com/DanySK/publish-on-central).

Di seguito, si riporta la configurazione comunemente utilizzata per pubblicare gli artefatti su
[Maven Central](https://central.sonatype.com/).

```kotlin
publishOnCentral {
    configureMavenCentral.set(true)
    projectDescription.set(projectInfo.description)
    projectLongName.set(projectInfo.longName)
    licenseName.set(projectInfo.license)
    licenseUrl.set(projectInfo.licenseUrl)
    repoOwner.set(projectInfo.repositoryOwner)
    projectUrl.set(projectInfo.website)
    scmConnection.set("scm:git:$projectUrl")
}

publishing {
    publications {
        withType<MavenPublication> {
            pom {
                developers {
                    developer {
                        name.set("Jahrim Gabriele Cesario")
                        email.set("jahrim.cesario2@studio.unibo.it")
                        url.set("https://jahrim.github.io")
                    }
                    developer {
                        name.set("Madina Kentpayeva")
                        email.set("madina.kentpayeva@studio.unibo.it")
                        url.set("https://madina9229.github.io")
                    }
                }
            }
        }
    }
}

signing {
    val signingKey: String? by project
    val signingPassword: String? by project
    useInMemoryPgpKeys(signingKey, signingPassword)
}
```

Siccome [Maven Central](https://central.sonatype.com/) necessita che gli artefatti siano firmati prima della
chiusura del repository che li contiene, è stato necessario configurare il plugin [Signing](https://docs.gradle.org/current/userguide/signing_plugin.html)
per utilizzare la chiave privata e la password dell'autore, in modo che il plugin potesse ricevere tali credenziali sotto
forma di _project properties_.

### Publishing on Gradle Plugin Portal

Per la pubblicazione dei plugin su [Gradle Plugin Portal](https://plugins.gradle.org/) è stato
utilizzato il plugin [Gradle Publishing Plugin](https://plugins.gradle.org/plugin/com.gradle.plugin-publish).

Di seguito, si riporta la configurazione comunemente utilizzata per pubblicare i plugin su
[Gradle Plugin Portal](https://plugins.gradle.org/).

```kotlin
gradlePlugin {
    website.set(projectInfo.website)
    vcsUrl.set("${projectInfo.website}.git")
    plugins {
        create("wartremover") {
            id = "${projectInfo.artifactGroup}.${projectInfo.artifactId}"
            displayName = projectInfo.longName
            description = projectInfo.description
            tags.set(projectInfo.tags)
            implementationClass = projectInfo.implementationClass
        }
    }
}

val setupPublishPlugin by tasks.registering {
    val gradlePublishKey: String? by project
    val gradlePublishSecret: String? by project
    gradlePublishKey?.apply{ System.setProperty("gradle.publish.key", this) }
    gradlePublishSecret?.apply{ System.setProperty("gradle.publish.secret", this) }
}
tasks.named("publishPlugins"){ dependsOn(setupPublishPlugin) }
```

Come si può notare, la task per la pubblicazione dei plugin fornita dal
[Gradle Publishing Plugin](https://plugins.gradle.org/plugin/com.gradle.plugin-publish) è stata configurata
in modo da poter accettare delle _project properties_ compatibili per essere specificate come variabili d'ambiente.
In questo modo, sarà più facile utilizzare il plugin all'interno della _CI_.

### Docker Build

Un altro aspetto importante della _build automation_ è la generazione di immagini per [Docker](https://www.docker.com/),
che saranno uno tra gli strumenti più importanti per semplificare l'esecuzione del sistema nella sua interezza.

A tale scopo, per ogni modulo eseguibile è stato definito un _Dockerfile_ che sarà utilizzato per costruire un'immagine
del modulo e quindi pubblicarla successivamente.

Per quanto riguarda il **Frontend Service**, è stato definito il seguente _Dockerfile_:

```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build-only          # vite build
EXPOSE 8080
CMD npm run dev:docker          # vite --host 0.0.0.0 --port 8080
```

Come si può osservare, il _Dockerfile_ si compone dei seguenti _stage_ principalmente:
1. Inizializza un container con sistema operativo [Alpine Linux](https://www.alpinelinux.org/), provvisto
   di [NodeJS](https://nodejs.org/);
2. Copia i file contenenti le dipendenze del modulo;
3. Installa le dipendenze del modulo;
4. Copia tutti i sorgenti del modulo, selezionandoli attraverso un'opportuna configurazione
   del file _.dockerignore_;
5. Produce l'eseguibile per il modulo;
6. Esegue il modulo.

La suddivisione in _stage_ circoscritti è importante per sfruttare al meglio i meccanismi
di caching di [Docker](https://www.docker.com/), riducendo notevolmente il tempo necessario a
costruire l'immagine corrispondente al _Dockerfile_.

Per quanto riguarda gli altri moduli basati su [Java](https://www.java.com/), è stato definito il seguente _Dockerfile_:

```dockerfile
FROM eclipse-temurin:17-alpine
COPY ./build/libs/*.jar ./app/app.jar
WORKDIR ./app
EXPOSE 8080/tcp
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Come si può osservare, il _Dockerfile_ si compone dei seguenti _stage_ principalmente:
1. Inizializza un container con sistema operativo [Alpine Linux](https://www.alpinelinux.org/),
   provvisto di [Java](https://www.java.com/);
2. Copia il jar eseguibile del modulo, selezionandolo attraverso un'opportuna configurazione
   del file _.dockerignore_;
3. Esegue il modulo.

Questo _Dockerfile_ ha il vantaggio di produrre immagini con dimensioni molto ridotte, tuttavia
suppone che il jar eseguibile del modulo sia stato generato prima della costruzione dell'immagine.
In particolare, queste dipendenze saranno gestite durante la pubblicazione delle immagini dei
moduli del sistema all'interno della _continuous integration_.

## Continuous Integration

In questa sezione, si descriveranno gli strumenti e gli script utilizzati per la configurazione della
_continuous integration_ dei moduli del sistema.

> _**NOTA**_: siccome ogni modulo del sistema può avere dei requisiti diversi per quanto riguarda la
> _continuous integration_, per semplicità si riporterà solo l'approccio più comunemente utilizzato.

### Continuous Integration Tools

Come strumento principale per la configurazione della _continuous integration_ dei moduli del sistema,
è stato utilizzato [GitHub Actions](https://github.com/features/actions), siccome è già integrato nelle
_repository_ di GitHub.

### Continuous Testing

Per automatizzare l'esecuzione dei test nei moduli del sistema, è stato definito il seguente _workflow_:

```yaml
name: Test
on:
  pull_request:
    paths-ignore:
      - "README.md"
      - "docs/**"
jobs:
    # Test the project
    test:
      strategy:
        matrix:
          os: [windows, macos, ubuntu]
          java-version: [8, 11, 17]
      runs-on: ${{matrix.os}}-latest
      steps:
        # Install the specified version of Java in the provided runner
        - name: Install Java
          uses: actions/setup-java@v3
          with:
            distribution: 'adopt'
            java-version: ${{matrix.java-version}}
        # Clone the repository, with full history and submodules
        - name: Clone Repository
          uses: actions/checkout@v3
          with:
            token: ${{ secrets.GH_TOKEN }}
            fetch-depth: 0
            submodules: recursive
        # Validate the gradle wrapper to avoid supply chain attacks 
        # from pull-requests that change the wrapper
        - name: Validate Gradle Wrapper
          uses: gradle/wrapper-validation-action@v1
        # Run the project tests
        - name: Test Build
          run: |
            chmod 777 ./gradlew
            ./gradlew test

    # Completes successfully if all previous job were completed
    # successfully (i.e. exit code is 0)
    success:
      runs-on: ubuntu-22.04
      needs:
        - test
      # Executes always (even if previous jobs have failed, but not if
      # any of them has been cancelled)
      if: >-
        always() && (
          contains(join(needs.*.result, ','), 'failure')
          || !contains(join(needs.*.result, ','), 'cancelled')
        )
      # Succeeds only if all previous jobs have succeeded
      steps:
        - name: Verify that there were no failures
          run: ${{ !contains(join(needs.*.result, ','), 'failure') }}
```

Come si può notare, il _workflow_ prevede i due _job_ seguenti: 
- **test**: prevede i seguenti *step*:
  1. Installa [Java](https://www.java.com/), per poter eseguire il _Gradle Wrapper_;
  2. Clona il _repository_, per poter accedere al codice da testare;
  3. Valida il _Gradle Wrapper_ utilizzato per eseguire i test, per verificare che non sia stato manomesso;
  4. Esegue i test.

  Questo _job_ viene eseguito per diversi sistemi operativi e versioni di [Java](https://www.java.com/)
  attraverso una _build matrix_, aumentando la robustezza della verifica del sistema.
- **success**: controlla i risultati di tutti i _job_ eseguiti precedentemente e ha successo solo se sono tutti
  completati con successo.

  Questo _job_ è stato utilizzato principalmente per la configurazione della _branch protection_ sul **master**,
  per bloccare le pull request nel caso in cui non avesse avuto successo, evitando di dover specificare
  tutti i _job_ generati dalla _build matrix_ di **test**.

Siccome il _workflow_ esegue solamente quando una pull request viene aggiornata, è stato necessario definire una
_branch protection_ sul **master** che permettesse modifiche sul branch solo quando provenienti da una pull
request (impedendo una push diretta).
In questo modo, fin quando la _branch protection_ è rispettata dagli sviluppatori, si è sicuri che il codice sul
**master** sia propriamente testato.

Un _workflow_ molto simile è stato realizzato anche per il **Frontend Service**, con l'unica differenza che si
basa su [npm](https://www.npmjs.com/) + [Vitest](https://vitest.dev/) invece che su [Gradle](https://gradle.org/) +
[JUnit](https://junit.org/junit5/).

### Continuous Delivery

Per automatizzare la pubblicazione e la distribuzione dei moduli del sistema, è stato definito il seguente _workflow_:

```yaml
name: Publish
on:
  workflow_dispatch:
  push:
    branches:
      - master
    paths-ignore:
      - "README.md"
      - "docs/**"

jobs:
  # Publish the project
  publish:
    runs-on: ubuntu-latest
    env:
      GITHUB_TOKEN: ${{ secrets.GH_TOKEN }}
      GIT_COMMITTER_NAME: Jahrim Gabriele Cesario
      GIT_COMMITTER_EMAIL: jahrim.cesario2@studio.unibo.it
      GIT_AUTHOR_NAME: Jahrim Gabriele Cesario
      GIT_AUTHOR_EMAIL: jahrim.cesario2@studio.unibo.it
      DOCKER_REGISTRY_USER: ${{ secrets.DOCKER_USERNAME }}
      DOCKER_REGISTRY_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
      ORG_GRADLE_PROJECT_signingKey: ${{ secrets.GPG_SIGNING_KEY }}
      ORG_GRADLE_PROJECT_signingPassword: ${{ secrets.GPG_SIGNING_PASSWORD }}
      ORG_GRADLE_PROJECT_mavenCentralUsername: ${{ secrets.MAVEN_CENTRAL_USERNAME }}
      ORG_GRADLE_PROJECT_mavenCentralPassword: ${{ secrets.MAVEN_CENTRAL_PASSWORD }}
      ORG_GRADLE_PROJECT_gradlePublishKey: ${{ secrets.GRADLE_PORTAL_KEY }}
      ORG_GRADLE_PROJECT_gradlePublishSecret: ${{ secrets.GRADLE_PORTAL_SECRET }}
    steps:
      # Install the specified version of Java in the provided runner
      - name: Install Java
        uses: actions/setup-java@v3
        with:
          distribution: 'adopt'
          java-version: 8
      # Install Node in the provided runner
      - name: Install Node
        uses: actions/setup-node@v3
      # Clone the repository, with full history and submodules
      - name: Clone Repository
        uses: actions/checkout@v3
        with:
          token: ${{ secrets.GH_TOKEN }}
          fetch-depth: 0
          submodules: recursive
      # Validate the gradle wrapper to avoid supply chain attacks from
      # pull-requests that change the wrapper
      - name: Validate Gradle Wrapper
        uses: gradle/wrapper-validation-action@v1
      # Publish artifacts on Maven, Gradle Portal, GitHub and/or Docker
      - name: Publish on Maven, Gradle Portal, GitHub and/or Docker
        run: |
          npm clean-install
          npx semantic-release
```

Come si può notare, il _workflow_ prevede un unico _job_:
- **publish**: prevede i seguenti *step*:
  1. Installa [Java](https://www.java.com/), per poter eseguire il _Gradle Wrapper_;
  2. Installa [NodeJS](https://nodejs.org/); per poter eseguire lo script di 
     [Semantic Release](https://github.com/semantic-release/semantic-release);
  3. Clona il _repository_, per poter accedere al codice da pubblicare;
  4. Valida il _Gradle Wrapper_ utilizzato per generare e pubblicare gli artefatti del modulo,
     per verificare che non sia stato manomesso;
  5. Esegue uno script di [Semantic Release](https://github.com/semantic-release/semantic-release) per
     generare e pubblicare gli artefatti del modulo.

  In questo _job_, sono specificati come variabili d'ambiente tutti i segreti necessari per autenticarsi
  ai diversi repository su cui saranno pubblicati gli artefatti del modulo.

Di seguito, si riporta uno script di [Semantic Release](https://github.com/semantic-release/semantic-release)
che sintetizza gli script utilizzati per generare e pubblicare gli artefatti di ciascun modulo. Infatti, ogni
modulo ha necessità di pubblicare gli artefatti su dei portali specifici e la soluzione per pubblicare su
ciascun portale può essere estratta dallo script riportato.

```yaml
branch: master
tagFormat: "${version}"
plugins:
  # Analyzes commits to identify the type of update (e.g. fix -> PATCH,
  # feat -> MINOR...)
  - "@semantic-release/commit-analyzer"

  # Generates the changelog text in the release
  - "@semantic-release/release-notes-generator"
  - "@semantic-release/changelog"

  # Executes a custom command for publishing
  - - "@semantic-release/exec"
    # Builds the project with the new evaluated version (creating an
    # annotated tag with the new version and deleting it in order to not
    # conflict with the other plugins)
    - prepareCmd: |
        git tag -a ${nextRelease.version} -f -m ${nextRelease.version}
        chmod 777 ./gradlew
        ./gradlew build
        git tag -d ${nextRelease.version}
        git pull --tags
      # Publish on Maven and Gradle Portal
      publishCmd: |
        ./gradlew publish --no-parallel
        ./gradlew publishPlugins --no-parallel

  # Publish on GitHub
  - - "@semantic-release/github"
    - assets:
      - path: "build/libs/*.jar"

  # Publish on Docker
  - - '@codedependant/semantic-release-docker'
    - dockerTags: ["latest", "{{version}}"]
      # (e.g. "jahrim/io.github.jahrim.chess.authentication-service")
      dockerImage: "..."
      dockerFile: "Dockerfile"
```

Uno script di [Semantic Release](https://github.com/semantic-release/semantic-release) è diviso
in ulteriori script chiamati _plugin_. Ogni _plugin_ implementa una sequenza standard di _release step_.
Quindi, per ogni _release step_, [Semantic Release](https://github.com/semantic-release/semantic-release)
esegue l'implementazione di tale step fornita dai _plugin_ nell'ordine in cui sono stati definiti.

I _plugin_ utilizzati dallo script di [Semantic Release](https://github.com/semantic-release/semantic-release)
sopra riportato sono i seguenti:
- [@semantic-release/commit-analyzer](https://github.com/semantic-release/commit-analyzer): analizza la storia
  dei commit, eventualmente determinando la nuova versione del software da rilasciare;
- [@semantic-release/release-notes-generator](https://github.com/semantic-release/release-notes-generator):
  genera una lista dei cambiamenti che ha subito il software dall'ultima versione sulla base della sua storia
  dei commit;
- [@semantic-release/changelog](https://github.com/semantic-release/changelog): genera un file che contiene
  la lista dei cambiamenti che ha subito il software dall'ultima versione;
- [@semantic-release/exec](https://github.com/semantic-release//exec): esegue uno script personalizzabile.
  In particolare, è stato utilizzato per generare gli artefatti del modulo e per pubblicarli su dei
  portali non direttamente supportati da _plugin_ ufficiali di [Semantic Release](https://github.com/semantic-release/semantic-release).
- [@semantic-release/github](https://github.com/semantic-release/github): pubblica gli artefatti del modulo
  su GitHub, insieme al tag relativo alla _release_ degli artefatti.
- [@codedependant/semantic-release-docker](https://github.com/esatterwhite/semantic-release-docker): pubblica
  un'immagine [Docker](https://www.docker.com/) su [DockerHub](https://hub.docker.com/), dato il _Dockerfile_
  con cui generarla.

Nell'ordine in cui sono stati registrati i _plugin_, ci si è assicurati che, prima di qualsiasi tipo di
pubblicazione, fossero generati gli artefatti del progetto.

### Continuous Documentation Delivery

Per quanto riguarda la pubblicazione della documentazione dei moduli del sistema, si è fatto affidamento
alle _GitHub Pages_, ovvero dei siti web già integrati con le _repository_ GitHub.

Gli artefatti relativi alle _GitHub Pages_ sono stati generati e pubblicati attraverso un _workflow_ fornito
dal _template repository_ di [just-the-docs](https://just-the-docs.com/), configurato opportunamente per le
necessità del modulo.

## Deployment

In questa sezione, si descriveranno gli strumenti utilizzati per rendere possibile l'esecuzione del sistema
nella sua interezza. Inoltre, si approfondirà come è possibile configurare l'esecuzione del sistema a proprio
piacere.

### Docker Compose

Per eseguire il sistema nella sua interezza, si è fatto affidamento a [Docker Compose](https://docs.docker.com/compose/).
In particolare, è stato definito il seguente _docker-compose_ file:

```yaml
version: "3.4"

services:
  authentication-service:
    image: "jahrim/io.github.jahrim.chess.authentication-service:${AUTHENTICATION_SERVICE_VERSION:-latest}"
    command:
      - --mongodb-connection
      - "${AUTHENTICATION_MONGODB_CONNECTION:?the authentication service won't be able to connect to MongoDB}"
      - --mongodb-database
      - "${AUTHENTICATION_MONGODB_DATABASE:-authentication}"
      - --mongodb-collection
      - "${AUTHENTICATION_MONGODB_COLLECTION:-users}"
      - --http-host
      - "0.0.0.0"
      - --http-port
      - "8080"
      - --allowed-origins
      - "${AUTHENTICATION_ALLOWED_ORIGINS:-*;}"
    ports:
      - "${HOSTNAME:-127.0.0.1}:${AUTHENTICATION_SERVICE_PORT:-8081}:8080"
    restart: always

  statistics-service:
    image: "jahrim/io.github.jahrim.chess.statistics-service:${STATISTICS_SERVICE_VERSION:-latest}"
    command:
      - --mongodb-connection
      - "${STATISTICS_MONGODB_CONNECTION:?the statistics service won't be able to connect to MongoDB}"
      - --mongodb-database
      - "${STATISTICS_MONGODB_DATABASE:-statistics}"
      - --mongodb-collection
      - "${STATISTICS_MONGODB_COLLECTION:-scores}"
      - --http-host
      - "0.0.0.0"
      - --http-port
      - "8080"
      - --allowed-origins
      - "${STATISTICS_ALLOWED_ORIGINS:-*;}"
    ports:
      - "${HOSTNAME:-127.0.0.1}:${STATISTICS_SERVICE_PORT:-8082}:8080"
    restart: always

  chess-game-service:
    depends_on:
      - statistics-service
    image: "jahrim/io.github.jahrim.chess.chess-game-service:${CHESS_GAME_SERVICE_VERSION:-latest}"
    command:
      - --statistics-service
      - "${HOSTNAME:-127.0.0.1}:${STATISTICS_SERVICE_PORT:-8082}"
      - --http-host
      - "0.0.0.0"
      - --http-port
      - "8080"
      - --allowed-origins
      - "${CHESS_GAME_SERVICE_ALLOWED_ORIGINS:-*;}"
    ports:
      - "${HOSTNAME:-127.0.0.1}:${CHESS_GAME_SERVICE_PORT:-8083}:8080"
    restart: always

  frontend:
    depends_on:
      - authentication-service
      - statistics-service
      - chess-game-service
    image: "jahrim/io.github.jahrim.chess.frontend:${FRONTEND_VERSION:-latest}"
    ports:
      - "${HOSTNAME:-127.0.0.1}:${FRONTEND_SERVICE_PORT:-8080}:8080"
    restart: always
    environment:
      VITE_RUNTIME_ENVIRONMENT: >
        {
          "AUTHENTICATION_SERVICE": "${HOSTNAME:-127.0.0.1}:${AUTHENTICATION_SERVICE_PORT:-8081}",
          "STATISTICS_SERVICE": "${HOSTNAME:-127.0.0.1}:${STATISTICS_SERVICE_PORT:-8082}",
          "CHESS_GAME_SERVICE": "${HOSTNAME:-127.0.0.1}:${CHESS_GAME_SERVICE_PORT:-8083}"
        }
```

Come si può notare, il _docker compose_ file, inizializza i servizi del sistema a partire dalle loro
immagini, reperite in locale se disponibili, altrimenti su [DockerHub](https://hub.docker.com/).

Ogni servizio è stato configurato adeguatamente attraverso degli argomenti a linea di comando oppure
delle variabili d'ambiente. Tale configurazione può essere modificata definendo delle variabili d'ambiente
all'interno della propria console, oppure più opportunamente definendo un file _.env_ che le contiene.

### Configurazione

Di seguito, si riporta un esempio di file _.env_, che illustra le variabili d'ambiente che possono essere
specificate per configurare l'esecuzione del sistema.

```properties
### System
# The name of the system (used to name the containers)
COMPOSE_PROJECT_NAME=system
# The hostname where the system is deployed
HOSTNAME=127.0.0.1

### Frontend Service (https://github.com/ldss-project/frontend)
# The version of the frontend service to use
FRONTEND_VERSION=latest
# The port at which the frontend service will be listening
FRONTEND_PORT=8080

### Authentication Service (https://github.com/ldss-project/authentication-service)
# The version of the authentication service to use
AUTHENTICATION_SERVICE_VERSION=latest
# The port at which the authentication service will be listening
AUTHENTICATION_SERVICE_PORT=8081
# The connection to the MongoDB instance where the data of the authentication service will be stored
AUTHENTICATION_MONGODB_CONNECTION=...
# The MongoDB database where the data of the authentication service will be stored
AUTHENTICATION_MONGODB_DATABASE=authentication
# The collection inside the MongoDB database where the data of the authentication service will be stored
AUTHENTICATION_MONGODB_COLLECTION=users
# The origins that are allowed to interact with the authentication service
AUTHENTICATION_ALLOWED_ORIGINS=http://localhost:8080;http://127.0.0.1:8080;http://localhost:5173;http://127.0.0.1:5173;

### Statistics Service (https://github.com/ldss-project/statistics-service)
# The version of the statistics service to use
STATISTICS_SERVICE_VERSION=latest
# The port at which the statistics service will be listening
STATISTICS_SERVICE_PORT=8082
# The connection to the MongoDB instance where the data of the statistics service will be stored
STATISTICS_MONGODB_CONNECTION=...
# The MongoDB database where the data of the statistics service will be stored
STATISTICS_MONGODB_DATABASE=statistics
# The collection inside the MongoDB database where the data of the statistics service will be stored
STATISTICS_MONGODB_COLLECTION=scores
# The origins that are allowed to interact with the statistics service
STATISTICS_ALLOWED_ORIGINS=http://localhost:8080;http://127.0.0.1:8080;http://localhost:5173;http://127.0.0.1:5173;

### Chess Game Service (https://github.com/ldss-project/chess-game-service)
# The version of the chess game service to use
CHESS_GAME_SERVICE_VERSION=latest
# The port at which the chess game service will be listening
CHESS_GAME_SERVICE_PORT=8083
# The origins that are allowed to interact with the chess game service
CHESS_GAME_SERVICE_ALLOWED_ORIGINS=http://localhost:8080;http://127.0.0.1:8080;http://localhost:5173;http://127.0.0.1:5173;
```

Tutte queste configurazioni sono opzionali da specificare, ad eccezione delle stringhe di connessione ai database
[MongoDB](https://www.mongodb.com/) da cui alcuni servizi dipendono. Infatti, queste stringhe contengono alcuni
dati sensibili, come le credenziali per autenticarsi ai database, quindi non sono state rese pubbliche.
Inoltre, potrebbe essere necessario modificare la configurazione relativa alla _CORS Policy_, in base al browser
utilizzato.

### Esecuzione

In sintesi, supponendo di aver installato [Docker](https://www.docker.com/) nella propria macchina,
esistono due modalità per eseguire il sistema:
- **Via release**:
  1. Scaricare i file `docker-compose.yml` e `default.env` disponibili al seguente 
     [link](https://github.com/ldss-project/frontend/releases);
  2. Mettere i file nella stessa directory;
  3. Modificare il file `default.env`, configurando l'esecuzione del sistema a piacere ed
     includendo i segreti necessari alla sua esecuzione;
  4. Rinominare il file `default.env` a `.env`;
  5. Eseguire `docker compose up` all'interno della directory in cui si trovano i due file.
- **Via repository**:
  1. Clonare il [frontend](https://github.com/ldss-project/frontend), in cui sono presenti i
     file `docker-compose.yml` e `.env.public`;
  2. Creare un file `.env` nella root del progetto, configurando l'esecuzione del sistema a piacere ed
     includendo i segreti necessari alla sua esecuzione. Il file `.env.public` può essere utilizzato
     come template per create il file `.env`;
  3. Eseguire `docker compose up` o `npm run deploy` all'interno della root del progetto.

## Licensing

Il software realizzato per questo progetto è soggetto alla licenza [MIT](https://opensource.org/license/mit/),
per cui è sia _libero_, garantendo la possibilità di utilizzare, modificare e ridistribuire il software, che
_open-source_, garantendo l'accesso al codice sorgente del progetto.

La compatibilità della licenza utilizzata rispetto alle licenze dei software da cui questo progetto dipende, è
stata verificata tramite [Fossa](https://fossa.com/), che è uno strumento che monitora le licenze applicate a
delle _repository_ GitHub, verificando che siano compatibili con quelle delle dipendenze esplicitate dal _repository_.

[Back to Top](#top) |
[Previous Chapter](/docs/3-tactical-design)
