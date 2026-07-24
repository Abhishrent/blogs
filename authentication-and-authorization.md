# Authentication and Authorization
The code implementation for this blog is [here](https://github.com/Abhishrent/authentication-authorization-implementation).
## A Brief History

HTTP has always been stateless. It was designed to serve static pages, so this wasn't a problem at first. The server didn't need to remember anything about a client between requests.

As websites moved from static pages to dynamic, personalized content, this statelessness became a bottleneck. Applications needed to know who was making a request and keep track of information about that user across multiple requests. This need gave rise to sessions, and later, JWTs.

---

## Sessions

A session is a way to create temporary, server-side context for each user, so the server can recognize the same user across multiple requests.

### How a Session Works

**1. Session creation**
When a user logs in, the server creates a unique session ID and stores it alongside the user's data in a persistent store, such as a database or Redis.

**2. Sending the session ID to the client**
The session ID is sent to the client as a cookie. Every subsequent request the client makes includes this cookie automatically.

**3. Expiry**
Sessions are short-lived and have an expiry time, after which the user is required to log in again.

### The Evolution of Session Storage

Early implementations of sessions were file-based. This didn't scale well as the user base grew, so servers moved to database-backed sessions, and eventually to distributed storage systems like Redis and Memcached, which could handle much larger volumes of session data efficiently.

---

## JWTs (JSON Web Tokens)

### Why JWTs Emerged

Stateful systems (like session-based auth) work well, but they introduce a bottleneck as an application scales:

- **Memory overhead** – Maintaining session data for millions of users becomes costly.
- **Replication overhead** – Synchronizing session data across servers in different geographic regions introduces latency.

JWTs solve this by being a **stateless** mechanism for transferring claims (pieces of information) between two systems. A JWT is self-contained: it holds the user's data and a cryptographic signature together in one token, encoded in base64. You can inspect and decode JWTs at [jwt.io](https://jwt.io).

### Benefits

- **Scalability** – Since no session data needs to be stored server-side, JWTs can be verified by any server that has the secret key, making it easy to scale horizontally.
- **Portability** – JWTs are lightweight and can be stored in cookies, local storage, or sent in headers.

### Challenges

- **Token theft** – Since there's no server-side record of issued tokens, if a JWT is stolen, an attacker can impersonate the user. The server has no way to invalidate that specific token before it expires, short of changing the secret key used to verify *all* tokens.
- **Revocation** – There's no built-in way to track a JWT's status (valid or revoked) until it naturally expires.
- **Hybrid approaches** – Some systems use JWTs as session IDs to get some benefits of both approaches, but this reintroduces some of the same statefulness problems JWTs were meant to solve.

---

## Cookies

A cookie is a small piece of data that the server stores on the client's browser. Once set, the browser automatically attaches that cookie to every subsequent request made to the same server.

---

## Stateful Authentication Workflow

1. The client sends a username and password to the server.
2. The server validates the credentials and creates a session ID.
3. The server bundles the session ID with the user's data and stores it in Redis (or another store).
4. The server sends the session ID back to the client in a cookie (typically an HTTP-only cookie).
5. Every subsequent request from the client includes this cookie.
6. The server checks Redis for the cookie's existence, expiry, and associated user data.
7. Once the user is identified and authorized, they can call the API.

**Note:** Using a cookie isn't mandatory. How you transport the session ID or token is up to you as the developer. You could send it in the JSON body of a response and expect the client to return it in an `Authorization: Bearer <token>` header, or use cookies. Cookies are popular because browsers handle them automatically and support useful security flags like `HttpOnly`.

The "session ID" itself can be a JWT, a random cryptographic string, or any other unique identifier, depending on the implementation.

**Pros/Cons:** Offers centralized control and easy token revocation, but has limited scalability and higher operational complexity. Ideal for standard web applications.

---

## Stateless Authentication Workflow

1. The user sends their username and password to the server.
2. The server validates the credentials and generates a signed JWT using its own secret key (the same key it uses later to verify tokens).
3. The server sends the JWT back to the client (with or without a cookie).
4. The client sends the JWT back to the server on future requests, typically in the `Authorization` header.
5. The server extracts the token, verifies it using the secret key, and identifies the user from its payload.

The JWT can be placed inside a cookie, but it doesn't have to be. In a stateless workflow, the server can send the JWT back to the client in two primary ways:

- **In the response body (JSON):** The server sends `{ "token": "eyJhbG..." }`. The client (e.g., a React or Vue app) captures this and stores it in memory or `localStorage`, then manually attaches it to the `Authorization: Bearer <token>` header on future requests.
- **In an HttpOnly cookie:** The server sets a `Set-Cookie` header in the response. The browser automatically stores it and attaches it to every subsequent request.

**Pros/Cons:** Highly scalable and portable, but revoking access before the token expires is extremely complex without forcing all users to log out by rotating the secret key.

---

## API Key-Based Authentication

This workflow enables programmatic access to a server in a confined, limited manner. It's ideal for machine-to-machine communication and is straightforward to implement.

---

## OAuth 2.0 and OpenID Connect (OIDC)

### The Delegation Problem

Historically, if one app needed to access resources on another (e.g., a travel app reading your Gmail), users had to share their actual passwords. This was a major security risk and made revocation nearly impossible, since the only way to cut off access was to change your password everywhere.

### OAuth 2.0

OAuth 2.0 solved this by letting a client app request a specific **token** with limited permissions (e.g., read-only access to contacts) from an Authorization Server, instead of the user's password. This handles **authorization**: what an app is allowed to do.

### OpenID Connect (OIDC)

OAuth 2.0 alone doesn't verify a user's identity, it only handles permissions. OIDC was built on top of OAuth 2.0 to solve this. It introduces the **ID Token** (a JWT) which securely shares user profile data, such as email and name. This is what powers "Sign in with Google" and similar features.

---

## When to Use What: Rules of Thumb

| Use Case | Recommended Approach |
|---|---|
| Web applications | Stateful (sessions) |
| APIs | Stateless (JWTs) |
| Third-party access | OAuth 2.0 |
| Server-to-server | API keys |

---

## Authorization

### Role-Based Access Control (RBAC)

**The concept:** Not all users should have the same capabilities. For example, standard users and admins should have different levels of access, such as to a "trash" area of deleted files.

**How RBAC works:** Users are assigned specific roles (e.g., User, Admin, Moderator). Each role maps to a strict set of resource permissions (Read, Write, Delete).

**Execution:** During the request cycle, the server determines the user's role from their token or session. If a user without the required permissions attempts an action, the server rejects the request with a `403 Forbidden` status.

---

## Critical Security Best Practices

### Use Generic Error Messages

Never respond with specific messages like "User not found" or "Incorrect password." Attackers can use these details to figure out which usernames are valid, narrowing down their attack surface. Always use a generic message, such as *"Authentication failed."*

### Prevent Timing Attacks

Attackers can measure how long a server takes to respond to infer information about a request. If a username is invalid, the server typically fails instantly. But if the username is valid and only the password is wrong, the server takes longer, because it has to run a computationally heavy password-hashing algorithm to check it.

**Solution:** Backend engineers should equalize response times using constant-time operations, or by simulating a fake response delay (e.g., a fixed 200ms delay) so attackers cannot use timing differences to distinguish valid usernames from invalid ones.
