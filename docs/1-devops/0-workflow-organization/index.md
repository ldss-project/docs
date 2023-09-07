---
title: Workflow Organization
layout: default
parent: DevOps
nav_order: 1
---

# Workflow Organization
{: .no_toc}

In questo capitolo si approfondiranno le buone pratiche di _DevOps_ che sono
state adottate dal team durante lo sviluppo del sistema, definendo un processo
e un ambiente di sviluppo standard per i membri del team.

Lo scopo di questo processo è di definire un'organizzazione del processo di sviluppo
a priori, promuovendo l'autonomia dei membri del team a posteriori.

## Contenuti
{: .no_toc}

- TOC
{:toc}

---

## GitHub Organization

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

## DVCS Workflow

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

## Versioning

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

## GitHub Template Repositories

Per evitare di dover definire la stessa configurazione per ogni _repository_ quando creato, sono stati utilizzati
dei _template repository_.

In particolare, per la documentazione dei moduli è stato applicato un template di terze parti, chiamato
[just-the-docs](https://just-the-docs.com/), mentre per l'inizializzazione di progetti Scala 3 è stato
creato un template interno, chiamato [scala3-project-template](https://github.com/ldss-project/scala3-project-template).

Un altro metodo utilizzato agli stessi scopi, è quello di creare delle _fork_ da progetti pre-configurati. Ad
esempio, questa stessa documentazione è stata creata a partire da una _fork_ della documentazione del progetto
realizzato per il corso di [Project Management 2022-2023](https://www.unibo.it/it/didattica/insegnamenti/insegnamento/2022/412683),
disponibile al seguente [link](https://github.com/jahrim/pm).

---

[Back to Top](#top) |
[To Domain Driven Design](/docs/0-domain-driven-design) |
[Next Chapter](/docs/1-devops/1-build-automation)