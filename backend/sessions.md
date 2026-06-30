# Sessions

## Why were Sessions invented?

HTTP is a stateless protocol.

After every request, the server forgets everything about the client.

However, real-world applications require continuity.

Examples include:

- Shopping carts
- Logged-in users
- Multi-step forms
- User preferences

Applications therefore needed a mechanism to associate multiple HTTP requests with the same user.

Sessions solve this problem.

---

## What is a Session?

A session is server-side state associated with a unique identifier.

Example:

Session Table

| Session ID | User | Shopping Cart |
|------------|------|---------------|
| A12B34     | John | iPhone, AirPods |

The browser never sees the shopping cart directly.

Instead, it only stores the Session ID.

---

## How does the browser know its Session ID?

When the server creates a session, it sends:

```http
Set-Cookie: SessionId=A12B34
```

The browser stores this cookie.

For every subsequent request to the same domain, the browser automatically sends:

```http
Cookie: SessionId=A12B34
```

The browser handles this automatically.

Angular, React or JavaScript do not manually attach the Cookie header.

---

## Session Lifecycle

1. User visits the application.
2. Server creates a Session ID.
3. Server stores session state.
4. Server returns `Set-Cookie`.
5. Browser stores the cookie.
6. Browser automatically sends the cookie on subsequent requests.
7. Server retrieves session data using the Session ID.

---

## Advantages

- Session data never leaves the server.
- Easy to invalidate.
- Sensitive data remains server-side.

---

## Limitations

Single-server deployments work well.

Multiple application servers introduce challenges.

Example:

```
          Load Balancer
         /             \
   Server A         Server B
```

If the session exists only in Server A's memory and the next request reaches Server B, the session cannot be found.

This problem leads to:

- Sticky Sessions
- Distributed Session Stores
- Redis

These will be discussed later.

---

## Key Takeaways

- Sessions exist because HTTP is stateless.
- Sessions store user state on the server.
- Browsers remember only the Session ID.
- Cookies transport the Session ID.
- Sessions become challenging in distributed systems.

---

# Interview Discussion

A good Staff Engineer answer should explain:

- Why sessions were invented.
- Why HTTP intentionally avoided user state.
- Why sessions become problematic in horizontally scaled systems.
- Why distributed session stores such as Redis are commonly introduced.