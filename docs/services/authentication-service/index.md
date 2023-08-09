---
title: Authentication Service
layout: default
nav_order: 2
parent: Servizi
---

# Authentication Service
{: .no_toc}

- `POST` **Register user**: register a new user to the application. Creates a session.
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
    session: {     
      username: string      # the user owning the session
      token: {              # the token of the just-created active session of the user
        id: string          # the token identifier
        expiration: date    # the date of expiration of the token
      }
    }
    ```
  - Errors:
    - `400`: MalformedInputException
    - `400`: UsernameAlreadyTakenException

- `POST` **Log in user**: authenticate a user in the application. Creates a session.
    - URL: `user/{username}/log-in`
    - Input:
      ```yaml
      user: {
        password: string      # the password of the user
      }
      ```
    - Output:
      ```yaml
      session: {     
        username: string      # the user owning the session
        token: {              # the token of the just-created active session of the user
          id: string          # the token identifier
          expiration: date    # the date of expiration of the token
        }
      }
      ```
    - Errors:
      - `400`: MalformedInputException
      - `400`: IncorrectPasswordException
      - `404`: UserNotFoundException

- `GET` **Get user information**: get the profile information of a user in the application. Requires a session.
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
      - `403`: UserNotAuthorizedException
      - `404`: UserNotFoundException

- `PUT` **Update password**: update the password of a user in the application. Requires a session.
    - URL: `user/{username}/password`
    - Input:
      ```yaml
      user: { 
          password: string
      }
      ```
    - Output: ` `
    - Errors:
      - `403`: UserNotAuthorizedException
      - `404`: UserNotFoundException

- `POST` **Log out user**: revoke the authentication of a user in the application. Delete the current session.
    - URL: `user/:username/log-out`
    - Input: ` `
    - Output: ` `
    - Errors: ` `
