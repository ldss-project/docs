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
      game: {
        configuration: {
          time-contraint?: {                        # if not present, then the time constraint is "No Limit"
            type: move-limit | player-limit         # if present, the time constraint can be "Move Limit" or "Player Limit"
            time: Time
          }
          private?: {                               # if not present, then the game is public
            gameId: string                          # the private game identifier
          } 
        }
      }
      ```
    - Output: ` `
    - Errors:
        - `400`: malformed input
        - `400`: gameId already taken

- `GET` **Join public game**: join a public game in the Game Service. Upgrade to websocket connection.
  - URL: `game`
  - Input: ` `
  - Output: ` `
  - Errors:

- `GET` **Join private game**: join a public game in the Game Service. Upgrade to websocket connection.
  - URL: `game/{gameId}`
  - Input: ` `
  - Output: ` `
  - Errors:
    - `403`: game already started
    - `404`: game not found

- `WEBSOCKET` **Play game**: play within a game in the Game Service.
  - Input flow:
    ```yaml
    input: {
      find-moves?: {
        position: Position
      }
      apply-move?: {
        move: Move
      }
      promote?: {
        pawn: Piece
        to: knight, bishop, rook, queen
      }
    }  
    ```
  - Output flow:
    ```yaml 
    game: {                                                    # the game the player is connected to
      server: {                                                # the server hosting the game
        state: waiting-for-players | running | terminated      # the state of the server
      }
      state: {                                                 # the state of the game
        turn: white | black                                    # the player in control of the chessboard
        situation?: check | stale | checkmate | promotion      # the situation on the chessboard, if any
      }
      white: {                                                 # the information concerning the white player
        player: Player                                         # the white player
        chessboard: ChessboardStatus                           # the state of the chessboard for the white player
        history: Move[]                                        # the history of white moves
        time: Time                                             # the time remaining for the white player  
      }
      black: {                                                 # the information concerning the black player
        player: Player                                         # the black player
        chessboard: ChessboardStatus                           # the state of the chessboard for the black player
        history: Move[]                                        # the history of black moves
        time: Time                                             # the time remaining for the black player  
      }
    }
    ```
  
## Schemas

```yaml
Time: {                                                  # A duration
  value: number                                          # the number of time units
  unit: seconds | minutes | hours                        # the time unit for the time configuration
}

Position: {                                              # A position in the chessboard
  file: a | b | c | d | e | f | g | h                    # the column in the chessboard
  rank: 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8                    # the row in the chessboard
}

Player: {                                                # A player in the game
  username: string                                       # the name of the player        
}

ChessboardStatus: {                                      # The state of the chessboard wrt a player
  pieces: Piece[]                                        # the pieces of the player
  moves: Move[]                                          # the moves of the player
}

Piece: {                                                 # A piece on the chessboard
  type: pawn | knight | bishop | rook | queen | king     # the type of piece
  position: Position                                     # the position of the piece on the chessboard
}

Move: {                                                  # A move in the game
  type: move | capture                                   # the type of move
  from: Position                                         # the starting position of the move
  to: Position                                           # the arriving position of the move
  
  castling?: {                                           # If present, the move is a castling
    rook: {                                              # the rook involved in the castling
      from: Position                                     # the starting position of the rook
      to: Position                                       # the arriving position of the rook
    }
  }
  en-passant?: {                                         # If present, the move is an en-passant
    opponent-pawn: Piece                                  # the opponent pawn captured by the en-passant
  }
}
```
