<!-- source: bulletproof-react docs/security.md -->

# Security: authentication and authorization

Client-side auth improves the user experience, but it is not a substitute for server-side enforcement. Every resource still needs to be protected on the server regardless of what the client does. Treat everything below as complementary to that, not a replacement for it.

## Authentication

Verifies who the user is. The standard pattern for single-page apps is JWT: the user logs in, receives a token, and the token is sent with every subsequent authenticated request.

**Where to store the token matters and the tradeoffs are not optional to consider:**

- Storing the token only in application state is the most secure option against theft, but it resets on page refresh, so on its own it's not practical.
- `localStorage` and `sessionStorage` are readable by any JavaScript running on the page. If the app has an XSS vulnerability, the token can be stolen directly. Don't store tokens here unless you've deliberately decided the tradeoff is acceptable for the app in question.
- Cookies with the `HttpOnly` flag are the safer default. `HttpOnly` cookies aren't accessible to client-side JavaScript at all, which closes off the XSS token-theft path.

Regardless of storage choice, sanitize all user input before rendering it. This is the other half of XSS defense, storage security alone doesn't help if unsanitized input gets rendered back into the page. See the OWASP client-side security top 10 for the fuller threat list.

Treat the authenticated user object as global state, available anywhere in the app. If already using react-query, the react-query-auth library handles this pattern directly. Otherwise, react context + hooks or another state library works. The convention: the app considers a user authenticated if and only if a user object is present.

## Authorization

Verifies what an authenticated user is allowed to do. Two models, and they're not mutually exclusive:

**Role-based (RBAC):** Define roles (`USER`, `ADMIN`, and so on) and grant each role a set of permissions. Simple to reason about, and it's the right default for most access decisions in an app.

**Permission-based (PBAC):** Finer-grained than roles. Use it when access depends on something more specific than role membership, most commonly ownership: only the author of a comment can delete that comment, regardless of what role they hold. RBAC alone can't express that; PBAC checks a policy specific to the resource instance.

In practice, gate UI and actions with a component that accepts either allowed roles (RBAC) or a policy check function (PBAC), and use whichever fits the specific permission being enforced.
