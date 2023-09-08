---
title: Domain Analysis
layout: default
parent: Domain Driven Design
nav_order: 2
---

# Domain Analysis
{: .no_toc}

In questo capitolo si descriverà il processo di _Knowledge Crunching_ adoperato
per esplorare il dominio del problema da risolvere, identificando i processi di
cui si compone e le entità coinvolte in tali processi, quindi suddividendo il
dominio in sottodomini e formalizzando la conoscenza acquisita attraverso un
_Ubiquitous Language_.

Gli scopi di questo processo sono di raccogliere conoscenza sul dominio, delineare
delle aree di conoscenza più facilmente controllabili e migliorare la comunicazione
nel team durante le fasi successive del progetto.

## Contenuti
{: .no_toc}

- TOC
{:toc}

---

## Domain Storytelling

Come primo processo della _Domain Analysis_, si è deciso di approfondire i
processi delineati dai [requisiti di massima](/docs/0-domain-driven-design/0-problem#requisiti-di-massima)
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
un limite massimo di tempo per mossa), la `modalità di gioco` (ad esempio, giocatore contro
giocatore o giocatore contro computer), un `identificatore per la partita` e se la partita è
`pubblica` o `privata`.

Dopo aver ricevuto la configurazione, il sistema crea una partita di conseguenza.
Quindi, genera le coordinate attraverso le quali un qualsiasi giocatore può unirsi alla
partita, inoltrandole all'host.

Infine, l'host partecipa alla partita connettendosi alle coordinate ricevute e inviando
al sistema la `squadra` a cui vuole essere assegnato.

![Create game story](/docs/resources/images/domain-storytelling/create-game.png)

### Partecipazione a una partita pubblica

Il processo di `partecipazione a una partita pubblica` prevede prima la creazione di una
partita pubblica da parte di un giocatore host.

Dopodiché, un altro giocatore può richiedere al sistema di gestione della partita
d'individuare una partita pubblica a cui può partecipare. In questo contesto, questo
giocatore assume il ruolo di `guest` della partita.

Quindi, il sistema ricerca una partita pubblica tra quelle disponibili e genera le coordinate
attraverso le quali un qualsiasi giocatore può unirsi a essa, inoltrandole al guest.

Infine, il guest partecipa alla partita connettendosi alle coordinate ricevute e inviando al sistema
la squadra a cui vuole essere assegnato.

![Join public game story](/docs/resources/images/domain-storytelling/join-public-game.png)

### Partecipazione a una partita privata

Il processo di `partecipazione a una partita privata` è simile a quello di partecipazione a
una partita pubblica, ma richiede un'ulteriore interazione tra i due giocatori, in cui l'host
condivide con il guest l'identificatore della partita privata da lui creata.

Quindi, il guest può utilizzare tale identificatore per richiedere al sistema di gestione
delle partite le coordinate per potersi connettere alla partita dell'host.

![Join private game story](/docs/resources/images/domain-storytelling/join-private-game.png)

### Ottenimento dello stato di una partita

Il processo di `ottenimento dello stato di una partita` prevede che un giocatore richieda
al sistema di gestione delle partite lo stato di una partita specifica dato il suo
identificatore.

Dopodiché, il sistema reperisce tale stato inoltrandolo al giocatore.

Nello stato di una partita sono compresi lo `stato del server di gioco` (ovvero lo stato del processo che
esegue la partita) e lo `stato del gioco`, che comprende il `turno corrente` dei giocatori, lo stato
della `scacchiera`, la `situazione sulla scacchiera` (`scacco`, `stallo`, `scacco matto` o `promozione`),
il tempo rimasto a ciascuno dei giocatori, la configurazione della partita, lo `storico
delle mosse` eseguite ed eventualmente il `risultato della partita` nel caso in cui fosse
terminata.

![Retrieve game state story](/docs/resources/images/domain-storytelling/get-state.png)

### Ottenimento delle mosse di un pezzo sulla scacchiera

Il processo di `ottenimento delle mosse di un pezzo sulla scacchiera` prevede che un giocatore
richieda al sistema di gestione delle partite le `mosse` di un `pezzo` sulla scacchiera data la sua
`posizione`. In particolare, una posizione sulla scacchiera è individuata da un `file` (da A a H),
indicante la colonna della `casella`, e da un `rank` (da 1 a 8), indicante la riga della casella.

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
validare la sessione dell'utente, da una parte salvandolo nel sistema di memorizzazione dei
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

## Subdomain analysis

A partire dalle storie individuate durante il _Domain Storytelling_, è stato possibile estrarre i
concetti principali del dominio del problema.

Di seguito, si riporta una proiezione dei concetti del dominio posizionati nello spazio in base alla
loro similarità. Inoltre, in grassetto si riporta la posizione delle storie descritte durante il
_Domain Storytelling_.

![Concepts grouping](/docs/resources/images/domain-analysis/subdomains.png)

Analizzando le similarità e le relazioni tra tali concetti, è stato quindi possibile suddividere il
dominio del problema in diversi sottodomini. In maggiore dettaglio, sono stati individuati tre
macro-domini:

- **Authentication Subdomain**: racchiude tutti i concetti necessari per descrivere il profilo di un utente
  nel sistema, permettendo la registrazione di nuovi utenti, l'autenticazione di utenti già registrati e
  l'autorizzazione a operazioni sensibili, come l'aggiornamento del profilo di un utente. Per quanto riguarda la
  persistenza di tali dati si potrebbe individuare un dominio separato, detto **Authentication Storage Subdomain**.
- **Statistics Subdomain**: racchiude tutti i concetti necessari per descrivere le statistiche di un giocatore
  del sistema, permettendo il salvataggio dei punteggi dei giocatori, il reperimento delle statistiche di un
  giocatore e il reperimento della classifica globale dei giocatori. Per quanto riguarda la persistenza di tali
  statistiche si potrebbe individuare un dominio separato, detto **Statistics Storage Subdomain**.
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

A questi si aggiunge il **Web Application Subdomain**, che gestisce la presentazione del sistema agli utenti finali e
le interazioni tra tali utenti e gli altri sottodomini del problema.

Dopo aver individuato i sottodomini del problema, è stata eseguita un'analisi per classificarli in base alla loro
complessità e al business value da loro generato. Di seguito, si riporta il _Domain Chart_ risultato da tale analisi.

![Domain Chart](/docs/resources/images/domain-analysis/domain-chart.png)

Come si può notare dal _Domain Chart_, i sottodomini sono stati classificati nel modo seguente:
- **Chess Game Subdomain**: dominio di tipo _Decisive Core_, con elevata complessità, dovuta alla grande quantità
  di concetti da modellare, e con elevato business value, costituendo infatti le fondamenta del sistema da realizzare.
- **Statistics Subdomain**: dominio di tipo _Short-Term Core_, con discreta complessità, dovuta alla ridotta quantità
  di concetti da modellare, e con elevato business value, costituendo infatti un elemento di competitività tra i
  giocatori e quindi contribuendo al loro coinvolgimento nell'applicazione.
- **Statistics Storage Subdomain**: dominio di tipo _Supporting_, con bassa complessità, dovuta all'esistenza di
  soluzioni standard per agevolare la sua implementazione, e con discreto business value, dovuto alla dipendenza dello
  **Statistics Subdomain** da questo dominio.
- **Authentication Subdomain**: dominio di tipo _Generic_, con elevata complessità, dovuta alla criticità delle
  operazioni che gestisce, e con basso business value, costituendo infatti solo un componente necessario al
  funzionamento del sistema, senza però contribuire alla sua originalità.
- **Authentication Storage Subdomain**: dominio di tipo _Generic_, con bassa complessità, dovuta all'esistenza di
  soluzioni standard per agevolare la sua implementazione, e con basso business value, costituendo infatti solo un
  componente necessario al funzionamento del sistema, senza però contribuire alla sua originalità.
- **Web Application Subdomain**: dominio di tipo _Decisive Core_, con elevata complessità, dovuta alla gestione delle
  interazioni con gli altri sottodomini, e con elevato business value, costituendo infatti la facciata del sistema e
  quindi contribuendo al coinvolgimento degli utenti nell'applicazione.

## Ubiquitous Language

Come ultima fase della _Domain Analysis_, sono stati formalizzati i concetti identificati nelle fasi
precedenti, precisando il loro effettivo significato e quindi producendo il seguente _Ubiquitous Language_.

| Concetto                                                  | Traduzione                    | Significato                                                                                                                                                                                                                                                                                                                                                                 | Relazioni                                |
|-----------------------------------------------------------|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------|
| Aggiornamento del profilo utente                          | User profile update           | Operazione in cui un utente autenticato aggiorna le informazioni del proprio profilo utente.                                                                                                                                                                                                                                                                                |                                          |
| Autenticazione                                            | Log in                        | Operazione in cui un utente guest diventa utente autenticato tramite l'invio delle proprie credenziali.                                                                                                                                                                                                                                                                     |                                          |
| Credenziali                                               | Credentials                   | Informazioni di un utente necessarie a distinguerlo all'interno del sistema.                                                                                                                                                                                                                                                                                                | Meronimo di profilo utente.              |
| Email                                                     | Email                         | Indirizzo di posta elettronica di un utente.                                                                                                                                                                                                                                                                                                                                | Meronimo di profilo utente.              |
| Nome utente                                               | Username                      | Nome con cui l'utente è riconoscibile all'interno del sistema.                                                                                                                                                                                                                                                                                                              | Meronimo di credenziali.                 |
| Password                                                  | Password                      | Termine segreto conosciuto da un utente, con il quale l'utente può autenticarsi nel sistema.                                                                                                                                                                                                                                                                                | Meronimo di credenziali.                 |
| Profilo utente                                            | User profile                  | Insieme di tutte le informazioni di un utente che il sistema può conoscere.                                                                                                                                                                                                                                                                                                 |                                          |
| Registrazione                                             | Sign in                       | Operazione in cui un utente guest fornisce le proprie informazioni al sistema, rendendosi noto come utilizzatore dell'applicazione.                                                                                                                                                                                                                                         |                                          |
| Sistema di autenticazione                                 | Authentication System         | Sistema che si occupa di gestire la registrazione, l'autenticazione e l'aggiornamento del profilo degli utenti dell'applicazione.                                                                                                                                                                                                                                           |                                          |
| Sistema di memorizzazione dei dati utente                 | Authentication Storage System | Sistema che si occupa di gestire la memorizzazione dei dati degli utenti dell'applicazione.                                                                                                                                                                                                                                                                                 |                                          |
| Token                                                     | Token                         | Termine univoco e segreto consegnato all'utente dopo essersi autenticato, attraverso il quale è possibile dimostrare la propria autenticazione al sistema.                                                                                                                                                                                                                  |                                          |
| Utente                                                    | User                          | Utilizzatore del sistema.                                                                                                                                                                                                                                                                                                                                                   |                                          |
| Utente autenticato                                        | Authenticated user            | Utente in possesso di un token.                                                                                                                                                                                                                                                                                                                                             | Iponimo di utente.                       |
| Utente guest                                              | Guest user                    | Utente non autenticato.                                                                                                                                                                                                                                                                                                                                                     | Iponimo di utente.                       |
|                                                           |                               |                                                                                                                                                                                                                                                                                                                                                                             |                                          |
| Classifica                                                | Leaderboard                   | Graduatoria dei giocatori dell'applicazione, basata sui loro punteggi.                                                                                                                                                                                                                                                                                                      |                                          |
| Ottenimento della classifica                              | Leaderboard retrieval         | Operazione in cui un utente richiede i dati relativi a una porzione della classifica del sistema.                                                                                                                                                                                                                                                                           |                                          |
| Ottenimento delle statistiche di un giocatore             | Statistics retrieval          | Operazione in cui un utente richiede lo storico dei punteggi di un giocatore dell'applicazione (incluso sè stesso).                                                                                                                                                                                                                                                         |                                          |
| Nome giocatore                                            | Player name                   | Nome di un giocatore nell'applicazione.                                                                                                                                                                                                                                                                                                                                     | Meronimo di punteggio.                   |
| Posizione in classifica                                   | Rank                          | Posizione di un giocatore nella classifica del sistema.                                                                                                                                                                                                                                                                                                                     | Meronimo di punteggio.                   |
| Punteggio                                                 | Score                         | Dati che descrivono le performance di un giocatore in un dato momento nel sistema.                                                                                                                                                                                                                                                                                          | Meronimo di statistiche.                 |
| Ratio                                                     | Ratio                         | Quoziente tra il numero di vittorie e di sconfitte di un giocatore.                                                                                                                                                                                                                                                                                                         | Meronimo di punteggio.                   |
| Salvataggio delle statistiche di un giocatore             | Score persistence             | Operazione in cui il nuovo punteggio di un giocatore viene reso noto al sistema.                                                                                                                                                                                                                                                                                            |                                          |
| Sconfitta                                                 | Loss                          | Risultato di un giocatore in cui ha perso a un gioco.                                                                                                                                                                                                                                                                                                                       | Meronimo di punteggio.                   |
| Sistema di gestione delle statistiche dei giocatori       | Statistics Management System  | Sistema che si occupa di gestire l'ottenimento e l'elaborazione dei punteggi dei giocatori del sistema.                                                                                                                                                                                                                                                                     |                                          |
| Sistema di memorizzazione delle statistiche dei giocatori | Statistics Storage System     | Sistema che si occupa di gestire la memorizzazione dei dati degli utenti dell'applicazione.                                                                                                                                                                                                                                                                                 |                                          |
| Statistiche                                               | Statistics                    | Insieme di tutti i punteggi di un giocatore del sistema.                                                                                                                                                                                                                                                                                                                    |                                          |
| Vittoria                                                  | Win                           | Risultato di un giocatore in cui ha vinto a un gioco.                                                                                                                                                                                                                                                                                                                       | Meronimo di punteggio.                   |
|                                                           |                               |                                                                                                                                                                                                                                                                                                                                                                             |                                          |
| Alfiere                                                   | Bishop                        | Pezzo degli scacchi in grado di muoversi di un qualsiasi numero di caselle in una delle direzioni diagonali.                                                                                                                                                                                                                                                                | Iponimo di pezzo.                        |
| Applicazione di una mossa                                 | Move application              | Operazione in cui un giocatore richiede al sistema di interpretare una mossa e modificare lo stato della partita di conseguenza.                                                                                                                                                                                                                                            |                                          |
| Arrocco                                                   | Castling                      | Mossa in cui un re, che ancora non ha effettuato alcuna mossa, può muoversi di due caselle in direzione di una delle due torri, soltanto nel caso in cui anch'esse non abbiano effettuato alcuna mossa e non sia presente alcun altro pezzo nelle celle intermedie tra i due pezzi. A questo punto la torre si muove del numero di caselle necessarie per scavalcare il re. |                                          |
| Cavallo                                                   | Knight                        | Pezzo degli scacchi in grado di muoversi a L (di due caselle in una delle direzioni cardinali e poi di una casella nella direzione ortogonale a quella scelta), con possibilità di scavalcare gli altri pezzi sulla scacchiera.                                                                                                                                             | Iponimo di pezzo.                        |
| Casella                                                   | Cell                          | Porzione della scacchiera occupabile da un pezzo.                                                                                                                                                                                                                                                                                                                           | Meronimo di scacchiera.                  |
| Cattura                                                   | Capture                       | Mossa in cui un pezzo dell'avversario viene rimosso dalla scacchiera.                                                                                                                                                                                                                                                                                                       | Iponimo di mossa.                        |
| Connessione a una partita                                 | Game connection               | Collegamento tra un giocatore e il sistema, persistente per la durata di una partita di scacchi, in cui il giocatore interagisce con la partita fino al suo termine.                                                                                                                                                                                                        |                                          |
| Configurazione di una partita                             | Game configuration            | Impostazione iniziale di una partita di scacchi che ne determina la modalità di svolgimento.                                                                                                                                                                                                                                                                                | Meronimo di stato di una partita.        |
| Creazione di una partita                                  | Game creation                 | Operazione in cui un utente fornisce una configurazione al sistema, allo scopo di creare una nuova partita di conseguenza.                                                                                                                                                                                                                                                  |                                          |
| File                                                      | File                          | Termine che identifica la colonna di una posizione nella scacchiera. I possibili valori sono A, B, C, D, E, F, G, H.                                                                                                                                                                                                                                                        | Meronimo di posizione.                   |
| Giocatore                                                 | Player                        | Utente che sta giocando o ha già giocato a una partita nel sistema.                                                                                                                                                                                                                                                                                                         |                                          |
| Giocatore guest                                           | Guest player                  | Giocatore che sta giocando a una partita che non è stata creata da lui.                                                                                                                                                                                                                                                                                                     | Iponimo di giocatore.                    |
| Giocatore host                                            | Host player                   | Giocatore che sta giocando a una partita da lui creata.                                                                                                                                                                                                                                                                                                                     | Iponimo di giocatore.                    |
| Identificatore di una partita                             | Game identifier               | Termine che identifica univocamente una partita nel sistema.                                                                                                                                                                                                                                                                                                                | Meronimo di configurazione.              |
| Modalità di gioco                                         | Game mode                     | Modalità di svolgimento di una partita di scacchi.                                                                                                                                                                                                                                                                                                                          | Meronimo di configurazione.              |
| Mossa                                                     | Move                          | Azione in una partita di scacchi che sposta un pezzo da una posizione di partenza a una di arrivo, eventualmente rimuovendo un pezzo dell'avversario, secondo le regole degli scacchi.                                                                                                                                                                                      | Meronimo di storico delle mosse.         |
| Partecipazione a una partita                              | Game participation            | Operazione in cui un utente richiede al sistema una connessione verso una partita pubblica qualsiasi o una partita privata di cui conosce l'identificatore.                                                                                                                                                                                                                 |                                          |
| Partita                                                   | Game                          | Gioco in cui due giocatori competono per la vittoria.                                                                                                                                                                                                                                                                                                                       |                                          |
| Partita privata                                           | Private game                  | Partita il cui accesso è limitato a qualunque giocatore ne conosca l'identificatore segreto.                                                                                                                                                                                                                                                                                | Iponimo di partita.                      |
| Partita pubblica                                          | Public game                   | Partita il cui accesso è permesso a qualunque giocatore nel sistema.                                                                                                                                                                                                                                                                                                        | Iponimo di partita.                      |
| Pedone                                                    | Pawn                          | Pezzo degli scacchi in grado di muoversi di una casella in avanti (oppure due nel caso della prima mossa) e di catturare pezzi avversari solo in diagonale.                                                                                                                                                                                                                 | Iponimo di pezzo.                        |
| Pezzo                                                     | Piece                         | Pedina sulla scacchiera, attraverso la quale è possibile giocare a scacchi.                                                                                                                                                                                                                                                                                                 |                                          |
| Posizione sulla scacchiera                                | Position                      | Insieme di coordinate che identificano una casella sulla scacchiera.                                                                                                                                                                                                                                                                                                        |                                          |
| Presa al varco                                            | En-passant                    | Mossa di cattura in cui nel turno precedente un pedone avversario si è mosso di due caselle in avanti, ritrovandosi esattamente accanto al pedone del giocatore di questo turno. Quindi, il pedone del giocatore di questo turno può catturare il pedone avversario come se si fosse mosso di una casella sola.                                                             | Iponimo di mossa.                        |
| Promozione                                                | Promotion                     | Situazione sulla scacchiera in cui un pedone ha raggiunto l'estremo opposto della scacchiera rispetto alla propria posizione di partenza.                                                                                                                                                                                                                                   | Iponimo di situazione di una partita.    |
| Promozione di un pedone                                   | Pawn promotion                | Operazione in cui un giocatore indica al sistema il tipo di pezzo a cui promuovere un pedone in attesa di promozione.                                                                                                                                                                                                                                                       |                                          |
| Ottenimento dello stato di una partita                    | Game state retrieval          | Operazione in cui un giocatore richiede lo stato attuale di una partita.                                                                                                                                                                                                                                                                                                    |                                          |
| Rank                                                      | Rank                          | Termine che identifica la riga di una posizione nella scacchiera. I possibili valori sono 1, 2, 3, 4, 5, 6, 7, 8.                                                                                                                                                                                                                                                           | Meronimo di posizione.                   |
| Re                                                        | King                          | Pezzo degli scacchi in grado di muoversi di una casella in una qualsiasi direzione.                                                                                                                                                                                                                                                                                         | Iponimo di pezzo.                        |
| Regina                                                    | Queen                         | Pezzo degli scacchi in grado di muoversi di un qualsiasi numero di caselle in una qualsiasi direzione.                                                                                                                                                                                                                                                                      | Iponimo di pezzo.                        |
| Regola di movimento                                       | Movement rule                 | Parte del regolamento degli scacchi che indica quali sono le mosse ammissibili per un pezzo in un certa posizione.                                                                                                                                                                                                                                                          |                                          |
| Ricerca di una mossa                                      | Move search                   | Operazione in cui un giocatore indica al sistema la posizione di un pezzo di cui vuole conoscere le mosse ammissibili.                                                                                                                                                                                                                                                      |                                          |
| Risultato di una partita                                  | Game result                   | Descrizione della conclusione di una partita che indica il giocatore vincitore, il giocatore perdente oppure il pareggio tra i giocatori.                                                                                                                                                                                                                                   | Meronimo di stato di una partita.        |
| Scacchiera                                                | Chessboard                    | Tavola sulla quale sono posizionati i pezzi degli scacchi.                                                                                                                                                                                                                                                                                                                  | Meronimo di stato di una partita.        |
| Scacco                                                    | Check                         | Situazione sulla scacchiera in cui il re del giocatore di turno potrebbe essere catturato dal giocatore avversario nel turno successivo.                                                                                                                                                                                                                                    | Iponimo di situazione di una partita.    |
| Scacco matto                                              | Checkmate                     | Situazione sulla scacchiera in cui si verifica sia lo scacco che lo stallo.                                                                                                                                                                                                                                                                                                 | Iponimo di situazione di una partita.    |
| Server di gioco                                           | Game Server                   | Processo che si occupa dell'esecuzione di una partita di scacchi.                                                                                                                                                                                                                                                                                                           |                                          |
| Sistema di gestione delle partite                         | Game Management System        | Sistema che si occupa di gestire la creazione, l'esecuzione e la terminazione delle partite di scacchi.                                                                                                                                                                                                                                                                     |                                          |
| Situazione di una partita                                 | Game Situation                | Particolare configurazione dei pezzi sulla scacchiera.                                                                                                                                                                                                                                                                                                                      | Meronimo di stato di una partita.        |
| Squadra                                                   | Team                          | Colore che distingue i pezzi sulla scacchiera ed i loro proprietari.                                                                                                                                                                                                                                                                                                        |                                          |
| Stallo                                                    | Stale                         | Situazione sulla scacchiera in cui il giocatore di turno non ha mosse disponibili.                                                                                                                                                                                                                                                                                          | Iponimo di situazione di una partita.    |
| Stato del server di gioco                                 | Server state                  | Insieme dei dati che descrivono un server di gioco in un certo instante di tempo.                                                                                                                                                                                                                                                                                           |                                          |
| Stato della partita                                       | Game state                    | Insieme dei dati che descrivono una partita in un certo istante di tempo.                                                                                                                                                                                                                                                                                                   | Meronimo di stato di un server di gioco. |
| Storico delle mosse                                       | Move history                  | Sequenza delle mosse eseguite dall'inizio di una partita dai due giocatori partecipanti.                                                                                                                                                                                                                                                                                    | Meronimo di stato di una partita.        |
| Timeout                                                   | Timeout                       | Situazione di una partita in cui il tempo concesso a uno dei giocatori per effettuare la sua mossa è scaduto.                                                                                                                                                                                                                                                               |                                          |
| Torre                                                     | Rook                          | Pezzo degli scacchi in grado di muoversi di un qualsiasi numero di caselle in una delle direzioni cardinali.                                                                                                                                                                                                                                                                | Iponimo di pezzo.                        |
| Turno                                                     | Turn                          | Parte dello stato di una partita che indica a quale dei due giocatori è concesso di effettuare la prossima mossa.                                                                                                                                                                                                                                                           | Meronimo di stato di una partita.        |
| Vincolo di tempo                                          | Time Constraint               | Modalità di gioco che impone che i giocatori effettuino le proprie mosse entro lo scadere di uno specifico limite di tempo.                                                                                                                                                                                                                                                 | Meronimo di configurazione.              |

Attraverso questo _Ubiquitous Language_, il team di sviluppo potrà descrivere il dominio del problema senza
equivoci, facilitando la comunicazione tra i membri del team.

---

[Back to Top](#top) |
[Previous Chapter](/docs/0-domain-driven-design/0-problem) |
[Next Chapter](/docs/0-domain-driven-design/2-strategic-design)