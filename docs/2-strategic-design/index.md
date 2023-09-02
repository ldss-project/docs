---
title: Strategic Design
layout: default
nav_order: 4
---

# Strategic Design
{: .no_toc}

In questo capitolo si approfondiranno i domini individuati durante la _Domain Analysis_
dal punto di vista del sistema informatico che dovrà essere realizzato, identificando i
suoi _bounded context_ e le relazioni tra di essi.

Gli scopi di questo processo sono di giustificare il sistema informatico da realizzare alla
luce dei sottodomini del problema, suddividere il lavoro efficientemente nel team di sviluppo,
minimizzando l'overhead dovuto alla necessità di coordinazione, e delineare le dipendenze tra
le diverse aree di sviluppo identificate, facilitandone l'integrazione successivamente.

## Contenuti
{: .no_toc}

- TOC
{:toc}

---

## Bounded context

Come prima fase dello _Strategic Design_, sono stati identificati i _bounded context_ del
sistema da realizzare, a partire dai sottodomini individuati durante la _Domain Analysis_,
suddividendo il sistema in moduli ad alto livello il più possibile indipendenti tra di loro.

Di seguito, si riportano i _bounded context_ individuati, ognuno descritto da un proprio
[Bounded Context Canvas](https://github.com/ddd-crew/bounded-context-canvas).

> _**NOTA**_: i diagrammi sono stati realizzati tramite l'applicazione online
> [Miro](https://miro.com).

### Authentication

L'**Authentication Bounded Context** è il _bounded context_ corrispondente all'**Authentication Subdomain**
e si occupa della gestione della registrazione e dell'autenticazione degli utenti nel sistema.

In particolare, viene utilizzato dal **Frontend Bounded Context** e utilizza l'**Authentication Storage Bounded Context**.

![Authentication bounded context](/docs/resources/images/bounded-contexts/authentication-bounded-context.jpg)

### Authentication Storage

L'**Authentication Storage Bounded Context** è il _bounded context_ corrispondente all'**Authentication Storage Subdomain**
e si occupa della gestione della persistenza dei dati degli utenti nel sistema.

In particolare, viene utilizzato dall'**Authentication Bounded Context**.

![Authentication storage bounded context](/docs/resources/images/bounded-contexts/authentication-storage-bounded-context.jpg)

### Statistics

Lo **Statistics Bounded Context** è il _bounded context_ corrispondente allo **Statistics Subdomain**
e si occupa della gestione dei punteggi dei giocatori e della classifica globale del sistema.

In particolare, viene utilizzato dal **Frontend Bounded Context** e dal **Game Manager Bounded Context** e utilizza lo
**Statistics Storage Bounded Context**.

![Statistics bounded context](/docs/resources/images/bounded-contexts/statistics-bounded-context.jpg)

### Statistics Storage

Lo **Statistics Storage Bounded Context** è il _bounded context_ corrispondente allo **Statistics Storage Subdomain**
e si occupa della gestione della persistenza dei punteggi dei giocatori del sistema.

In particolare, viene utilizzato dallo **Statistics Bounded Context**.

![Statistics storage bounded context](/docs/resources/images/bounded-contexts/statistics-storage-bounded-context.jpg)

### Game Executor

Il **Game Executor Bounded Context** è un _bounded context_ appartenente al **Chess Game Subdomain**
e si occupa della gestione dell'esecuzione di una partita di scacchi nel sistema.

In particolare, viene utilizzato dal **Game Manager Bounded Context** e si basa sui componenti dell'[engine
degli scacchi legacy](https://github.com/jahrim/PPS-22-chess), realizzato per il progetto di
Paradigmi di Programmazione e Sviluppo.

![Game executor bounded context](/docs/resources/images/bounded-contexts/game-executor-bounded-context.jpg)

### Game Manager

Il **Game Manager Bounded Context** è un _bounded context_ appartenente al **Chess Game Subdomain**
e si occupa della gestione della creazione, rimozione e ricerca delle partite di scacchi
nel sistema, oltre che della connessione dei giocatori a tali partite.

In particolare, viene utilizzato dal **Frontend Bounded Context** e utilizza lo **Statistics Bounded Context** e il
**Game Executor Bounded Context**.

![Game manager bounded context](/docs/resources/images/bounded-contexts/game-manager-bounded-context.jpg)

### Frontend

Il **Frontend Bounded Context** è un _bounded context_ appartenente al **Web Application Subdomain**
e si occupa della presentazione del sistema e della comunicazione tra gli utenti umani
del sistema e il sistema stesso.

In particolare, viene utilizzato dagli utenti umani del sistema e utilizza l'**Authentication Bounded Context**,
lo **Statistics Bounded Context** e il **Game Manager Bounded Context**.

![Frontend bounded context](/docs/resources/images/bounded-contexts/frontend-bounded-context.jpg)

## Context Map

Dopo aver individuato i _bounded context_ del sistema, sono state analizzate le relazioni e le
dipendenze tra di essi, producendo una _context map_ per rappresentarle graficamente.

![Context Map](/docs/resources/images/context-map/context-map.png)

Come mostrato dalla _context map_, per le interazioni verso _bounded context_ appartenenti a domini 
core diversi dal proprio, si é deciso di proteggere il consumatore attraverso un _Anti-Corruption Layer_,
in modo da minimizzare i danni al consumatore nel caso in cui l'interfaccia dei fornitori cambiasse.
Infatti, per i domini core si prevede un'evoluzione continua e quindi un'interfaccia molto dinamica.

Tale _Anti-Corruption Layer_ non è stato adottato invece per i _bounded context_ appartenenti allo stesso
dominio, per i quali si è deciso di risparmiare delle risorse in termini di tempo, considerando che il loro
sviluppo potrebbe essere assegnato allo stesso membro del team, in modo da ridurre la complessità della
coordinazione tra gli eventuali collaboratori dovuta a relazioni di tipo supplier-consumer conformiste.

Infine, nella _context map_ è stata esplicitata anche la relazione tra il **Game Executor Bounded Context** e
il **Legacy Chess Engine**, per cui il primo assume il ruolo di interfaccia verso i componenti del secondo.

---

[Back to Top](#top) |
[Previous Chapter](/docs/1-domain-analysis) |
[Next Chapter](/docs/3-tactical-design)
