---
title: Tactical Design
layout: default
parent: Domain Driven Design
nav_order: 4
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
- **Frontend Service**: modella il **Frontend BC**;
  si occupa di esporre e presentare le funzionalità del sistema agli utenti umani
  dell'applicazione.
- **Authentication Service**: modella l'**Authentication BC**;
  si occupa della gestione della registrazione e dell'autenticazione degli utenti nel
  sistema.
  La persistenza dei dati degli utenti del servizio è delegata a un
  **Authentication Database**.
- **Statistics Service**: modella lo **Statistics BC**;
  si occupa della gestione dei punteggi degli utenti nel sistema.
  La persistenza dei punteggi degli utenti del servizio è delegata a uno
  **Statistics Database**.
- **Chess Game Service**: modella il **Game Manager BC** e il
  **Game Executor BC**; si occupa della gestione della creazione,
  rimozione e ricerca delle partite di scacchi nel sistema, oltre che della
  connessione dei giocatori a tali partite e del loro svolgimento.

Mentre il **Frontend Service**, l'**Authentication Service** e lo **Statistics Service**
sono dei servizi _stateless_, il **Chess Game Service** è un servizio _stateful_, il cui stato
consiste nell'insieme delle partite di scacchi e dello stato in cui si trovano.

![Microservice Architecture](/docs/resources/images/tactical-design/microservices.png)

### Clean Architecture

Per la progettazione dei microservizi del sistema, si è deciso di adottare le
buone pratiche della _Clean Architecture_, detta anche _Hexagonal Architecture_.

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

Il contratto dell'**Authentication Database** descrive la struttura dei dati contenuti nel database,
ovvero la rappresentazione di un utente guest e di un utente autenticato nel sistema. Sulla base di
questa struttura, altri servizi potranno definire delle query da richiedere al database.

Un utente nel sistema può essere modellato come un'_entità_ descritta dai seguenti attributi:
- **Username**: un _value object_ che modella il nome dell'utente, usato come un identificatore
  human-friendly all'interno del sistema;
- **Password**: un _value object_ che modella la password dell'utente, utilizzata per permettere
  all'utente di autenticarsi all'applicazione;
- **Email**: un _value object_ che modella l'email dell'utente, utilizzata come un esempio di dato
  che potrebbe appartenere al profilo di un utente;
- **Token**: un'_entità_ che modella il token dell'utente. Se presente, indica che l'utente è
  autenticato all'applicazione.

Di seguito, si riporta lo schema relativo a un utente nel database.

```yaml
title: User
description: A user in the authentication database.
properties:
  # The unique machine-friendly id used to identify this user
  # in the authentication database.
  _id:
    bsonType: objectId
  # The unique human-friendly id used to identify this user
  # in the authentication database.
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

Un token nel sistema può essere modellato come un'_entità_, descritta da un _value object_
rappresentante la data di scadenza del token.

Di seguito, si riporta lo schema relativo a un token nel database.

```yaml
title: Token
description: A token in the authentication database.
properties:
  # The unique machine-friendly id used to identify this token in the
  # authentication database.
  id:
    bsonType: string
  # The date of expiration of this token.
  expiration:
    bsonType: date
```

Ad esempio, un utente guest nel sistema può essere rappresentato dal seguente _JSON_:
```yaml
{
  "_id": { "$oid": "64ad811808f8e112f8c06521" },
  "username": "ExampleGuestUser",
  "password": "$2a$10$HNRNLhKyjLGCZ3/R6yj0QuenqRSdpRf/J4r0NvVhpNcnRYtSNJoIy",
  "email": "example.user@chessgame.com",
}
```

Mentre, un utente autenticato nel sistema può essere rappresentato dal seguente _JSON_:
```yaml
{
  "_id": { "$oid": "64ad811808f8e112f8c06521" },
  "username": "ExampleAuthenticatedUser",
  "password": "$2a$10$HNRNLhKyjLGCZ3/R6yj0QuenqRSdpRf/J4r0NvVhpNcnRYtSNJoIy",
  "email": "example.user@chessgame.com",
  "token": {
    "id": "bb9763cc-ac0b-462b-88d0-2b0c66fe0cd1",
    "expiration": { "$date": { "$numberLong": "1689326583861" } }
  }
}
```

#### Authentication Service

Il contratto dell'**Authentication Service** espone le funzionalità relative alla registrazione e
all'autenticazione degli utenti nel sistema.

In particolare, le funzionalità esposte dal servizio sono le seguenti:
- **Sign in**: permette a un utente guest di registrarsi all'applicazione, rendendosi noto come
  utilizzatore del sistema e diventando un utente autenticato;
- **Log in**: permette a un utente guest di autenticarsi all'applicazione, identificandosi come
  uno tra gli utilizzatori noti al sistema e diventando un utente autenticato;
- **Log out**: permette a un utente autenticato di revocare la propria autenticazione all'applicazione,
  diventando un utente guest;
- **Get profile**: permette a un utente autenticato di ottenere le informazioni riguardanti il proprio
  profilo utente;
- **Update password**: permette a un utente autenticato di modificare la propria password.

Il contratto di questo servizio è un contratto *REST*, le cui specifiche sono disponibili al
seguente [link](/swagger-apis/authentication-service/latest/rest). All'interno delle specifiche
sono descritti in maggior dettaglio le modalità d'interazione con il servizio e i modelli dei dati
scambiati durante tali interazioni.

> _**NOTA**_: i contratti _REST_ del sistema sono stati definiti seguendo le specifiche di
> [OpenAPI](https://spec.openapis.org/oas/v3.0.0).

#### Statistics Database

Il contratto dello **Statistics Database** descrive la struttura dei dati contenuti nel database,
ovvero la rappresentazione del punteggio e delle statistiche di un giocatore. Sulla base di
questa struttura, altri servizi potranno definire delle query da richiedere al database.

Le statistiche di un giocatore possono essere modellate come un'_entità_ descritta dai seguenti
attributi:
- **Username**: un _value object_ che modella il nome del giocatore a cui appartengono le statistiche,
  usato come un identificatore human-friendly all'interno del sistema;
- **Latest Scores**: una sequenza di _value object_ che modellano i punteggi che il giocatore ha ottenuto
  nel sistema, nell'ordine in cui sono stati ottenuti.

Di seguito, si riporta lo schema relativo alle statistiche di un giocatore nel database.

```yaml
title: Statistics
description: The statistics of a player in the statistics database.
properties:
  # The unique machine-friendly id used to identify this set of
  # statistics in the statistics database.
  _id:
    bsonType: objectId
  # The player who owns this statistics. Used as a human-friendly
  # id of this set of statistics in the statistics database.
  username:
    bsonType: string
  # The scores achieved by this player in the order they were
  # achieved.
  latestScores:
    bsonType: array
    items: Score
```

Il punteggio di un giocatore nel sistema può essere modellato come un _value object_ con i seguenti attributi:
- **Insertion Date**: un _value object_ che modella la data di inserimento del punteggio nelle
  statistiche del giocatore;
- **Rank**: un _value object_ che modella la posizione in classifica del giocatore nel momento in
  cui questo punteggio è stato inserito;
- **Wins**: un _value object_ che modella il numero di vittorie del giocatore nel momento in cui
  questo punteggio è stato inserito;
- **Losses**: un _value object_ che modella il numero di sconfitte del giocatore nel momento in cui
  questo punteggio è stato inserito;
- **Ratio**: un _value object_ che modella il quoziente tra le vittorie e le sconfitte del giocatore
  nel momento in cui questo punteggio è stato inserito.

Di seguito, si riporta lo schema relativo a un punteggio nel database.

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

Ad esempio, le statistiche di un giocatore nel sistema possono essere rappresentate dal seguente _JSON_:

```yaml
{
  "_id": { "$oid": "64adad08b010d1f5ca422a14" },
  "username": "vittoriogiusti",
  "latestScores": [
    {
      "insertion": { "$date": { "$numberLong": "1686088800000" } },
      "rank": { "$numberLong": "2" },
      "wins": { "$numberInt": "1" },
      "losses": { "$numberInt":"0" },
      "ratio": { "$numberDouble":"1.0" }
    },
    {
      "insertion": { "$date": { "$numberLong": "1686434400000" } },
      "rank": { "$numberLong": "2" },
      "wins": { "$numberInt": "1" },
      "losses": { "$numberInt" : "1" },
      "ratio": { "$numberDouble" : "1.0" }
    },
  ]
}
```

#### Statistics Service

Il contratto dello **Statistics Service** espone le funzionalità relative all'inserimento dei
punteggi dei giocatori nel sistema e al reperimento della classifica globale dei giocatori.

In particolare, le funzionalità esposte dal servizio sono le seguenti:
- **Get Latest Score**: permette a un utente di ottenere l'ultimo punteggio di un giocatore nel
  sistema;
- **Get Statistics**: permette a un utente di ottenere tutti i punteggi di un giocatore nel sistema;
- **Get Leaderboard**: permette a un utente di ottenere la classifica globale dei giocatori nel sistema;
- **Add Score**: permette a un utente di aggiungere un nuovo punteggio alle statistiche di un giocatore
  del sistema;
- **Delete Statistics**: permette a un utente di eliminare le statistiche di un giocatore nel sistema.

Il contratto di questo servizio è un contratto *REST*, le cui specifiche sono disponibili al
seguente [link](/swagger-apis/statistics-service/latest/rest). All'interno delle specifiche
sono descritti in maggior dettaglio le modalità d'interazione con il servizio e i modelli dei dati
scambiati durante tali interazioni.

#### Chess Game Service

Il **Chess Game Service** prevede i due contratti seguenti:
- **Game Management API**: espone le funzionalità relative alla creazione e ricerca delle
  partite di scacchi nel sistema, oltre che della connessione dei giocatori a tali partite;
- **Game Execution API**: espone le funzionalità relative allo svolgimento di una partita
  di scacchi.

In particolare, la **Game Management API** espone le seguenti funzionalità:
- **Create Game**: permette a un utente di configurare e creare una partita di scacchi
  nel sistema. Quindi, fornisce all'utente le coordinate per connettersi a tale partita.
- **Find Private Game**: permette a un utente di ricercare una partita privata nel sistema,
  conoscendone l'identificatore. Quindi, fornisce all'utente le coordinate per connettersi a
  tale partita.
- **Find Public Game**: permette a un utente di ricercare una partita pubblica qualsiasi nel
  sistema. Quindi, fornisce all'utente le coordinate per connettersi a tale partita.

La **Game Management API** è un contratto di tipo _REST_, le cui funzionalità sono disponibili al
seguente [link](/swagger-apis/chess-game-service/latest/rest). All'interno delle specifiche
sono descritti in maggior dettaglio le modalità d'interazione con il servizio e i modelli dei dati
scambiati durante tali interazioni.

Per quanto riguarda la **Game Execution API**, il contratto espone le seguenti funzionalità:
- **Get State**: permette a un giocatore di ottenere lo stato del server che esegue la partita
  di scacchi, incluso lo stato della partita stessa.
- **Join Game**: permette a un giocatore di partecipare alla partita, specificando il proprio
  nome e la squadra vuole essere assegnato.
- **Find Moves**: permette a un giocatore di ottenere le mosse disponibili per un pezzo in una
  specifica posizione sulla scacchiera.
- **Apply Move**: permette a un giocatore di applicare una mossa a un pezzo sulla scacchiera.
- **Promote**: permette a un giocatore di promuovere il pedone che è attualmente in attesa di
  essere promosso.

In questo contratto, il servizio reagisce alle richieste dell'utente in due modalità:
- _Request-Response_: a ogni richiesta, il servizio produce una risposta corrispondente;
- _Domain Events_: a ogni richiesta, il servizio può propagare degli eventi specifici, in
  seguito a cambiamenti dello stato del server.

La **Game Execution API** è un contratto di tipo _Websocket_, le cui funzionalità sono disponibili al
seguente [link](/swagger-apis/chess-game-service/latest/async). All'interno delle specifiche
sono descritti in maggior dettaglio le modalità d'interazione con il servizio, i modelli dei dati
scambiati durante tali interazioni e gli eventi appartenenti al dominio del servizio.

> _**NOTA**_: i contratti _Websocket_ del sistema sono stati definiti seguendo le specifiche di
> [AsyncAPI](https://www.asyncapi.com/docs/reference/specification/v2.6.0).

#### Frontend Service

Il contratto del **Frontend Service** espone le funzionalità relative all'esposizione e 
presentazione delle funzionalità degli altri servizi del sistema agli utenti umani
dell'applicazione.

Il contratto prevede semplicemente la possibilità di connettersi al dominio del servizio
tramite browser per poter accedere a un'applicazione web. Tale applicazione gestisce la
presentazione del sistema e la comunicazione tra utenti umani e gli altri servizi del sistema.

### Dettagli aggiuntivi

Il design e l'implementazione di ogni servizio possono essere approfonditi nelle rispettive
pagine web:
- [Authentication Service](https://ldss-project.github.io/authentication-service)
- [Statistics Service](https://ldss-project.github.io/statistics-service)
- [Chess Game Service](https://ldss-project.github.io/chess-game-service)
- [Frontend Service](https://ldss-project.github.io/frontend)

---

[Back to Top](#top) |
[Previous Chapter](/docs/0-domain-driven-design/2-strategic-design) |
[To DevOps](/docs/1-devops)
