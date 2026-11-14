# Quiz: Session Auth

Review first: [Session Auth](../authentication/session-auth.md)

<quiz>
Your session-based API is vulnerable to CSRF. Which controls actually mitigate it? Select ALL that apply:

- [x] SameSite=Lax or Strict on the session cookie
- [ ] HttpOnly flag on the session cookie
- [x] A synchronizer (per-session) CSRF token required on mutating requests
- [x] Rejecting cross-origin requests without a valid `Origin` header

SameSite stops the browser attaching cookies to cross-site requests, synchronizer tokens prove the request originated from your page, and Origin validation is a legitimate server-side CSRF check. HttpOnly blocks XSS *cookie theft*, not CSRF — the browser still attaches the cookie to forged requests.
</quiz>

<quiz>
After login, your app keeps the same session ID it issued pre-authentication. What attack does this allow?

- [x] Session fixation — an attacker plants a known session ID on the victim and hijacks it once they log in
- [ ] CSRF — the pre-auth ID can be replayed cross-site
- [ ] Session hijacking via XSS
- [ ] None — the ID is only a random string until login

If the session ID survives the authentication transition unchanged, an attacker who can *set* the victim's ID beforehand (crafted link, subdomain, header injection) knows the session they'll land in post-login. The fix is a one-line habit: rotate the session ID at the moment of authentication.
</quiz>

<quiz>
You're setting the session cookie for `app.example.com`. Which attributes should it have? Select ALL that apply:

- [x] `HttpOnly`
- [x] `Secure`
- [x] `SameSite=Lax` (or `Strict`)
- [ ] `Domain=.example.com` so every subdomain shares the session

`HttpOnly` keeps the cookie away from JavaScript, `Secure` keeps it off plain HTTP, and `SameSite` stops it riding along on cross-site requests. Setting `Domain` widens the cookie to *every* subdomain — including marketing sites, user-content hosts, or a forgotten staging box — so any of them can read or overwrite it. Omit `Domain` to get the safest, host-only cookie.
</quiz>

<quiz>
Your session cookie is `HttpOnly`, but a pentester finds a stored XSS bug. What can the attacker still do?

- [ ] Nothing — `HttpOnly` neutralizes XSS for session-based apps
- [ ] Only read the session ID, not use it
- [x] Send authenticated requests from the victim's browser — the browser attaches the cookie automatically
- [ ] Only perform GET requests

`HttpOnly` stops the script from *reading* the cookie, so it can't be exfiltrated — but the injected script runs on your origin, and the browser still attaches the cookie to any request it makes. The attacker acts *as* the user for as long as the page is open. `HttpOnly` prevents theft, not abuse; CSP, output encoding, and XSS discipline are the real defense.
</quiz>

<quiz>
You scale a session-based app from one server to six behind a load balancer. Users get logged out randomly. What's the most robust fix?

- [ ] Move all session data into a client-side cookie with no signature, so any server can read it
- [x] Use a shared session store (e.g. Redis) with encrypted session data and short TTLs
- [ ] Increase cookie `Max-Age` to 30 days
- [ ] Disable session ID rotation at login so IDs stay stable

Each server had its own in-memory sessions, so requests routed elsewhere looked anonymous. A shared store fixes that while keeping instant server-side revocation; encrypting session data and keeping TTLs short limits damage if the store itself is breached. Unsigned client-side session data can be edited by the user, a longer `Max-Age` doesn't address the routing problem, and dropping rotation reopens session fixation.
</quiz>
