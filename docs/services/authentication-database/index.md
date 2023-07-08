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
      password: {        # the password of the user
        hash: string     # the hash of the password
        salt: string     # the salt of the password
      }
    }[]
```
