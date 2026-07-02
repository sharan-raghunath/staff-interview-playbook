# Logout, SSO and SLO

## Local Logout

Local logout clears the application session.

For Invoice Manager this means the Edge API / Web App clears its local authenticated state and instructs the browser to remove relevant cookies.

After local logout, the application no longer knows the user.

---

## Identity Provider Session

The Identity Provider, such as Microsoft Entra, may still have its own session cookie in the browser.

If the user returns to the application after local logout, the app redirects to Entra.

If Entra still has an active session, it may issue fresh tokens without asking for username/password/MFA again.

This explains why users can appear to be automatically logged in after logout.

---

## Identity Provider Logout

To force logout from the Identity Provider, the application redirects to the IdP logout endpoint.

This clears the IdP session.

Next time the user visits an application relying on that IdP, the user may need to authenticate again.

---

## Single Sign-On (SSO)

SSO means the user authenticates once with the Identity Provider and can access multiple applications that trust that IdP.

Example:

- Outlook
- Teams
- SharePoint
- Invoice Manager

All rely on Entra.

---

## Single Logout (SLO)

SLO attempts to terminate the user's session across participating applications.

In theory, global logout should sign the user out of all applications in the SSO session.

In practice, logout can be nuanced because:

- applications may maintain local sessions
- browser tabs may have cached state
- refresh tokens may still exist until revoked/expired
- some applications may not fully participate in SLO
- network failures can interrupt logout notifications

---

## Staff-Level Summary

> Logout has multiple layers. Clearing the application session does not necessarily clear the Identity Provider session. A complete logout strategy must consider local application state, Identity Provider session state, Refresh Tokens and whether Single Logout is supported across applications.
