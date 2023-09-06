---
title: Presentazione del problema
layout: default
nav_order: 2
---

# Presentazione del problema
{: .no_toc}

Di seguito, si riporta la proposta di progetto pubblicata sul forum progetti,
usata come base per definire il problema da risolvere.

## Contenuti
{: .no_toc}

- TOC
{:toc}

---

## Scenario
L'obiettivo del progetto è quello di realizzare un'applicazione web
che permetta di giocare online a scacchi.

Tale applicazione sarà realizzata a partire dall'engine degli scacchi
già implementato per il corso di 
[Paradigmi di Programmazione e Sviluppo 2022-2023](https://www.unibo.it/it/didattica/insegnamenti/insegnamento/2022/412597)
(reperibile al seguente [link](https://github.com/jahrim/PPS-22-chess)),
che sarà modificato opportunamente e a cui saranno aggiunte nuove funzionalità.

In particolare, ora l'engine permette di gestire un'unica partita di scacchi
tra due giocatori in locale, fornendo la possibilità di configurare la partita
(es.: modalità di gioco a tempo...). Questa applicazione dovrà invece permettere
di gestire più partite tra diversi giocatori online.

## Requisiti di massima
- Creare una partita di scacchi, pubblica o privata;
- Partecipare a una partita di scacchi casuale, se pubblica, o conoscendone l'identificatore,
  se privata;
- Registrarsi all'applicazione;
- Fare login all'interno dell'applicazione;
- Accedere all'applicazione come ospite (senza richiedere un account);
- Salvare i risultati delle partite, tenendo traccia delle vittorie/sconfitte degli utenti;
- Gestire il proprio profilo utente;
- Visualizzare il progresso di un certo utente nel tempo;
- Visualizzare la classifica globale dei giocatori.

## Strategia di massima
Seguendo le best practice del Domain-Driven Design, si ha intenzione di decomporre il sistema
in servizi il più possibile autonomi. Inoltre, si applicherà a ciascuno di essi le best
practice di DevOps e Continuous Integration.

---

[Back to Top](#top) |
[Previous Chapter](/docs) |
[Next Chapter](/docs/1-domain-analysis)