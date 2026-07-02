# OpenID Connect (OIDC)

## Why OIDC Exists

OAuth answers:

> Can this client access this protected resource?

It does not primarily answer:

> Who logged in?

Applications also need authentication information:

- user name
- email
- tenant
- authentication time
- MFA information
- identity provider

OIDC adds authentication on top of OAuth.

---

## ID Token

OIDC introduces the ID Token.

The ID Token is intended for the client application.

It answers:

> Who is the authenticated user?

Example claims:

```json
{
  "sub": "...",
  "name": "Sharan Kumar",
  "preferred_username": "user@company.com",
  "tid": "tenant-id",
  "auth_time": 1234567890,
  "amr": ["pwd", "mfa"]
}
```

---

## ID Token vs Access Token

| Token | Audience | Purpose | Consumer |
|-------|----------|---------|----------|
| ID Token | Client application | Authentication | Web app / client |
| Access Token | Resource API | Authorization | API |

ID Token answers:

> Who logged in?

Access Token answers:

> What can this caller access?

---

## APIs Should Not Accept ID Tokens

An API should reject an ID Token because the ID Token is not intended for the API.

Example:

```json
{
  "aud": "edge-api-webapp"
}
```

If API Service expects:

```text
aud = api-service
```

then validation should fail.

Principle:

> Never use a token outside its intended audience and purpose.

---

## OIDC and SSO

OIDC enables applications to rely on a central Identity Provider.

The user signs in with the Identity Provider, and applications trust authentication responses from it.

This supports Single Sign-On.

---

## Staff-Level Summary

> OAuth is for delegated authorization. OIDC layers authentication on top of OAuth by adding the ID Token, which tells the client application who authenticated. APIs should authorize requests using Access Tokens, not ID Tokens.
