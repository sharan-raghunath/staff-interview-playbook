# HTTP

## Why was HTTP designed to be stateless?

This is one of the most fundamental design decisions of the web.

HTTP was intentionally designed as a **stateless protocol**.

This means every request is independent, and the server does not remember previous requests from the same client.

For example:

```http
GET /products
```

The server processes the request and returns a response.

If the browser immediately sends another request:

```http
GET /cart
```

The server has no built-in knowledge that both requests came from the same user.

Every request is treated as a brand-new request.

---

## Why not make HTTP stateful?

Imagine HTTP itself remembered every connected client.

For every request, every web server would need to remember:

- Which user made previous requests.
- Shopping cart contents.
- Authentication state.
- Previously visited pages.
- Session information.

Even a request for a small static resource like:

```http
GET /logo.png
```

would require the server to maintain client state.

This would significantly increase server memory usage and reduce scalability.

Instead, HTTP follows a simple philosophy:

> HTTP is responsible for transporting requests and responses.
>
> It is **not** responsible for remembering users.

This separation of responsibilities allows servers to process millions of independent requests efficiently.

---

## The problem this creates

Modern web applications require user state.

Examples:

- Shopping carts
- Logged-in users
- User preferences
- Multi-step forms

If HTTP forgets everything after every request, how can an application remember a user?

This limitation led to the invention of **sessions**.

---

## Key Takeaways

- HTTP is stateless by design.
- Every request is independent.
- Statelessness improves scalability.
- HTTP intentionally avoids managing user state.
- Application-level mechanisms are required to maintain user state.

---

## Next

- Sessions
- Cookies
- Load Balancers
- Distributed Sessions
- JWT

---

# Interview Discussion

### Why was HTTP designed to be stateless?

A good answer should include:

- Scalability
- Separation of responsibilities
- Independent request processing
- Simpler server design
- User state handled at the application layer

Avoid saying only:

> "HTTP doesn't store state."

Instead explain **why** the protocol was intentionally designed that way.
