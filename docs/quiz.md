# Knowledge Check

Production-grade scenarios, not trivia. Each question presents a situation
you'd actually hit building or reviewing an API — pick the best answer(s),
and read the explanation that appears after you respond. Your progress is
saved in the browser as you go.

## Basic Auth

<quiz>
You're building a webhook endpoint that receives callbacks from a payment provider. The provider only supports Basic Auth over HTTPS. Which additional control matters MOST?

- [ ] Base64-encode the expected credentials before comparing them
- [x] Enforce an allowlist of the provider's source IPs and rate-limit the endpoint
- [ ] Rotate the password every 24 hours via an automated job
- [ ] Switch the endpoint to digest authentication

Basic Auth is acceptable for server-to-server webhooks because TLS hides the credentials and the caller set is tiny. The real risk is an attacker *finding* the endpoint and brute-forcing it — IP allowlisting plus rate limiting shrinks that surface. Base64 adds no protection (it's encoding, not secrecy), aggressive rotation breaks the provider, and digest auth isn't supported.
</quiz>

<quiz>
During a review you find this comparison in an auth check: `if (request.password == stored)`. What's the concrete vulnerability?

- [x] Timing attack — the comparison exits early on the first mismatching byte, leaking password content through latency
- [ ] The password is stored in plaintext
- [ ] The comparison is vulnerable to SQL injection
- [ ] Nothing — string equality is safe for credentials

Language-level string equality short-circuits on the first differing character, so response time reveals how much of the guess was right. Use a constant-time compare (Python `secrets.compare_digest`, Java `MessageDigest.isEqual`). The snippet doesn't reveal the storage format, so "plaintext" isn't supported by the evidence.
</quiz>

## Token Auth & API keys

<quiz>
Your mobile app ships with an embedded API key. A security review flags it as exposed. What's the correct fix?

- [ ] Obfuscate the key by splitting it across resources and code
- [x] Issue short-lived user-scoped tokens after login; treat any app-embedded secret as public
- [ ] Move the key into the app's native keystore so it can't be extracted
- [ ] Rotate the embedded key monthly via an app update

Anything shipped inside a client binary is extractable — obfuscation and keystore storage only raise the effort slightly, and rotation via app updates can't keep pace. The real fix is architectural: the app authenticates *users* and receives revocable, short-lived tokens instead of a static shared key.
</quiz>

<quiz>
A partner API accepts keys via `GET /data?apikey=...`. The provider asks how to reduce accidental key leakage. Select ALL that apply:

- [x] Move the key to an `Authorization` or `X-API-Key` request header
- [ ] Require the key to be sent in the request body instead
- [x] Strip the apikey query param in the gateway before it reaches origin logs
- [x] Ensure URLs are never written to access logs, browser history, or analytics

Query-string credentials land in logs, browser history, proxies, and referrer headers — headers avoid all of that. Stripping the param at the gateway keeps it out of origin logs. A request body doesn't work for GET and still isn't logged safely in most pipelines.
</quiz>

## JWT

<quiz>
A JWT verifier you're reviewing accepts whatever algorithm the token's `alg` header declares, including `"none"` and `HS256` — while the IdP signs with RS256. What attack does this enable? Select ALL that apply:

- [x] An attacker strips the signature entirely by sending alg=none
- [x] An attacker signs a forged token with the *public* key as the HMAC secret (alg confusion)
- [ ] An attacker can decrypt the token payload
- [ ] None — the algorithm header is integrity-protected by the signature

Two classic JWT exploits. `alg=none` yields an unsigned token a naive verifier accepts. RS256→HS256 confusion lets an attacker sign with the *public* key (which is public!) as the HMAC secret — the verifier uses it as an HMAC key and accepts the forgery. The fix: pin the expected algorithm server-side and ignore the header's claim. JWT payloads are only base64url-encoded — they're never encrypted unless you use JWE.
</quiz>

<quiz>
Your SPA stores JWT access tokens in localStorage "so they survive reloads". A pentester flags this. Why?

- [ ] localStorage tokens expire when the tab closes
- [ ] localStorage is sent automatically with cross-origin requests, enabling CSRF
- [x] Any XSS payload anywhere on the page can exfiltrate the token — localStorage is readable by all JS
- [ ] There's no risk — localStorage is origin-scoped

Origin-scoping is exactly the problem: every script running on your origin — including an injected XSS payload — reads localStorage freely. Tokens there are exfiltratable by any XSS. HttpOnly SameSite cookies keep tokens out of JS reach (and SameSite + CSRF tokens handle the forgery angle). LocalStorage is *not* sent automatically, and it persists across tab closes.
</quiz>

<quiz>
Your microservices all verify JWTs with a shared HS256 secret stored in each service's config. An architect suggests RS256 instead. What's the actual improvement?

- [x] Only the issuer holds the private key — a compromised verifier can't mint valid tokens
- [ ] RS256 signatures are faster to verify than HS256
- [ ] RS256 tokens are encrypted, so claims are hidden
- [ ] RS256 keys never expire

With HS256, every service that can *verify* a token can also *forge* one — the shared secret does both. RS256 separates signing (private key, held only by the issuer) from verification (public key, freely distributed via JWKS). HMAC is actually *faster* than RSA verification, signatures aren't encryption, and key lifetimes are a rotation policy, not an algorithm property.
</quiz>

## OAuth 2.0

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

## Session Auth

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

## Cross-cutting

<quiz>
Your access logs capture full request URIs and headers. Which of these must be scrubbed or excluded? Select ALL that apply:

- [x] Authorization header values
- [x] Session cookies
- [x] Query parameters containing tokens or API keys
- [ ] The HTTP method and path template

Logs are a credential-leak vector — anyone with log access (often broad) can replay captured secrets. Scrub auth headers, cookies, and sensitive query params. Method and route template are the *useful* parts of an access log — keep them (raw paths may still need parameter scrubbing).
</quiz>

<quiz>
An endpoint throttles to 5 failed logins per IP per minute. An attacker still stuffs 50k leaked credentials/day by rotating botnet IPs. Which control closes the gap the per-IP limit leaves open?

- [x] Rate-limit by account identity too — e.g. backoff after N failures for a given username
- [ ] Increase the per-IP limit to reduce false positives
- [ ] Add CAPTCHA only to the password-reset form
- [ ] Move the login endpoint behind a WAF

Per-IP limits fail against distributed attacks; per-*account* limits catch stuffing aimed at one identity regardless of source IP. You need both dimensions — and keep lockout windows short to avoid denial-of-service on the victim's account. A WAF is generic; the other options don't address distribution.
</quiz>

<quiz>
Which credential produces the smallest worst-case blast radius if stolen?

- [x] A 5-minute JWT access token — expires before most exfiltration completes
- [ ] A 30-day refresh token — long-lived but single-use and rotatable
- [ ] A session ID with idle-timeout sliding expiry
- [ ] A static API key rotated quarterly

Shorter-lived credentials bound the incident: a 5-minute token may expire before an attacker even exfiltrates it. Refresh tokens are riskier but rotation + reuse-detection catches theft. Sliding sessions outlive user attention. Static keys are the worst — a stolen key works for months unless someone notices and rotates it.
</quiz>

<quiz>
TLS terminates at your load balancer, then traffic continues to backends over plain HTTP "because it's the internal network". What's the honest assessment?

- [ ] Always fine — internal traffic can't be intercepted
- [x] It depends on the trust boundary — acceptable only where the internal segment is genuinely controlled; defense-in-depth favors mTLS inside
- [ ] Always a critical violation — HTTP inside a VPC is equivalent to public exposure
- [ ] Fine if the load balancer adds an X-Forwarded-For header

"Internal" is only as trustworthy as your segmentation. Flat networks, shared VPCs, compromised sidecars, and insider threats all make plaintext internal traffic interceptable — and compliance regimes (PCI, HIPAA) increasingly require it encrypted regardless. mTLS to backends is the current best practice where feasible. X-Forwarded-For adds metadata, not confidentiality.
</quiz>
