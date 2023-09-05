---
title: Tactical Design
layout: default
nav_order: 5
---

# Tactical Design
{: .no_toc}

In questo capitolo si approfondiranno i _bounded context_ individuati in fase di
_Strategic Design_, scegliendo l'architettura che sarà adottata per implementare
il sistema, producendo i modelli dei diversi sottodomini e prendendo alcune
decisioni di maggior dettaglio da cui dipenderà l'implementazione del sistema.

Lo scopo di questo processo è di definire le interfacce attraverso le quali i
_bounded context_ comunicano tra di loro, permettendo successivamente ai membri
del team di occuparsi del design di dettaglio di ciascun componente del sistema
in modo autonomo, purché tali interfacce siano rispettate.

## Contenuti
{: .no_toc}

- TOC
{:toc}

---

## Architettura del sistema

Come prima fase del _Tactical Design_, è stata scelta un'architettura adeguata per
la realizzazione del sistema. In particolare, si è deciso di adottare 
l'_architettura a microservizi_, allo scopo di realizzare un sistema altamente
modulare, facilitando la gestione e la manutenzione dei singoli moduli ed eventualmente
permettendone il riutilizzo in altri contesti di sviluppo in futuro.

### Architettura a microservizi

I microservizi di cui si compone l'architettura del sistema sono i seguenti:
- **Frontend Service**: modella il **Frontend Bounded Context**;
  si occupa di esporre e presentare le funzionalità del sistema agli utenti umani
  dell'applicazione.
- **Authentication Service**: modella l'**Authentication Bounded Context**;
  si occupa della gestione della registrazione e dell'autenticazione degli utenti nel
  sistema.
  La persistenza dei dati degli utenti del servizio è delegata a un
  **Authentication Database**.
- **Statistics Service**: modella lo **Statistics Bounded Context**;
  si occupa della gestione dei punteggi degli utenti nel sistema.
  La persistenza dei punteggi degli utenti del servizio è delegata a uno
  **Statistics Database**.
- **Chess Game Service**: modella il **Game Manager Bounded Context** e il
  **Game Executor Bounded Context**; si occupa della gestione della creazione,
  rimozione e ricerca delle partite di scacchi nel sistema, oltre che della
  connessione dei giocatori a tali partite e del loro svolgimento.

Mentre il **Frontend Service**, l'**Authentication Service** e lo **Statistics Service**
sono dei servizi stateless, il **Chess Game Service** è un servizio stateful, il cui stato
consiste nell'insieme delle partite di scacchi e dello stato in cui si trovano.

![Microservice Architecture](/docs/resources/images/tactical-design/microservices.png)

### Clean Architecture

Per la progettazione dei microservizi del sistema, si è deciso di adottare le
buone pratiche della _Clean Architecture_, anche detta _Architettura Esagonale_.

In dettaglio, ogni servizio è stato scomposto in quattro strati concentrici:
- **Modelli**: lo strato più interno al sistema. Un modello contiene la business
  logic del servizio, tutta o in parte, e dovrebbe essere nascosto agli utenti
  del servizio.
- **Porte**: lo strato al di sopra dei modelli. Una porta espone la logica di
  un modello sotto forma di un insieme di funzionalità relative a uno specifico
  caso d'uso del servizio.
- **Adapters**: lo strato al di sopra delle porte. Un adapter abilita una
  specifica tecnologia per comunicare con una porta del servizio.
- **Proxies**: lo strato al di sopra degli adapter. Un proxy è un componente dalla
  parte dell'utente di un servizio, che espone le funzionalità del servizio secondo
  lo specifico caso d'uso necessario all'utente e minimizza i danni causati da
  eventuali cambiamenti di interfaccia del servizio (fungendo da _Anti-Corruption Layer_).
  Un servizio potrebbe necessitare di un proxy nel caso in cui dipendesse da un altro
  servizio.

Allo scopo di facilitare la creazione di servizi che aderiscono alle buone pratiche
della _Clean Architecture_, si è deciso di sviluppare una libreria che modella questi
concetti, chiamata [HexArc](https://github.com/ldss-project/hexarc). In particolare,
la libreria offre un DSL per definire in modo dichiarativo dei servizi ed eseguirne
il deployment.

## Architettura di dettaglio del sistema

Come seconda fase del _Tactical Design_, si è deciso di approfondire l'architettura
introdotta nella fase precedente, stabilendo le tecnologie utilizzate dai servizi per
comunicare tra di loro e conseguentemente i loro contratti.

### Tecnologie utilizzate

Di seguito, si riporta un diagramma che mostra le tecnologie utilizzate per la comunicazione tra i 
servizi del sistema.

![Detailed Microservice Architecture](/docs/resources/images/tactical-design/detailed-microservices.png)

Come mostrato dal diagramma, tutti i servizi espongono un adapter HTTP per permettere all'utente di
interagirvi tramite browser. Inoltre, il **Chess Game Service** supporta anche una comunicazione
basata su Websocket, ideale per una comunicazione bidirezionale in tempo reale e per poter gestire
la sottoscrizione ad eventuali eventi del servizio.

Infine, per quanto riguarda i database, sono stati scelti dei database _Document-Oriented_, ovvero un
tipo particolare di database _NoSQL_, in cui i dati sono rappresentati come dei documenti, invece che
come tabelle, quindi risultando più flessibili durante la modellazione del dominio. Più in dettaglio,
per il formato dei documenti si è scelto il _BSON (Binary JSON)_.

### Contratti

Di seguito, si riportano i contratti dei servizi del sistema, ovvero le descrizioni delle funzionalità
esposte dai vari servizi e di come accedervi.

#### Authentication Database

User Schema
```yaml
  title: User
  description: A user in the authentication database.
  properties:
    # The unique id used to identify this user in the
    # authentication database.
    _id:
      bsonType: objectId
    # The unique name used to identify this user in the
    # authentication database.
    username:             
      bsonType: string
    # The encrypted password of this user (salt and hash).
    password:             
      bsonType: string
    # The email of this user.
    email:                
      bsonType: string
    # The token assigned to this user, if authenticated.
    token: Token
```

Token Schema
```yaml
  title: Token
  description: A token in the authentication database.
  properties:
    # The unique id used to identify this token in the
    # authentication database.
    id:
      bsonType: string
    # The date of expiration of this token.
    expiration:
      bsonType: date
```

Guest user example
```yaml
{
  "_id": { "$oid": "64ad811808f8e112f8c06521" },
  "username": "ExampleGuestUser",
  "password": "$2a$10$HNRNLhKyjLGCZ3/R6yj0QuenqRSdpRf/J4r0NvVhpNcnRYtSNJoIy",
  "email": "example.user@chessgame.com",
}
```

Authenticated user example
```yaml
{
  "_id": { "$oid": "64ad811808f8e112f8c06521" },
  "username": "ExampleAuthenticatedUser",
  "password": "$2a$10$HNRNLhKyjLGCZ3/R6yj0QuenqRSdpRf/J4r0NvVhpNcnRYtSNJoIy",
  "email": "example.user@chessgame.com",
  "token": {
    "id": "bb9763cc-ac0b-462b-88d0-2b0c66fe0cd1",
    "expiration": { 
      "$date": { "$numberLong": "1689326583861" }
    }
  }
}
```

#### Authentication Service

[REST API](/swagger-apis/authentication-service/latest/rest)

#### Statistics Database

Statistics Schema
```yaml
  title: Statistics
  description: The statistics of a player in the statistics database.
  properties:
    # The unique id used to identify this set of statistics in the 
    # statistics database.
    _id:
      bsonType: objectId
    # The player who owns this statistics. Used to identify this
    # set of statistics in the statistics database.
    username:
      bsonType: string
    # The scores achieved by this player in the order they were
    # achieved.
    latestScores:
      bsonType: array
      items: Score
```

Score Schema
```yaml
  title: Score
  description: The score of a player bound to a specific time.
  properties:
    # The date when this score was achieved by its owner.
    insertion:
      bsonType: date
    # The rank of the owner of this score when it was achieved.
    rank:
      bsonType: long
    # The number of wins of the owner of this score when it was
    # achieved.
    wins:
      bsonType: int
    # The number of losses of the owner of this score when it
    # was achieved.
    losses:
      bsonType: int
    # The proportion between the wins and the losses of the
    # owner of this score when it was achieved.
    ratio:
      bsonType: double
```

Statistics Example
```yaml
{
  "_id": { "$oid": "64adad08b010d1f5ca422a14" },
  "username": "vittoriogiusti",
  "latestScores": [
    {
      "insertion": {
        "$date": { "$numberLong": "1686088800000" }
      },
      "rank": { "$numberLong": "2" },
      "wins": { "$numberInt": "1" },
      "losses": { "$numberInt":"0" },
      "ratio": { "$numberDouble":"1.0" }
    },
    {
      "insertion": {
        "$date": { "$numberLong": "1686434400000" }
      },
      "rank": { "$numberLong": "2" },
      "wins": { "$numberInt": "1" },
      "losses": { "$numberInt" : "1" },
      "ratio": { "$numberDouble" : "1.0" }
    },
  ]
}
```

#### Statistics Service

[REST API](/swagger-apis/statistics-service/latest/rest)

#### Chess Game Service

[REST API](/swagger-apis/chess-game-service/latest/rest)
[Websocket API](/swagger-apis/chess-game-service/latest/async)

#### Frontend Service

---

[Back to Top](#top) |
[Previous Chapter](/docs/2-strategic-design) |
[Next Chapter](/docs/4-devops)
