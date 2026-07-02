# OAuth 2.0

## Why OAuth Exists

Before OAuth, a third-party application that wanted access to a user's data often had to ask for the user's username and password.

Example:

```text
Travel Planner wants to read Gmail contacts.
Without OAuth, it asks for Gmail credentials.
```

Problems:

- The third-party app sees the password.
- It can do everything the user can do.
- The user cannot grant limited access.
- Password changes break integrations.
- A breach of the third-party app leaks the user's real password.

OAuth was invented to solve this problem.

OAuth allows delegated authorization:

> Let an application access a protected resource on behalf of a user without ever learning the user's password.

---

## OAuth Is Not JWT

JWT is a token format.

OAuth is a protocol for obtaining tokens.

Correct statement:

> We use OAuth 2.0, and our Identity Provider issues JWT Access Tokens.

Incorrect statement:

> We use OAuth instead of JWT.

---

## OAuth Actors

| Actor | Meaning |
|-------|---------|
| Resource Owner | The user who owns or can authorize access to data |
| Client | The application requesting access |
| Authorization Server | The Identity Provider issuing tokens |
| Resource Server | The API being accessed |

Invoice Manager mapping:

| OAuth Actor | Invoice Manager Example |
|-------------|-------------------------|
| Resource Owner | User using Invoice Manager |
| Client | Edge API / Web App |
| Authorization Server | Microsoft Entra |
| Resource Server | API Service or another protected API |

---

## Client Registration

Before an application can use OAuth, it must be registered with the Identity Provider.

The Identity Provider stores:

- Client ID
- Redirect URI
- Allowed flows
- Credentials for confidential clients
- Scopes / permissions

This prevents arbitrary applications from impersonating legitimate clients.

---

## Redirect URI Validation

After login, the Identity Provider redirects the user back to the application.

The redirect URI must match a registered URI.

This prevents an attacker from injecting:

```text
https://evil.com/callback
```

and stealing the authorization response.

---

## Authorization Code Flow

Modern web applications should use Authorization Code Flow.

Simplified flow:

```text
User
  ↓
Client application
  ↓ redirect
Identity Provider
  ↓ login + consent
Authorization Code
  ↓
Client exchanges code with client authentication
  ↓
Access Token
```

The Authorization Code is:

- short-lived
- single-use
- not the final access credential

The client must exchange it for tokens.

---

## Why Authorization Code Exists

The Identity Provider wants to answer two questions before issuing an Access Token:

1. Did the user approve?
2. Is the application requesting the token really the registered client?

The Authorization Code enables this second verification step.

Staff phrasing:

> The Authorization Code prevents the Identity Provider from issuing an Access Token immediately after user consent. Instead, it issues a short-lived code that the client must redeem using its registered credentials or PKCE. This ensures only the legitimate client receives the Access Token.

---

## Implicit Flow

Implicit Flow historically returned tokens directly through the browser.

```text
Browser → Identity Provider → Access Token in browser
```

This is less secure because browser-exposed tokens are vulnerable to:

- XSS
- malicious extensions
- browser history/logging exposure
- client-side leakage

Modern systems should prefer Authorization Code Flow.

Invoice Manager note:

> Invoice Manager currently uses Implicit Flow and is being migrated to Authorization Code Flow. Since the app is server-backed, the main improvement is moving away from tokens returned directly through the browser. PKCE is primarily critical for public clients, though it may still be supported depending on configuration.

---

## PKCE

PKCE is designed for public clients such as SPAs and mobile apps that cannot safely store a client secret.

Key idea:

1. Client generates a random Code Verifier.
2. Client sends a hashed Code Challenge during authorization.
3. Later, client sends the Code Verifier when exchanging the code.
4. Identity Provider hashes it and compares with the original challenge.

This proves the same client that started the flow is redeeming the code.

Important principle:

> You cannot safely keep a secret in JavaScript.

---

## Client Credentials Flow

Used when an application acts as itself and no user is involved.

Example uses:

- service calls Azure Key Vault
- scheduled cleanup job
- health monitoring
- service-to-service internal operation without user context

Flow:

```text
Service
  ↓ client credentials
Identity Provider
  ↓
Access Token representing service
  ↓
Resource API
```

Identity represented:

```text
Cleanup Service
```

not:

```text
Sharan
```

---

## On-Behalf-Of Flow

Used when a middle-tier API calls a downstream API on behalf of a user.

Example:

```text
Sharan
  ↓
Edge API / Web App
  ↓
API Service
```

Transport caller:

```text
Edge API / Web App
```

Business actor:

```text
Sharan
```

The downstream API may need the user's identity for authorization or audit.

OBO preserves the user context while issuing a token intended for the downstream API.

```text
Token A: aud = edge-api, sub = Sharan
Edge API exchanges it
Token B: aud = api-service, sub = Sharan
```

---

## When Not to Propagate User Identity

Do not blindly propagate user identity to every service.

Ask:

> Does this service need the user identity for authorization, audit or business logic?

For Text Extractor and Data Extractor, the answer is usually no.

They should authenticate and authorize the calling service using service identity, not end-user identity.

---

## Staff-Level Summary

> OAuth is a delegated authorization protocol. It defines how a client obtains permission to access a protected resource without handling the user's password. The specific token may be a JWT, but OAuth is about the flow and trust relationships, not the token format.
