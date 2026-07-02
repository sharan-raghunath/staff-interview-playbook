# Token Lifecycle

## Login

User opens the application and is redirected to the Identity Provider.

The Identity Provider authenticates the user using:

- username/password
- MFA
- Conditional Access
- enterprise policies

Then it issues tokens depending on the flow.

---

## Tokens

### ID Token

Purpose:

> Tell the client application who authenticated.

Consumer:

> Client application.

### Access Token

Purpose:

> Allow an API to authorize a request.

Consumer:

> Resource API.

### Refresh Token

Purpose:

> Obtain a new Access Token without prompting the user to log in again.

Consumer:

> Client application or server-side web app.

---

## Access Token Expiry

Access Tokens should be short-lived.

Why?

- limits damage if stolen
- permission changes take effect sooner
- account disablement takes effect sooner
- Conditional Access changes propagate sooner

Do not issue one-year Access Tokens for normal applications.

---

## Refresh

When the Access Token is about to expire, the application uses the Refresh Token to obtain a new Access Token.

Better implementation:

- proactively refresh before expiry
- avoid waiting for a failed API call when possible
- rely on identity libraries where appropriate

---

## Refresh Token Rotation

Modern systems may rotate Refresh Tokens.

```text
RT1 used
↓
RT2 issued
↓
RT1 invalidated
```

If RT1 is reused later, the Identity Provider can detect possible theft and revoke the token family.

---

## When All Tokens Expire

If Access Token and Refresh Token are expired or revoked, the application redirects to the Identity Provider again.

If the Identity Provider session is still valid, the user may get new tokens without entering credentials.

If the Identity Provider session is gone, the user must authenticate again.

---

## Logout

Application logout should clear local application state and cookies.

Identity Provider logout clears the Identity Provider session.

Both may be needed for strong logout semantics.

---

## Why Not Long-Lived Access Tokens?

A long-lived Access Token increases risk.

If stolen, an attacker can impersonate the user until expiry.

Short-lived Access Tokens reduce the blast radius.

---

## Staff-Level Summary

> Access Tokens should be short-lived because they are bearer credentials. Refresh Tokens preserve user experience while allowing Access Tokens to expire quickly. Once refresh is no longer possible, the user must return to the Identity Provider, where an active SSO session may or may not avoid a fresh login prompt.
