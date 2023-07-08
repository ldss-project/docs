---
title: Statistics Service
layout: default
nav_order: 4
parent: Servizi
---

# Statistics Service
{: .no_toc}

- `POST` **Add new score**: add a new score to the Statistics Database.
  - URL: `/score/{username}`
  - Input:
    ```yaml
    score: {
      hasWon: boolean       # if the player has won
    }
    ```
  - Output:
    ```yaml
    result: {
      code: number          # the operation code (0: success, !0: error)
      errorMessage?: string # the error message
    }
    ```
      
- `DELETE` **Delete scores by username**: remove the scores of a player.
  - URL: `/score/{username}`
  - Input: ` `
  - Output:
    ```yaml
    result: {
      code: number          # the operation code (0: success, !0: error)
      errorMessage?: string # the error message
    }
    ```

- `GET` **Get score**: get the score of a player.
  - URL: `/score/{username}`
  - Input: ` `
  - Output:
    ```yaml
    score: {
      rank: number          # the rank of the player on the leaderboard
      wins: number          # the number of times the player has won
      losses: number        # the number of times the player has lost
      ratio: number         # the latest ratio between wins and losses (ratio = wins/losses)
    }
    ```

- `GET` **Get score history**: get the score history of a player.
  - URL: `/score/history/{username}`
  - Input: ` `
  - Output:
    ```yaml
    scores: {
      date: datetime         # the date when the score was added
      ratio: number          # the ratio between wins and losses (ratio = wins/losses)
    }[]
    ```

- `GET` **Get leaderboard**: get a leaderboard created from the registered scores.
  - URL: `/leaderboard/?first?last`
  - Input: ` `
  - Output:
    ```yaml
    leaderboard: {
      scores: {
        username: string        # the player who owns this score
        rank: number            # the rank of the player on the leaderboard
        wins: number            # the number of times the player has won
        losses: number          # the number of times the player has lost
        ratio: number           # the ratio between wins and losses (ratio = wins/losses)
      }[]
    }
    ```
