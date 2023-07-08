---
title: Authentication Database
layout: default
nav_order: 3
parent: Servizi
---

# Authentication Database
{: .no_toc}

```yaml
    users: {
      username: string   # the name (and unique identifier) of the user
      email: string      # the email of the user
      password: {        # the password of the user
        hash: string     # the hash of the password
        salt: string     # the salt of the password
      }
      token: {                # the token of the currently active session of the user
        id: string            # the token identifier
        expiration: datetime  # the date of expiration of the token
      }
    }[]
```
