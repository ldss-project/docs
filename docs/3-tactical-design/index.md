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
- **Frontend Service**: modella il **Frontend Bounded Context**. 
  Si occupa di esporre e presentare le funzionalità del sistema agli utenti umani
  dell'applicazione.
- **Authentication Service**: modella l'**Authentication Bounded Context**.
  Si occupa della gestione della registrazione e del'autenticazione degli utenti nel
  sistema.
  
  La persistenza dei dati degli utenti del servizio è delegata a un
  **Authentication Database**.
- **Statistics Service**: modella lo **Statistics Bounded Context**.
  Si occupa della gestione dei punteggi degli utenti nel sistema.

  La persistenza dei punteggi degli utenti del servizio è delegata a uno
  **Statistics Database**.
- **Chess Game Service**: modella il **Game Manager Bounded Context** e il
  **Game Executor Bounded Context**.
  Si occupa della gestione della creazione, rimozione e ricerca delle partite
  di scacchi nel sistema, oltre che della connessione dei giocatori a tali
  partite e del loro svolgimento.

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
  parte dell'utente del servizio, che espone le funzionalità del servizio secondo
  lo specifico caso d'uso necessario all'utente e minimizza i danni causati da
  eventuali cambiamenti di interfaccia del servizio (fungendo da _Anti-Corruption Layer_).

Allo scopo di facilitare la creazione di servizi che aderiscono alle buone pratiche
della _Clean Architecture_, si è deciso di sviluppare una libreria che modella questi
concetti, chiamata [HexArc](https://github.com/ldss-project/hexarc). In particolare,
la libreria offre un DSL per definire in modo dichiarativo dei servizi ed eseguirne
il deployment.

## Architettura di dettaglio del sistema

---

[Back to Top](#top) |
[Previous Chapter](/docs/2-strategic-design) |
[Next Chapter](/docs/4-devops)
