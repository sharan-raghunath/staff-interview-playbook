# Sessions

## Why were Sessions invented?

HTTP is stateless.

After each request, HTTP itself forgets the client.

Applications, however, need user-specific state:

- logged-in user
- shopping cart
- preferences
- tenant information
- roles / claims
- workflow state

Sessions were introduced to associate multiple HTTP requests with the same user.

---

## What is a Session?

A session is server-side state associated with a unique Session ID.

Example:

| Session ID | Server-side Data |
|------------|------------------|
| ABC123 | userId, roles, tenant, cart |

The browser does not store the whole session.

The browser usually stores only the Session ID.

---

## Session Data vs Session ID

Browser stores:

```text
SessionId = ABC123
```

Server stores:

```json
{
  "sessionId": "ABC123",
  "userId": "user-42",
  "roles": ["Reviewer"],
  "tenant": "AF",
  "cart": ["item1", "item2"]
}
```

This distinction is critical.

The browser has an opaque identifier.

The server has the actual identity, claims and application state.

---

## How Cookies help Sessions

When the server creates a session, it sends:

```http
Set-Cookie: SessionId=ABC123; HttpOnly; Secure; SameSite=Lax
```

The browser stores the cookie.

For subsequent requests to the same domain/application, the browser automatically sends:

```http
Cookie: SessionId=ABC123
```

The server uses the Session ID to look up session data.

---

## Single Server

Sessions work simply when there is only one server:

```text
Browser
↓
Server A
↓
In-memory session store
```

Server A can look up the session in its own memory.

---

## Multiple Servers

With load balancing:

```text
Browser
↓
Load Balancer
├── Server A
└── Server B
```

If Server A created the session but the next request reaches Server B, Server B may not have that session in memory.

The Session ID is not necessarily invalid. It is valid-looking, but Server B cannot find matching state.

Possible outcomes:

- user appears logged out
- cart appears empty
- session lookup returns null
- application throws an error

---

## Shared Session Store

To solve this, session state can be moved to a centralized store:

```text
Server A ─┐
          ├── Redis / Database
Server B ─┘
```

Now all servers can resolve the same Session ID.

Common stores:

- Redis
- distributed cache
- database

---

## Trade-offs

Shared session stores solve horizontal scaling but introduce new problems:

- extra network call per request
- Redis/database becomes a dependency
- additional latency
- HA/failover required
- operational complexity
- session expiry and cleanup management

---

## Sessions vs JWT

Session-based model:

```text
Browser stores opaque Session ID.
Server stores identity and state.
```

JWT model:

```text
Browser stores signed token containing claims.
API validates token without session lookup.
```

Neither is universally better.

Sessions make revocation easier.

JWT improves stateless validation but makes immediate revocation harder.

---

## Interview Discussion

Strong answer:

> Sessions exist because HTTP is stateless. The server creates state for the user and maps it to a Session ID. The browser stores only the Session ID, usually in a cookie, and sends it on future requests. In horizontally scaled systems, in-memory sessions break because requests may hit different servers, so session state is often moved to a distributed store like Redis. This solves scaling but introduces a new dependency in the request path.
