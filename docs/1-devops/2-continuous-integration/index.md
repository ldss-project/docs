---
title: Continuous Integration
layout: default
parent: DevOps
nav_order: 3
render_with_liquid: false
---

# Continuous Integration
{: .no_toc}

In questo capitolo si approfondiranno le tecniche utilizzate per integrare
lo sviluppo dei moduli del sistema con i processi automatici di testing e di
pubblicazione, in parte definiti nella _build automation_, in modo che ad ogni
incremento di un modulo venga eseguita prima la sua verifica e poi la sua
pubblicazione.

Gli scopi di questo processo sono quelli di promuovere l'incrementalità dello
sviluppo del sistema e di garantire la riproducibilità dei fallimenti e la
robustezza del software.

## Contenuti
{: .no_toc}

- TOC
{:toc}

---

## Continuous Integration Tools

Come strumento principale per la configurazione della _continuous integration_ dei moduli del sistema,
è stato utilizzato [GitHub Actions](https://github.com/features/actions), siccome è già integrato nelle
_repository_ di GitHub.

> _**NOTA**_: dal momento che ogni modulo del sistema può avere dei requisiti diversi per quanto
> riguarda la _continuous integration_, per semplicità, di seguito si riporterà solo l'approccio
> più comunemente utilizzato.

## Continuous Testing

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
**master** è stato propriamente testato.

Un _workflow_ molto simile è stato realizzato anche per il **Frontend Service**, con l'unica differenza che si
basa su [npm](https://www.npmjs.com/) + [Vitest](https://vitest.dev/) invece che su [Gradle](https://gradle.org/) +
[Scalatest](https://www.scalatest.org/).

## Continuous Delivery

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
  ai diversi portali su cui saranno pubblicati gli artefatti del modulo.

Il _workflow_ viene eseguito ad ogni push sul branch **master**, pertanto ogni push sul branch potrebbe
rilasciare una nuova versione del modulo, in base alla storia dei commit a partire dalla sua ultima versione.

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
  genera una lista dei cambiamenti che ha subito il software dall'ultima versione, sulla base della sua storia
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

## Continuous Documentation Delivery

Per quanto riguarda la pubblicazione della documentazione dei moduli del sistema, si è fatto affidamento
alle _GitHub Pages_, ovvero dei siti web già integrati con le _repository_ GitHub.

Gli artefatti relativi alle _GitHub Pages_ sono stati generati e pubblicati attraverso un _workflow_ fornito
dal _template repository_ di [just-the-docs](https://just-the-docs.com/), configurato opportunamente per le
necessità del modulo.

---

[Back to Top](#top) |
[Previous Chapter](/docs/1-devops/1-build-automation) |
[Next Chapter](/docs/1-devops/3-deployment)