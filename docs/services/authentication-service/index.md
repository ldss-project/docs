---
title: Authentication Service
layout: default
nav_order: 2
parent: Servizi
---

# Authentication Service
{: .no_toc}

- `POST` **Register user**: register a new user to the application.
  - URL: `user/{username}/sign-in`
  - Input:
    ```yaml
    user: {
      email: string         # the email of the user
      password: string      # the password of the user
    }
    ```
  - Output:
    ```yaml
    user: {                 
      token: {              # the token of the just-created active session of the user
        id: string          # the token identifier
      }
    }
    ```
  - Errors:
    - `400`: malformed input
    - `400`: username already taken

- `POST` **Log in user**: authenticate a user in the application.
    - URL: `user/{username}/log-in`
    - Input:
      ```yaml
      user: {
        password: string      # the password of the user
      }
      ```
    - Output:
      ```yaml
      user: {                 
        token: {              # the token of the just-created active session of the user
          id: string          # the token identifier
        }
      }
      ```
    - Errors:
      - `400`: malformed input
      - `400`: incorrect password
      - `404`: username not found

- `GET` **Get user information**: get the profile information of a user in the application.
    - URL: `user/{username}/profile`
    - Input: ` `
    - Output:
      ```yaml
      user: {                 
        username: string
        email: string
      }
      ```
    - Errors:
        - `404`: username not found

- `PUT` **Update password**: update the password of a user in the application.
    - URL: `user/{username}/password`
    - Input:
      ```yaml
      user: { 
          password: string
      }
      ```
    - Output:
    - Errors:
        - `404`: username not found

- `GET` **Validate token**: validate a token of a user in the application.
    - URL: `token/{tokenId}/validate`
    - Input: ` `
    - Output:
      ```yaml
      user: {
        username: string        # the name (and unique identifier) of the user
      }
      ```
    - Errors:
        - `403`: expired token

- `DELETE` **Revoke token**: revoke a token of a user in the application.
    - URL: `token/{tokenId}/revoke`
    - Input: ` `
    - Output: ` `
      ```yaml
      user: {
        username: string        # the name (and unique identifier) of the user
      }
      ```
    - Errors:
        - `403`: expired token
