---
title: Game Service
layout: default
nav_order: 1
parent: Servizi
---

# Game Service
{: .no_toc}

- `POST` **Create game**: create a new game in the Game Service.
    - URL: `game`
    - Input:
      ```yaml
      gameConfiguration: GameConfiguration
      ```
    - Output:
      ```yaml
      connection:
        websocket: string   # the url the user will have to connect to via websocket to join the game (without the host)
      ```
    - Errors:
        - `400`: MalformedInputException
        - `403`: GameIdAlreadyTakenException

- `GET` **Find public game**: find a random available public game in the Game Service.
  - URL: `game`
  - Input: ` `
  - Output:
    ```yaml
    connection:
      websocket: string   # the url the user will have to connect to via websocket to join the game (without the host)
    ```
  - Errors:
    - `404`: NoAvailableGamesException

- `GET` **Find private game**: find a game with the specified id in the Game Service.
  - URL: `game/{gameId}`
  - Input: ` `
  - Output:
    ```yaml
    connection:
      websocket: string   # the url the user will have to connect to via websocket to join the game (without the host)
    ```
  - Errors:
    - `403`: GameAlreadyStartedException
    - `404`: GameNotFoundException

- `WEBSOCKET` **Join and play game**: join and play within a game in the Game Service.
  - URL: `game/{gameId}`
  - Request-Response:
    - GetState:
      - Input: 
        ```yaml
        methodCall:
          method: "GetState"
        ```
      - Output:
        ```yaml
        methodCall:
          method: "GetState"
          output: 
            serverState: ServerState
        ```
    - JoinGame:
      - Input:
        ```yaml
        methodCall:
          method: "JoinGame"
          input:
            player: Player
        ```
      - Output:
        ```yaml
        methodCall:
          method: "JoinGame"
        ```
    - FindMoves:
      - Input:
        ```yaml
        methodCall:
          method: "FindMoves"
          input:
            position: Position
        ```
      - Output:
        ```yaml
          methodCall:
            method: "FindMoves"
            output:
              moves: Move[]
        ``` 
    - ApplyMove:
      - Input:
        ```yaml
        methodCall:
          method: "ApplyMove"
          input:
            move: Move
        ```
      - Output:
         ```yaml
        methodCall:
          method: "ApplyMove"
        ```
    - Promote:
      - Input:
        ```yaml
        methodCall:
          method: "Promote"
          input:
            promotionChoice: PromotionChoice
        ```
      - Output:
        ```yaml
        methodCall:
          method: "Promote"
        ```
  - Events:
    - ChessGameServiceEvent:
      - LoggingEvent:
        ```yaml
        type: "LoggingEvent"
        payload: string
        ```
      - ServerStateUpdateEvent:
        - GameStateUpdateEvent:
          - ChessboardUpdateEvent:
          ```yaml
          type: "ChessboardUpdateEvent"
          payload: Chessboard
          ```
          - GameOverUpdateEvent:
          ```yaml
          type: "GameOverUpdateEvent"
          payload: GameOver
          ```
          - GameSituationUpdateEvent:
          ```yaml
          type: "GameSituationUpdateEvent"
          payload: GameSituation
          ```
          - MoveHistoryUpdateEvent:
          ```yaml
          type: "MoveHistoryUpdateEvent"
          payload: MoveHistory
          ```
          - PlayerUpdateEvent:
            - WhitePlayerUpdateEvent:
            ```yaml
            type: "WhitePlayerUpdateEvent"
            payload: Player
            ```
            - BlackPlayerUpdateEvent:
            ```yaml
            type: "BlackPlayerUpdateEvent"
            payload: Player
            ```
          - TimerUpdateEvent:
            - WhiteTimerUpdateEvent:
            ```yaml
            type: "WhiteTimerUpdateEvent"
            payload: Duration
            ```
            - BlackTimerUpdateEvent:
            ```yaml
            type: "BlackTimerUpdateEvent"
            payload: Duration
            ```
          - TurnUpdateEvent:
          ```yaml
          type: "TurnUpdateEvent"
          payload: Team
          ```
        - ServerErrorUpdateEvent:
        ```yaml
        type: "ServerErrorUpdateEvent"
        payload: ChessGameServiceException
        ```
        - ServerSituationUpdateEvent:
        ```yaml
        type: "ServerSituationUpdateEvent"
        payload: ServerSituation
        ```
        - SubscriptionUpdateEvent:
        ```yaml
        type: "SubscriptionUpdateEvent"
        payload: string[]
        ```
## Schemas

### ServerState
```yaml
situation: ServerSituation
subscriptions: string[]
error?: ChessGameServiceException
gameState: GameState
```

### ServerSituation
```scala
"NotConfigured" | "WaitingForPlayers" | "Ready" | "Running" | "Terminated"
```

### ChessGameServiceException
```yaml
type: string       # any class name of E <: ChessGameServiceException
message: string
```

### GameState
```yaml
chessboard: Chessboard
moveHistory: MoveHistory
currentTurn: Team
configuration: GameConfiguration
situation: GameSituation
gameOver?: GameOver
timers: TimerMap
```

### Chessboard
```yaml
pieces: {
  piece: Piece
  position: Position
}[]
```

### Piece
```yaml
type: PieceType
team: Team
```
### PieceType
```scala
"Pawn" | "Knight" | "Bishop" | "Rook" | "Queen" | "King"
```

### Team
```scala
"WHITE" | "BLACK"
```

### Position
```yaml
file: File
rank: Rank
```

### File
```scala
"A" | "B" | "C" | "D" | "E" | "F" | "G" | "H"
```

### Rank
```scala
"1" | "2" | "3" | "4" | "5" | "6" | "7" | "8"
```

### MoveHistory
```yaml
entries: {
  piece: Piece
  move: Move
}[]
```

### Move
```yaml
from: Position
to: Position
capturedPiece?: Piece
castling?:
  rookFrom: Position
  rookTo: Position
doubleMove?:
  passingBy: Position
enPassant?:
  capturedPawnPosition: Position
```

### GameConfiguration
```yaml
timeContraint?: TimeConstraint          # Default: "NoLimit"
whitePlayer?: Player                    # Default: NoWhitePlayer
blackPlayer?: Player                    # Default: NoBlackPlayer
gameMode?: GameMode                     # Default: "PVP"
isPrivate?: boolean                     # Default: false
gameId?: string                         # Default: random
```

### TimeConstraint
```yaml
type: TimeConstraintType
time?: Duration                         # absent for "NoLimit"
```

### TimeConstraintType
```scala
"NoLimit" | "MoveLimit" | "PlayerLimit"
```

### Duration
```yaml
value: { $numberLong: string }
unit: string                            # any java.util.concurrent.TimeUnit as string
```

### Player
```yaml
team: Team
name?: string                           # Default: "_" (meaning guest)
```

### GameMode
```scala
"PVP" | "PVE"
```

### GameSituation
```yaml
type: GameSituationType
promotingPawnPosition?: Position        # only if 'type' is 'Promotion'
```

### GameSituationType
```scala
"None" | "Check" | "Stale" | "Checkmate" | "Promotion"
```

### GameOver
```yaml
cause: GameOverCause
winner?: Player
```

### GameOverCause
```scala
"Checkmate" | "Stalemate" | "Surrender" | "Timeout"
```

### TimerMap
```yaml
white?: Duration
black?: Duration
```

### PromotionChoice
```scala
"Pawn" | "Knight" | "Bishop" | "Rook"
```