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
  wins: number            # the number of times the player has won
  losses: number          # the number of times the player has lost
  last_scores: {          # the last scores (e.g. in the last year) of the player
    date: datetime        # the date when the score was added
    ratio: number         # the ratio between wins and losses (ratio = wins/losses)
  }[]
}[]
```