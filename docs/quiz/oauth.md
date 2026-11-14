# Quiz: OAuth 2.0

Review first: [OAuth 2.0](../authentication/oauth.md)

<quiz>
A colleague builds an auth-code-flow integration for a public SPA *without* PKCE, arguing "it's only needed for mobile apps". Who's right?

- [ ] The colleague — PKCE is a mobile-only extension
- [x] Neither — PKCE is required for all public clients; an intercepted auth code is redeemable without the verifier
- [ ] They're both right — auth codes can't be intercepted over TLS
- [ ] The colleague, partially — PKCE is optional if `state` is used

RFC 9700 and the OAuth 2.1 draft make PKCE mandatory for public clients, full stop — SPAs included. The authorization code travels through the browser redirect, where history, extensions, or OS handlers can leak it; without the verifier an attacker who captured the code exchanges it for tokens. `state` defends against CSRF on the redirect, not code interception — they're orthogonal.
</quiz>

<quiz>
Which OAuth artifact should an API *never* accept as proof of identity for resource access?

- [ ] The access token
- [x] The ID token — it authenticates the user to the client, not to the API
- [ ] The client certificate
- [ ] The `aud` claim itself

ID tokens are an audience-of-the-client artifact: they tell the SPA "who logged in", not "what may this bearer do". APIs must require access tokens carrying appropriate scopes and audience — identity-vs-authorization confusion is a classic exam trap and a real-world bug.
</quiz>

<quiz>
A client registered its redirect URI as `https://*.client.com/callback` "to support preview environments". What's the risk?

- [ ] None — the wildcard is still restricted to the client's domain
- [x] Any subdomain an attacker can control or take over (or an open redirector on one) can receive authorization codes
- [ ] Wildcards disable PKCE for that client
- [ ] Wildcards cause the `state` parameter to be ignored

The redirect URI is where the authorization server sends the code. Wildcards and prefix matching widen that target to every forgotten preview site, dangling DNS record, or open redirect under the domain. Authorization servers must match redirect URIs **exactly**; register each environment explicitly instead.
</quiz>

<quiz>
What does the `state` parameter in the authorization request protect against?

- [ ] Interception of the authorization code by a malicious app
- [ ] Replay of a stolen access token
- [x] CSRF on the redirect — an attacker completing *their* login flow in the victim's browser
- [ ] Leakage of the client secret

`state` is a random value the client generates, stores, and checks on the callback. Without it, an attacker can trick a victim's browser into finishing a flow the attacker started, linking the victim's session to the attacker's account. Code interception is what PKCE handles; the two protections are complementary.
</quiz>

<quiz>
A smart TV app and a developer CLI both need to call an API on behalf of a signed-in user. Neither has a convenient browser or keyboard. Which grant fits?

- [ ] Resource Owner Password — let the user type their password into the app
- [ ] Implicit — return the token directly in the redirect
- [ ] Client Credentials — authenticate as the app itself
- [x] Device Authorization (RFC 8628) — the user approves on a second device using a short code

The device grant shows a code and URL; the user signs in on their phone or laptop while the device polls for the token. The password grant is deprecated because it hands user credentials to the client, Implicit is deprecated because it leaks tokens in URLs, and Client Credentials has no user — it authenticates the application, not a person.
</quiz>

<quiz>
Your authorization server rotates refresh tokens on every use. It receives a refresh token that was *already* exchanged an hour ago. What should it do?

- [ ] Accept it — the user may simply have retried a request
- [ ] Reject that request only, and keep the newer refresh token valid
- [x] Treat it as a theft signal: reject it and revoke the entire refresh-token family for that grant
- [ ] Issue a new access token but no new refresh token

With rotation, each refresh token should be used once. A reused old token means two parties hold copies — the legitimate client and an attacker — and the server can't tell which is which. Revoking the whole family forces a fresh login and cuts the attacker off, whichever copy they hold.
</quiz>
