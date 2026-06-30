# Cookies

## Why were Cookies invented?

Sessions solve the server-side problem.

However, the browser must identify itself on every request.

Cookies solve this client-side problem.

---

## What is a Cookie?

A cookie is a small piece of data stored by the browser.

The server sends:

```http
Set-Cookie: SessionId=A12B34
```

The browser stores it.

Future requests automatically include:

```http
Cookie: SessionId=A12B34
```

---

## Browser Behaviour

Cookie handling is implemented by the browser.

Application code typically does not manually add Cookie headers.

Instead:

ASP.NET writes:

```csharp
Response.Cookies.Append(...)
```

which generates:

```http
Set-Cookie
```

The browser stores the cookie and automatically sends it on future requests.

---

## Cookies vs Sessions

Cookies are stored on the client.

Sessions are stored on the server.

Cookies usually carry only the Session ID.

The Session ID is used to retrieve server-side state.

---

## Important Cookie Attributes

To be covered later:

- HttpOnly
- Secure
- SameSite
- Domain
- Path
- Expiration

---

## Future Topics

Cookies naturally lead into:

- Same-Origin Policy
- CORS
- withCredentials
- CSRF

These topics are intentionally deferred until later.

---

# Interview Discussion

Common interview question:

"Does Angular automatically send cookies?"

Answer:

No.

The browser automatically sends cookies according to the HTTP specification.

Angular simply asks the browser to make an HTTP request.