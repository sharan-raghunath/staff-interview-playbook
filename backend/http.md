# HTTP

## Why was HTTP designed to be stateless?

HTTP was intentionally designed as a **stateless protocol**.

This means each request is independent. The server does not automatically remember previous requests from the same client.

Example:

```http
GET /products
```

The server processes the request and returns a response.

Later:

```http
GET /cart
```

HTTP itself does not tell the server that both requests came from the same user.

---

## Why not make HTTP stateful?

If HTTP itself remembered every client, every server would need to maintain state for:

- who the user is
- what pages they visited
- their shopping cart
- login state
- preferences
- session information

This would increase memory usage and make horizontal scaling harder.

HTTP follows a clean responsibility boundary:

> HTTP transports requests and responses.  
> It does not remember users.

---

## The problem this creates

Real applications need continuity.

Examples:

- Shopping carts
- Logged-in users
- Multi-step forms
- User preferences

Since HTTP does not remember anything by itself, applications need higher-level mechanisms to maintain state.

This leads to:

```text
HTTP
↓
Stateless
↓
Sessions
↓
Cookies
```

---

## Key Takeaways

- HTTP is stateless by design.
- Statelessness improves scalability and simplicity.
- User state must be handled above HTTP.
- Sessions and cookies were introduced to solve continuity problems.

---

## Interview Discussion

A strong answer to "Why is HTTP stateless?" should include:

- Each request is independent.
- HTTP does not maintain user state.
- Statelessness simplifies server design.
- Statelessness supports scalability.
- Application-level mechanisms handle user continuity.

Avoid saying only:

> HTTP does not store state.

Instead explain why that design exists.
