---
title: Domain Analysis
layout: default
nav_order: 3
---

# Domain Analysis
{: .no_toc}

In questo capitolo, si descriverà il processo di _Knowledge Crunching_ adoperato
per esplorare il dominio del problema da risolvere, identificando i processi di
cui si compone e le entità coinvolte in tali processi, quindi suddividendo il
dominio in sottodomini e formalizzando la conoscenza acquisita attraverso un
_Ubiquitous Language_.

## Contenuti
{: .no_toc}

- TOC
{:toc}

---

## Domain Storytelling

Come primo processo del _Knowledge Crunching_, si è deciso di approfondire i
processi delineati dai [requisiti di massima](/docs/0-problem#requisiti-di-massima)
attraverso la tecnica del _Domain Storytelling_.

In mancanza di un committente e quindi di un _Domain Expert_ da poter coinvolgere
nel processo, i membri del team si sono dovuti calare nelle parti di _Domain Expert_.
In particolare, sono state organizzate delle riunioni di _Brainstorming_, durante
le quali sono stati analizzati i processi del dominio attraverso la definizione di
storie che coinvolgessero gli utenti del sistema, allo scopo di individuare i
concetti del dominio e le loro relazioni.

Nella definizione delle storie si è sempre fatto riferimento all'_happy path_, 
ovvero al flusso d'interazione di massimo guadagno per l'utente coinvolto, senza
considerare possibili fallimenti nei processi descritti.

> _**NOTA**_: durante la descrizione delle storie si evidenzieranno gli `attori`
> e le `entità` interessati dai processi mano a mano scoperti durante il _Domain
> Storytelling_.

> _**NOTA**_: i diagrammi sono stati realizzati tramite l'applicazione online 
> [Domain Story Modeler](https://egon.io/app/).

### Creazione di una partita

Il processo di `creazione di una partita` prevede che un `giocatore` definisca la
sua `configurazione` e la inoltri a un `sistema di gestione delle partite`. In questo
contesto, il giocatore assume il ruolo di `host` della `partita`.

La configurazione di una partita può specificare alcuni `vincoli temporali` (ad esempio,
un limite massimo di tempo per mossa), un `identificatore per la partita` e se la
partita è `pubblica` o `privata`.

Dopo aver ricevuto la configurazione, il sistema crea una partita di conseguenza.
Quindi, genera un `URL` attraverso il quale un qualsiasi giocatore può unirsi alla
partita, inoltrandolo all'host.

Infine, l'host partecipa alla partita connettendosi all'URL ricevuto.

![Create game story](/docs/resources/images/domain-storytelling/create-game.png)

### Partecipazione a una partita pubblica

Il processo di `partecipazione a una partita pubblica` prevede prima la creazione di una
partita pubblica da parte di un giocatore host.

Dopodiché, un altro giocatore può richiedere al sistema di gestione della partita
d'individuare una partita pubblica a cui può partecipare. In questo contesto, questo
giocatore assume il ruolo di `guest` della partita.

Quindi, il sistema ricerca una partita pubblica tra quelle disponibili e genera un URL
attraverso il quale un qualsiasi giocatore può unirsi a essa, inoltrandolo al guest.

Infine, il guest partecipa alla partita connettendosi all'URL ricevuto.

![Join public game story](/docs/resources/images/domain-storytelling/join-public-game.png)

### Partecipazione a una partita privata

Il processo di `partecipazione a una partita privata` è simile a quello di partecipazione a
una partita pubblica, ma richiede un'ulteriore interazione tra i due giocatori, in cui l'host
condivide con il guest l'identificatore della partita privata da lui creata.

Quindi, il guest può utilizzare tale identificatore per richiedere al sistema di gestione
delle partite un URL per potersi connettere alla partita dell'host.

![Join private game story](/docs/resources/images/domain-storytelling/join-private-game.png)

### Ottenimento dello stato di una partita

Il processo di `ottenimento dello stato di una partita` prevede che un giocatore richieda
al sistema di gestione delle partite lo `stato di una partita` specifica dato il suo
identificatore.

Dopodiché, il sistema reperisce tale stato inoltrandolo al giocatore.

Nello stato di una partita sono compresi il `turno corrente` dei giocatori, lo stato della
`scacchiera`, la `situazione sulla scacchiera` (`scacco`, `stallo`, `scacco matto` o `promozione`),
il tempo rimasto a ciascuno dei giocatori, la configurazione della partita, lo `storico
delle mosse` eseguite ed eventualmente il `risultato della partita` nel caso in cui fosse
terminata.

![Retrieve game state story](/docs/resources/images/domain-storytelling/get-state.png)

### Ottenimento delle mosse di un pezzo sulla scacchiera

Il processo di `ottenimento delle mosse di un pezzo sulla scacchiera` prevede che un giocatore
richieda al sistema di gestione delle partite le `mosse` di un `pezzo` sulla scacchiera data la sua
`posizione`. In particolare, una posizione sulla scacchiera è individuata da un `file` (da A a H),
indicante la colonna della `cella`, e da un `rank` (da 1 a 8), indicante la riga della cella.

Dopo aver ricevuto la richiesta del giocatore, il sistema controlla che il pezzo coinvolto
nell'operazione appartenga a tale giocatore, quindi gli inoltra le mosse disponibili per il
pezzo.

Una mossa negli scacchi consiste nello spostamento di un pezzo da una posizione di partenza a una
di arrivo, con eventuale rimozione di un pezzo dell'avversario (in tal caso, si parla di `cattura`).
Ogni pezzo ha le proprie `regole di movimento`, ma esistono alcune mosse particolari come l'`arrocco`
e la `presa al varco`.

![Find moves story](/docs/resources/images/domain-storytelling/find-moves.png)

### Applicazione di una mossa a un pezzo sulla scacchiera

Il processo di `applicazione di una mossa a un pezzo della scacchiera` prevede che un giocatore
richieda al sistema di gestione delle partite di applicare una mossa a un proprio pezzo sulla
scacchiera.

Dopo aver ricevuto la richiesta del giocatore, il sistema controlla che il pezzo coinvolto
nell'operazione appartenga a tale giocatore, quindi aggiorna la scacchiera (ovvero la posizione
dei pezzi di gioco), il giocatore di turno (ovvero a chi è dato il controllo di eseguire una
mossa) ed eventualmente la situazione sulla scacchiera. A ogni aggiornamento, notifica il
giocatore del nuovo stato della partita.

![Apply move story](/docs/resources/images/domain-storytelling/apply-move.png)

### Promozione di un pedone

Il processo di `promozione di un pedone` è simile a quello di applicazione di una mossa a
un pezzo della scacchiera. In questo caso però, il giocatore richiede al sistema di gestione
delle partite di promuovere il `pedone` a un pezzo a propria scelta (tra `cavallo`, `alfiere`, `torre`
e `regina`). Dopodiché, il sistema controlla l'esistenza di un pedone del giocatore in attesa di
promozione, applicando la promozione e aggiornando lo stato della partita di conseguenza.

![Promote pawn story](/docs/resources/images/domain-storytelling/promote.png)

### Salvataggio del risultato di una partita

Il processo di `salvataggio del risultato di una partita` inizia dal sistema di gestione delle
partite, dopo che ha analizzato una partita determinandone il termine. Il termine di una
partita può essere causato da uno scacco matto, uno stallo o un `timeout`.

Dopo aver notificato i giocatori coinvolti del termine della partita, il sistema la elimina.
Quindi, produce i risultati della partita che descrivono per ognuno dei giocatori se ha vinto,
perso oppure se la partita è finita in parità (ad esempio in caso di stallo).
Tali risultati sono inoltrati al `sistema di gestione delle statistiche dei giocatori`, che 
produce i nuovi `punteggi` in `classifica` per i due giocatori coinvolti nella partita, quindi
salvandoli in un `sistema di memorizzazione delle statistiche dei giocatori`.

Il punteggio di un giocatore può includere il suo `nome`, il numero di `sconfitte` e `vittorie`
conseguite, il `ratio` (ovvero il quoziente tra le vittorie e le sconfitte), la `posizione
in classifica` e la data d'inserimento.

![Send game result story](/docs/resources/images/domain-storytelling/send-game-result.png)

### Registrazione di un nuovo utente

Il processo di `registrazione di un nuovo utente` al sistema richiede che un `utente non autenticato`
prepari le informazioni con le quali vuole registrarsi all'applicazione, inviandole al `sistema di
autenticazione`. In questo contesto, l'utente non autenticato assume il ruolo di `utente guest`
rispetto al sistema.

Le `informazioni di un utente` possono includere il suo `nome utente`, la sua `email` e la sua
`password`.

Una volta ricevute le informazioni dell'utente guest, il sistema di autenticazione le codifica
salvandole in un `sistema di memorizzazione dei dati degli utenti`. Quindi, genera un `token` per
validare la sessione dell'utente, da una parte memorizzandolo nel sistema di memorizzazione dei
dati degli utenti, dall'altra inoltrandolo all'utente guest.

Alla ricezione del proprio token, l'utente guest assume il ruolo di `utente autenticato` al sistema.

![Sign-in story](/docs/resources/images/domain-storytelling/sign-in.png)

### Autenticazione di un utente

Il processo di `autenticazione di un utente` al sistema è simile a quello di registrazione di un
nuovo utente, ma richiede che un utente già registrato prepari le proprie `credenziali` inviandole
al sistema di autenticazione.

Diversamente dalle informazioni di un utente, le sue credenziali contengono solo i dati
necessari per essere riconosciuti all'interno del sistema, come nome utente e password.

![Log-in story](/docs/resources/images/domain-storytelling/log-in.png)

### Aggiornamento del profilo di un utente

Il processo di `aggiornamento del profilo di un utente` richiede che un utente autenticato invii al
sistema di autenticazione le informazioni aggiornate del suo `profilo` (ad esempio una nuova password),
oltre al token corretto necessario per abilitare tale operazione sensibile sui dati di quell'utente.

Alla ricezione dei dati, il sistema di autenticazione verifica la validità del token rispetto a quello
salvato precedentemente nel sistema di memorizzazione dei dati degli utenti. Quindi, salva le nuove
informazioni dell'utente nel sistema.

Infine, il sistema di autenticazione notifica l'utente del successo dell'operazione.

![Update password story](/docs/resources/images/domain-storytelling/update-password.png)

### Ottenimento della classifica globale dei giocatori

Il processo di `ottenimento della classifica globale` dei giocatori richiede che un utente invii al
sistema di gestione delle statistiche dei giocatori gli estremi della porzione della classifica a
cui è interessato.

Dopodiché, il sistema estrae tale porzione della classifica dal sistema di memorizzazione delle
statistiche dei giocatori, inoltrandola all'utente.

![Get leaderboard story](/docs/resources/images/domain-storytelling/get-leaderboard.png)

### Ottenimento dello storico dei punteggi di un giocatore

Il processo di `ottenimento dello storico dei punteggi di un giocatore` richiede che un utente invii
al sistema di gestione delle statistiche dei giocatori il nome dell'utente di cui vuole conoscere
le `statistiche`.

Dopodiché, il sistema estrae lo storico dei punteggi del giocatore con quel nome utente dal sistema
di memorizzazione delle statistiche dei giocatori, inoltrandolo all'utente.

![Get score history story](/docs/resources/images/domain-storytelling/get-score-history.png)

## Domain analysis

A partire dalle storie individuate durante il _Domain Storytelling_, è stato possibile estrarre i
concetti principali del dominio del problema.

Di seguito, si riporta una proiezione dei concetti del dominio posizionati nello spazio in base alla
loro similarità. Inoltre, in grassetto si riporta la posizione delle storie descritte durante il
_Domain Storytelling_.

![Concepts grouping](/docs/resources/images/domain-analysis/subdomains.png)

Analizzando le similarità e le relazioni tra tali concetti, è stato quindi possibile suddividere il
dominio del problema in diversi sottodomini. In maggiore dettaglio, sono stati individuati tre
macro-domini:

- **Chess Game Subdomain**: racchiude tutti i concetti necessari per descrivere una partita di scacchi e le
  possibili interazioni con essa. In particolare, questi concetti possono essere raggruppati ulteriormente
  in altri sottodomini:
  - **Game Configuration Subdomain**: racchiude tutti i concetti necessari per descrivere la configurazione
    di una partita di scacchi, permettendo la creazione, la partecipazione e la connessione a una partita di
    scacchi.
  - **Chessboard Subdomain**: racchiude tutti i concetti necessari per descrivere la scacchiera in una partita
    di scacchi, contribuendo al reperimento dello stato di una partita.
  - **Game Situation Subdomain**: racchiude tutti i concetti necessari per descrivere la situazione attuale
    in una partita di scacchi, permettendo la promozione di un pedone e la terminazione di una partita.
  - **Move Subdomain**: racchiude tutti i concetti necessari per descrivere le mosse disponibili per un pezzo
    sulla scacchiera, permettendo la ricerca e l'applicazione delle sue mosse.
- **Statistics Subdomain**: racchiude tutti i concetti necessari per descrivere le statistiche di un giocatore
  del sistema, permettendo il salvataggio dei punteggi dei giocatori, il reperimento delle statistiche di un
  giocatore e il reperimento della classifica globale dei giocatori. La gestione della memorizzazione di tali
  statistiche può essere affidata a un dominio separato, detto **Statistics Storage Subdomain**;
- **Authentication Subdomain**: racchiude tutti i concetti necessari per descrivere il profilo di un utente
  nel sistema, permettendo la registrazione di nuovi utenti, l'autenticazione di utenti già registrati e 
  l'autorizzazione a operazioni sensibili, come l'aggiornamento del profilo di un utente. La gestione della
  memorizzazione di tali dati può essere affidata a un dominio separato, detto **Authentication Storage Subdomain**.

Dopo aver individuato i sottodomini del problema, è stata eseguita un'analisi per classificarli in base alla loro
complessità e al business value da loro generato. Di seguito, si riporta il _Domain Chart_ risultato da tale analisi.

![Domain Chart](/docs/resources/images/domain-analysis/domain-chart.png)

## Ubiquitous Language

---

[Back to Top](#top) |
[Previous Chapter](/docs/0-problem) |
[Next Chapter](/docs/2-strategic-design)