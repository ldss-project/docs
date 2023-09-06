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

Durante lo sviluppo del sistema, si è cercato il più possibile di suddividere il lavoro tra i membri del team
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
- Altrimenti, se la versione non viene modificata.

In questo progetto, non si è mai fatto uso di _Breaking Change_ esplicitamente, infatti i moduli realizzati
sono ancora in una versione instabile, per cui ogni modifica del software potrebbe comportare delle modifiche
sostanziali alla sua API.

### GitHub Template Repositories

Per evitare di dover definire la stessa configurazione per ogni _repository_ quando creato, sono stati utilizzati
dei _Template Repository_.

In particolare, per la documentazione dei moduli, inclusa questa documentazione, è stato applicato un template di
terze parti, chiamato [just-the-docs](https://just-the-docs.com/), mentre per l'inizializzazione di progetti Scala
3 è stato create un template interno, chiamato [scala3-project-template](https://github.com/ldss-project/scala3-project-template).

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
- [Gradle](https://gradle.org/): utilizzato per generare e pubblicare gli artefatti dei progetti Scala 3;
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

### Project Information

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

## Quality Assurance

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
  [Scala Formatter](https://scalameta.org/scalafmt/) in modo che esegui ogni volta che il
  codice del modulo viene compilato e non solo durante la fase di _check_.
  Così facendo, la formattazione viene applicata anche quando il codice viene eseguito o testato
  all'interno di [IntelliJ Idea](https://www.jetbrains.com/idea/).
- [wartremover-gradle-plugin](https://github.com/ldss-project/wartremover-gradle-plugin): plugin
  che permette la configurazione di [WartRemover](https://www.wartremover.org/) come _code linter_
  per un progetto Scala 3.

  Il plugin era stato realizzato come plugin privato all'interno del progetto per il corso di
  [Paradigmi di Programmazione e Sviluppo 2022-2023](https://www.unibo.it/it/didattica/insegnamenti/insegnamento/2022/412597)
  (reperibile al seguente [link](https://github.com/jahrim/PPS-22-chess)), dal quale è stato estratto, quindi 
  refattorizzato, testato e pubblicato su [Gradle Plugin Portal](https://plugins.gradle.org/).
  Il nuovo _repository_ del plugin è disponibile al seguente [link](https://github.com/ldss-project/wartremover-gradle-plugin).
- [kotlin-qa](https://plugins.gradle.org/plugin/org.danilopianini.gradle-kotlin-qa): plugin che
  permette la configurazione di [Detekt](https://github.com/detekt/detekt) e [Ktlint](https://github.com/pinterest/ktlint)
  per un progetto Kotlin.
- [create-vue](https://github.com/vuejs/create-vue): lo script ufficiale per eseguire il _Project Scaffolding_
  per un progetto Vue, che include la configurazione di [ESLint](https://eslint.org/) e [Prettier](https://prettier.io/).

### Artifacts

Per la generazione degli artefatti di ogni modulo, sono state predisposte tre task principali:
- **sourcesJar**: produce il jar contenente i sorgenti del modulo;
- **javadocJar**: produce il jar contenente la documentazione del modulo;
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
forma di [project properties](https://docs.gradle.org/current/userguide/build_environment.html#sec:project_properties).

### Publishing on Gradle Plugin Portal

Per la pubblicazione dei plugin su [Gradle Plugin Portal](https://plugins.gradle.org/) è stato
utilizzato il plugin [Gradle Publishing Plugin](https://plugins.gradle.org/plugin/com.gradle.plugin-publish).

Di seguito, si riporta la configurazione comunemente utilizzata per pubblicare i plugin su
[Gradle Plugin Portal](https://plugins.gradle.org/).

```kotlin
gradlePlugin {
    website.set(ProjectInfo.website)
    vcsUrl.set("${ProjectInfo.website}.git")
    plugins {
        create("wartremover") {
            id = "${ProjectInfo.artifactGroup}.${ProjectInfo.artifactId}"
            displayName = ProjectInfo.longName
            description = ProjectInfo.description
            tags.set(ProjectInfo.tags)
            implementationClass = ProjectInfo.implementationClass
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
in modo da poter accettare delle [project properties](https://docs.gradle.org/current/userguide/build_environment.html#sec:project_properties)
compatibili per essere specificate come variabili d'ambiente. In questo modo, sarà più facile utilizzare il
plugin all'interno della _CI_.

## Continuous Integration

TODO

Sul branch **master** è stata applicata una _branch protection policy_ per richiedere l'integrazione degli aggiornamenti
tramite pull request, in modo da proteggere il branch da modifiche

## Deployment

## Licensing

---

[Back to Top](#top) |
[Previous Chapter](/docs/3-tactical-design)
