---
title: Deployment
layout: default
parent: DevOps
nav_order: 4
render_with_liquid: false
---

# Deployment
{: .no_toc}

In questo capitolo si descriveranno gli strumenti utilizzati per rendere possibile l'esecuzione del sistema
nella sua interezza. Inoltre, si approfondirà come è possibile configurare l'esecuzione del sistema in base
alle proprie esigenze.

## Contenuti
{: .no_toc}

- TOC
{:toc}

---

## Docker Compose

Per eseguire il sistema nella sua interezza, si è fatto affidamento a [Docker Compose](https://docs.docker.com/compose/).
In particolare, è stato definito il seguente _docker-compose_ file:

```yaml
version: "3.4"

services:
  authentication-service:
    image: "jahrim/io.github.jahrim.chess.authentication-service:${AUTHENTICATION_SERVICE_VERSION:-latest}"
    command:
      - --mongodb-connection
      - "${AUTHENTICATION_MONGODB_CONNECTION:?the authentication service won't be able to connect to MongoDB}"
      - --mongodb-database
      - "${AUTHENTICATION_MONGODB_DATABASE:-authentication}"
      - --mongodb-collection
      - "${AUTHENTICATION_MONGODB_COLLECTION:-users}"
      - --http-host
      - "0.0.0.0"
      - --http-port
      - "8080"
      - --allowed-origins
      - "${AUTHENTICATION_ALLOWED_ORIGINS:-*;}"
    ports:
      - "${HOSTNAME:-127.0.0.1}:${AUTHENTICATION_SERVICE_PORT:-8081}:8080"
    restart: always

  statistics-service:
    image: "jahrim/io.github.jahrim.chess.statistics-service:${STATISTICS_SERVICE_VERSION:-latest}"
    command:
      - --mongodb-connection
      - "${STATISTICS_MONGODB_CONNECTION:?the statistics service won't be able to connect to MongoDB}"
      - --mongodb-database
      - "${STATISTICS_MONGODB_DATABASE:-statistics}"
      - --mongodb-collection
      - "${STATISTICS_MONGODB_COLLECTION:-scores}"
      - --http-host
      - "0.0.0.0"
      - --http-port
      - "8080"
      - --allowed-origins
      - "${STATISTICS_ALLOWED_ORIGINS:-*;}"
    ports:
      - "${HOSTNAME:-127.0.0.1}:${STATISTICS_SERVICE_PORT:-8082}:8080"
    restart: always

  chess-game-service:
    depends_on:
      - statistics-service
    image: "jahrim/io.github.jahrim.chess.chess-game-service:${CHESS_GAME_SERVICE_VERSION:-latest}"
    command:
      - --statistics-service
      - "${HOSTNAME:-127.0.0.1}:${STATISTICS_SERVICE_PORT:-8082}"
      - --http-host
      - "0.0.0.0"
      - --http-port
      - "8080"
      - --allowed-origins
      - "${CHESS_GAME_SERVICE_ALLOWED_ORIGINS:-*;}"
    ports:
      - "${HOSTNAME:-127.0.0.1}:${CHESS_GAME_SERVICE_PORT:-8083}:8080"
    restart: always

  frontend:
    depends_on:
      - authentication-service
      - statistics-service
      - chess-game-service
    image: "jahrim/io.github.jahrim.chess.frontend:${FRONTEND_VERSION:-latest}"
    ports:
      - "${HOSTNAME:-127.0.0.1}:${FRONTEND_SERVICE_PORT:-8080}:8080"
    restart: always
    environment:
      VITE_RUNTIME_ENVIRONMENT: >
        {
          "AUTHENTICATION_SERVICE": "${HOSTNAME:-127.0.0.1}:${AUTHENTICATION_SERVICE_PORT:-8081}",
          "STATISTICS_SERVICE": "${HOSTNAME:-127.0.0.1}:${STATISTICS_SERVICE_PORT:-8082}",
          "CHESS_GAME_SERVICE": "${HOSTNAME:-127.0.0.1}:${CHESS_GAME_SERVICE_PORT:-8083}"
        }
```

Come si può notare, il _docker compose_ file, inizializza i servizi del sistema a partire dalle loro
immagini, reperite in locale se disponibili, altrimenti su [DockerHub](https://hub.docker.com/).

## Configurazione

Ogni servizio del sistema è stato configurato adeguatamente attraverso degli argomenti a linea di comando oppure
delle variabili d'ambiente. Tale configurazione può essere modificata definendo delle variabili d'ambiente
all'interno della propria console, oppure più opportunamente definendo un file _.env_ che le contiene.

Di seguito, si riporta un esempio di file _.env_, che illustra le variabili d'ambiente che possono essere
specificate per configurare l'esecuzione del sistema.

```properties
### System
# The name of the system (used to name the containers)
COMPOSE_PROJECT_NAME=system
# The hostname where the system is deployed
HOSTNAME=127.0.0.1

### Frontend Service (https://github.com/ldss-project/frontend)
# The version of the frontend service to use
FRONTEND_VERSION=latest
# The port at which the frontend service will be listening
FRONTEND_PORT=8080

### Authentication Service (https://github.com/ldss-project/authentication-service)
# The version of the authentication service to use
AUTHENTICATION_SERVICE_VERSION=latest
# The port at which the authentication service will be listening
AUTHENTICATION_SERVICE_PORT=8081
# The connection to the MongoDB instance where the data of the authentication
# service will be stored
AUTHENTICATION_MONGODB_CONNECTION=...
# The MongoDB database where the data of the authentication service will be stored
AUTHENTICATION_MONGODB_DATABASE=authentication
# The collection inside the MongoDB database where the data of the authentication
# service will be stored
AUTHENTICATION_MONGODB_COLLECTION=users
# The origins that are allowed to interact with the authentication service
AUTHENTICATION_ALLOWED_ORIGINS=http://localhost:8080;http://127.0.0.1:8080;

### Statistics Service (https://github.com/ldss-project/statistics-service)
# The version of the statistics service to use
STATISTICS_SERVICE_VERSION=latest
# The port at which the statistics service will be listening
STATISTICS_SERVICE_PORT=8082
# The connection to the MongoDB instance where the data of the statistics service
# will be stored
STATISTICS_MONGODB_CONNECTION=...
# The MongoDB database where the data of the statistics service will be stored
STATISTICS_MONGODB_DATABASE=statistics
# The collection inside the MongoDB database where the data of the statistics
# service will be stored
STATISTICS_MONGODB_COLLECTION=scores
# The origins that are allowed to interact with the statistics service
STATISTICS_ALLOWED_ORIGINS=http://localhost:8080;http://127.0.0.1:8080;

### Chess Game Service (https://github.com/ldss-project/chess-game-service)
# The version of the chess game service to use
CHESS_GAME_SERVICE_VERSION=latest
# The port at which the chess game service will be listening
CHESS_GAME_SERVICE_PORT=8083
# The origins that are allowed to interact with the chess game service
CHESS_GAME_SERVICE_ALLOWED_ORIGINS=http://localhost:8080;http://127.0.0.1:8080;
```

Tutte queste configurazioni sono opzionali da specificare, ad eccezione delle stringhe di connessione ai database
[MongoDB](https://www.mongodb.com/) da cui alcuni servizi dipendono. Infatti, queste stringhe contengono alcuni
dati sensibili, come le credenziali per autenticarsi ai database, quindi non sono state rese pubbliche.
Inoltre, potrebbe essere necessario modificare la configurazione relativa alla _CORS Policy_, in base al browser
utilizzato.

## Esecuzione

In sintesi, supponendo di aver installato [Docker](https://www.docker.com/) nella propria macchina,
esistono due modalità per eseguire il sistema:
- **Via release**:
  1. Scaricare i file `docker-compose.yml` e `default.env` disponibili al seguente
     [link](https://github.com/ldss-project/frontend/releases);
  2. Mettere i file nella stessa directory;
  3. Modificare il file `default.env`, configurando l'esecuzione del sistema in base alle
     proprie esigenze ed includendo i segreti necessari alla sua esecuzione;
  4. Rinominare il file `default.env` a `.env`;
  5. Eseguire `docker compose up` all'interno della directory in cui si trovano i due file;
  6. Connettersi al `frontend` (default: http://127.0.0.1:8080).
- **Via repository**:
  1. Clonare il [frontend](https://github.com/ldss-project/frontend), in cui sono presenti i
     file `docker-compose.yml` e `.env.public`;
  2. Creare un file `.env` nella root del progetto, configurando l'esecuzione del sistema in base alle
     proprie esigenze ed includendo i segreti necessari alla sua esecuzione. Il file `.env.public` può
     essere utilizzato come template per creare il file `.env`;
  3. Eseguire `docker compose up` o `npm run deploy` all'interno della root del progetto;
  4. Connettersi al `frontend` (default: http://127.0.0.1:8080).
---

[Back to Top](#top) |
[Previous Chapter](/docs/1-devops/2-continuous-integration) |
[Next Chapter](/docs/1-devops/4-licensing)