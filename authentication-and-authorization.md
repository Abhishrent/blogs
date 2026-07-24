#Authentication and Authorization

Sesssions, JWTs and Cookies

a little bit of history...
Sessions
HTTP was the backbone for all communication on the web but was/is stateless. It was ideal for early web because pages were static. 
But as websites transitioned to dynamic content, statelessness of http became a bottleneck. Stateful interactions became a need.

## Sesssions

A session is a way to establish temporary server side context for each user. 

### How does it work? 

### session creation

When the user logged in, the server created a unique session id and it stores it alongside the data in a persistent store (database or redis).

### session 
Session id was sent to the client as a cookie. All the requests the client makes after that request included that cookie. 

### these sessions were short lived. had an expiry. 

initial implementations of sessions were file based which were not scalable as the user base grew... then servers started to implement db backed sessions. and eventually distributed storage (redis, memcache)

## JWTs
the addtional benefits of using JWTs...

## Emergence of JWTs.
stateful systems even though they were effective, they caused a bottleneck.

because. 
memory -> maintaining session data for millions of users became costly, a large overhead.
replication -> synchronizing session data accross different servers in different geographical locations introduced latency.

JWTs were a stateless mechanism for transferring claims between different two systems. were self contained tokens. tokens contain user data and cryptographic signatures in one token which are base 64 encoded.
jwt.io is the site for it. 

scalability -> propagate across servers...

portability -> lightweight, could be stored in cookies, etc..

challlenges: 
token theft -> no stored mechanism to validate the user identity... can impersonate... no machanism from the server side to invalidate that token until it expired  manually unless the server changed it's secret using which server verified all the users... 

revokation -> have no way to track JWT tokens status until they are expired.

hybrid approach (using JWTs as session ids), raises some questions again. 

## Cookies
Cookies are a way of storing some information from the server on the client side. then that cookie is sent with every subsequent request made by the client to that server. 

## Statelful authentication workflow

client sends the username and pw -> server validates it and creates a session id -> bundles the session id with the user data -> stores in redis -> sends the session id back to the client in a cookie (http only cookie), the use of cookie is not mandatory however and is just the general preferred practice, really comes down to your own implementation. -> every other subsequent request after that contains that cookie -> cookie is checked by the server in the redis for its existence/expiry/user data etc. -> user identified and authorized -> user has ability to call the api


note:
How you transport that ID or token is up to you as the developer. You can pass it in the JSON body of a response and then send it back in an HTTP header (like Authorization: Bearer <token>), or you can use cookies. Cookies are just highly popular because browsers handle them automatically and they offer good security flags like HttpOnly.




> the session id can be JWT token/ cryptographic string , etc depending on the implementation.

*Pros/Cons:* Offers centralized control and easy token revocation, but has limited scalability and higher operational complexity. Ideal for standard web applications.

## Stateless authenticaiton workflow
user sends the username and pw ->  server validates it and generates a signed JWT token with a secret key. The server has it's own secret key using which it can sign and verify JWTs. -> sends back to client (can use cookie or not for this) -> the client can send the JWT back to the server in the header called authorization. -> server extracts the token and tries to vierify using the secret key and then finds out user id. 

the jwt absolutely can be placed inside a cookie, but it doesn't have to be. In a stateless workflow, the server can send the JWT back to the client in two primary ways:

    In the Response Body (JSON): The server sends { "token": "eyJhbG..." }. The client (usually React, Vue, etc.) captures this and stores it in memory or localStorage. It then manually attaches it to the Authorization: Bearer <token> header for future requests.

    In an HttpOnly Cookie: The server sets a Set-Cookie header in the response. The browser automatically saves it and automatically attaches it to every subsequent request.

*Pros/Cons:* Highly scalable and portable, but revoking access before the token expires is extremely complex without forcing all users to log out by changing the secret key.

### api key based authentication workflow
programmatic accesss to a server in a confined manner.
ideal for machine to machine communication and is easy.

### OAuth 2.0


4.  *OAuth 2.0 & OpenID Connect (OIDC):*
    *   *The Delegation Problem:* Users historically shared passwords so one app could access resources on another (e.g., a travel app scanning Gmail). This was a massive security risk and made revocation impossible.
    *   *OAuth 2.0:* Solved this by allowing a client app to request a specific *token* with limited permissions (e.g., read-only access to contacts) from an Authorization Server. This handles *authorization*.
    *   *OpenID Connect (OIDC):* Because OAuth didn't verify identity*, OIDC was built on top. It introduced the **ID Token* (a JWT), which securely shares user profile data (like email and name). This powers the "Sign in with Google" features we use today.

When to use what type of authentication? 

thumbrules:

stateful -> web
stateless -> apis
OAuth -> third party
api key -> server to server


# Authorization
## RBAC role based access control
### *5. Authorization: Role-Based Access Control (RBAC)*
*   *The Concept:* Not all users should have the same capabilities (e.g., standard users vs. admins accessing a "dead zone" of deleted files). 
*   *How RBAC Works:* Users are assigned specific roles (User, Admin, Moderator). Each role maps to strict resource permissions (Read, Write, Delete).
*   *Execution:* During the request cycle, the server deduces the user's role from their token. If an unauthorized user attempts an action, the server rejects it with a `403 Forbidden` status.

### *6. Critical Security Best Practices*
*   *Use Generic Error Messages:* Never respond with "User not found" or "Incorrect password." Attackers use these friendly messages to deduce valid usernames and increase their attack surface. Always use a generic response like *"Authentication failed"*.
*   *Prevent Timing Attacks:* Attackers can measure the time it takes for a server to respond. If a username is invalid, the server fails instantly. If the username is valid but the password is wrong, the server takes longer because it has to run the heavy password-hashing algorithm. 
    *   Solution: Backend engineers must equalize response times using *constant-time operations* or by *simulating a fake response delay* (e.g., a standard 200ms delay) so attackers cannot measure the difference.
