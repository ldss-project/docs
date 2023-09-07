---
title: Build Automation
layout: default
parent: DevOps
nav_order: 2
---

# Build Automation
{: .no_toc}

In questo capitolo si approfondiranno le tecniche utilizzate per automatizzare
i processi di verifica, di controllo qualità, di generazione e pubblicazione
degli artefatti realizzati per implementare il sistema.

Gli scopi di questo processo sono di automatizzare il più possibile le attività
ripetitive del processo di sviluppo, riducendo il tempo necessario ad ultimare
il progetto, e di migliorare la qualità del software realizzato.

## Contenuti
{: .no_toc}

- TOC
{:toc}

---

## Build Automation Tools

I principali strumenti di _build automation_ utilizzati in questo progetto sono:
- [Gradle](https://gradle.org/): utilizzato per generare e pubblicare gli artefatti dei progetti Scala 3.
- [npm](https://www.npmjs.com/): utilizzato per generare gli artefatti del **Frontend Service** e per
  l'integrazione di [Semantic Release](https://github.com/semantic-release/semantic-release) nei moduli del sistema.

Per quanto riguarda il **Frontend Service**, si eviterà di scendere nel dettaglio poiché gli script utilizzati sono
quelli forniti di default dall'installazione di [Vue + Vite](https://vuejs.org/guide/quick-start.html), solo leggermente
modificati. Pertanto, ci si concentrerà sulla descrizione della _build automation_ con [Gradle](https://gradle.org/).

> _**NOTA**_: siccome ogni modulo del sistema può avere dei requisiti diversi per quanto riguarda la
> _build automation_, per semplicità si riporterà solo l'approccio più comunemente utilizzato.

## Gradle Catalog

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

## Module Information

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

## Artifacts

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
    from(
      configurations.runtimeClasspath.get()
        .filter { it.name.endsWith("jar") }
        .map { zipTree(it) }
    )

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

## Publishing on Maven

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

## Publishing on Gradle Plugin Portal

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

## Docker Build

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

---

[Back to Top](#top) |
[Previous Chapter](/docs/1-devops/0-workflow-organization) |
[Next Chapter](/docs/1-devops/2-continuous-integration)