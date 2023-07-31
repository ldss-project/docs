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
      gameConfiguration: {
        timeContraint?: {                        # if not present, then the time constraint is "No Limit"
          type: MoveLimit | PlayerLimit          # if present, the time constraint can be "Move Limit" or "Player Limit"
          time: Time
        }
        isPrivate: boolean                       # if true the game is private, otherwise it's public
        gameId?: string                          # the private game identifier
      }
      ```
    - Output: ` `
    - Errors:
        - `400`: malformed input
        - `400`: gameId already taken

- `GET` **Find public game**: find a random available public game in the Game Service.
  - URL: `game`
  - Input: ` `
  - Output:
    ```yaml
    connection: {
      websocket: string   # the url the user will have to connect to via websocket to join the game
    }
    ```
  - Errors:
    - `404`: there are no games waiting for new players

- `GET` **Find private game**: find a game with the specified id in the Game Service.
  - URL: `game/{gameId}`
  - Input: ` `
  - Output:
    ```yaml
    connection: {
      websocket: string   # the url the user will have to connect to via websocket to join the game
    }
    ```
  - Errors:
    - `403`: game already started
    - `404`: game not found

- `WEBSOCKET` **Join and play game**: join and play within a game in the Game Service.
  - URL: `game/{gameId}` (ws://)
  - Input flow:
    ```yaml
    input: {
      findMoves?: {
        position: Position
      }
      applyMove?: {
        move: Move
      }
      promote?: {
        pawn: Piece
        to: Knight, Bishop, Rook, Queen
      }
    }  
    ```
  - Output flow:
    ```yaml 
    game: {                                                    # the game the player is connected to
      server: {                                                # the server hosting the game
        state: WaitingForPlayers | Running | Terminated        # the state of the server
        error?: {                                              # the latest error occurred in the server, if any
          type: Generic | PlayerDisconnected,                  # the type of error
          message: string                                      # the description of the error
        } 
      }
      state: {                                                 # the state of the game
        turn: White | Black                                    # the player in control of the chessboard
        situation?: {                                          # the situation on the chessboard, if any
          type: Check | Stale | Checkmate | Promotion          # the type of situation
          promotingPawnPosition?: Position                     # the position of the pawn to promote (only Promotion)
        },
        white: {                                               # the information concerning the white player
          player: Player                                       # the white player
          chessboard: ChessboardStatus                         # the state of the chessboard for the white player
          history: Move[]                                      # the history of white moves
          time: Time                                           # the time remaining for the white player  
        },
        black: {                                               # the information concerning the black player
          player: Player                                       # the black player
          chessboard: ChessboardStatus                         # the state of the chessboard for the black player
          history: Move[]                                      # the history of black moves
          time: Time                                           # the time remaining for the black player  
        },
        gameOver?: {                                           # if present, the match is over
          cause: Timeout | Checkmate | Stalemate               # the cause of the match termination
          winner?: White | Black                               # the team of the winner of the match
        } 
      }
    }
    ```
  
## Schemas

```yaml
Time: {                                                  # A duration
  value: number                                          # the number of time units
  unit: scala.TimeUnit                                   # the time unit for the time configuration
}

Position: {                                              # A position in the chessboard
  file: A | B | C | D | E | F | G | H                    # the column in the chessboard
  rank: _1 | _2 | _3 | _4 | _5 | _6 | _7 | _8            # the row in the chessboard
}

Player: {                                                # A player in the game
  username: string                                       # the name of the player        
}

ChessboardStatus: {                                      # The state of the chessboard wrt a player
  pieces: Piece[]                                        # the pieces of the player
  moves: Move[]                                          # the moves of the player
}

Piece: {                                                 # A piece on the chessboard
  type: Pawn | Knight | Bishop | Rook | Queen | King     # the type of piece
  position: Position                                     # the position of the piece on the chessboard
}

Move: {                                                  # A move in the game
  type: Move | Capture                                   # the type of move
  from: Position                                         # the starting position of the move
  to: Position                                           # the arriving position of the move
  
  castling?: {                                           # If present, the move is a castling
    rook: {                                              # the rook involved in the castling
      from: Position                                     # the starting position of the rook
      to: Position                                       # the arriving position of the rook
    }
  }
  enPassant?: {                                          # If present, the move is an en-passant
    opponentPawn: Piece                                  # the opponent pawn captured by the en-passant
  }
}
```
