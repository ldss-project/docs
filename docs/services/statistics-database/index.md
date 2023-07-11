---
title: Statistics Database
layout: default
nav_order: 5
parent: Servizi
---

# Statistics Database
{: .no_toc}

```yaml
scores: {
  username: string        # the player who owns this score
  last_scores: {          # the last scores (e.g. in the last year) of the player
    date: datetime        # the date when the score was added
    rank: number          # the rank of the player on the leaderboard
    wins: number          # the number of times the player has won
    losses: number        # the number of times the player has lost
    ratio: number         # the latest ratio between wins and losses (ratio = wins/losses)
  }[]
}[]
```