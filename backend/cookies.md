# Cookies

## Why were Cookies invented?

Sessions solve the server-side problem:

> How do we store user state?

Cookies solve the client-side transport problem:

> How does the browser remember and send the Session ID on future requests?

---

## What is a Cookie?

A cookie is a small piece of data stored by the browser and automatically sent to matching domains.

Server response:

```http
Set-Cookie: SessionId=ABC123; HttpOnly; Secure; SameSite=Lax
```

Future request:

```http
Cookie: SessionId=ABC123
```

---

## Browser Behaviour

Cookie handling is browser behaviour.

Angular, React or JavaScript do not manually add the `Cookie` header.

The browser decides which cookies belong to a request based on:

- domain
- path
- expiry
- Secure
- SameSite
- other cookie attributes

Application frameworks such as ASP.NET generate `Set-Cookie` headers, but the browser stores and resends cookies.

---

## ASP.NET Role

ASP.NET can create cookies using APIs like:

```csharp
Response.Cookies.Append("SessionId", sessionId);
```

This results in an HTTP response header:

```http
Set-Cookie: SessionId=...
```

The browser takes over from there.

---

## Cookies vs Sessions

| Concept | Stored Where | Purpose |
|---------|--------------|---------|
| Cookie | Browser | Transport data such as Session ID |
| Session | Server | Store user/application state |

A common pattern:

```text
Cookie carries Session ID.
Session store contains user state.
```

---

## Important Attributes

### HttpOnly

Prevents JavaScript from reading the cookie.

Useful for reducing XSS impact.

### Secure

Cookie is sent only over HTTPS.

### SameSite

Controls whether cookies are sent with cross-site requests.

Important for CSRF protection.

### Domain and Path

Control where the browser sends the cookie.

### Expiry / Max-Age

Controls cookie lifetime.

---

## Interview Discussion

Question:

> Does Angular send cookies?

Good answer:

> Angular does not manually send the Cookie header. The browser automatically attaches cookies to matching requests according to cookie rules. Angular only initiates the HTTP request.

---

## Future Topics

Cookies naturally lead to:

- Same-Origin Policy
- CORS
- `withCredentials`
- CSRF
- Secure cookie storage
