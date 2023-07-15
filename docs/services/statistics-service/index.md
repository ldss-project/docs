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
  - Output: ` `
  - Errors:
    - `400`: malformed input
      
- `DELETE` **Delete scores by username**: remove the scores of a player.
  - URL: `/score/{username}`
  - Input: ` `
  - Output: ` `
  - Errors:
    - `404`: username not found

- `GET` **Get score**: get the score of a player.
  - URL: `/score/{username}`
  - Input: ` `
  - Output:
    ```yaml
    score: {
      date: datetime        # the date when the score was added
      rank: number          # the rank of the player on the leaderboard
      wins: number          # the number of times the player has won
      losses: number        # the number of times the player has lost
      ratio: number         # the latest ratio between wins and losses (ratio = wins/losses)
    }
    ```
  - Errors:
    - `404`: username not found

- `GET` **Get score history**: get the score history of a player.
  - URL: `/score/history/{username}`
  - Input: ` `
  - Output:
    ```yaml
    scores: {
      date: datetime        # the date when the score was added
      rank: number          # the rank of the player on the leaderboard
      wins: number          # the number of times the player has won
      losses: number        # the number of times the player has lost
      ratio: number         # the latest ratio between wins and losses (ratio = wins/losses)
    }[]
    ```
  - Errors:
    - `404`: username not found

- `GET` **Get leaderboard**: get a leaderboard created from the registered scores.
  - URL: `/leaderboard/?first=1?last=100`
  - Input: ` `
  - Output:
    ```yaml
    leaderboard: {
      scores: {
        username: string        # the player who owns this score
        date: datetime          # the date when the score was added
        rank: number            # the rank of the player on the leaderboard
        wins: number            # the number of times the player has won
        losses: number          # the number of times the player has lost
        ratio: number           # the ratio between wins and losses (ratio = wins/losses)
      }[]
    }
    ```
