# Quiz: JWT

Review first: [JWT](../authentication/jwt.md)

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

<quiz>
A service signs HS256 tokens with the secret `changeme123`. The tokens are never exposed publicly — only to logged-in users. How exploitable is this?

- [ ] Not exploitable — the secret is never sent over the network
- [ ] Only exploitable if the attacker can read the server's config
- [x] Highly exploitable — any user holding one token can brute-force the secret offline, then forge tokens for anyone
- [ ] Only exploitable if the token lacks an `exp` claim

Every token contains the signed `header.payload` and its signature, which is everything an attacker needs to test guesses offline with tools like `hashcat` — no requests to your server, no rate limits. A weak secret falls in seconds, and then the attacker can mint tokens with any `sub` or role. Use ≥256-bit random secrets from a secrets manager, or move to asymmetric signing.
</quiz>

<quiz>
A verifier checks the signature and `exp`, but ignores `aud` and `iss`. Your IdP issues tokens to several APIs. What's the realistic attack?

- [ ] An attacker can extend a token's expiry
- [x] A token issued for a different (possibly weaker) API is replayed against yours and accepted
- [ ] The token payload becomes readable
- [ ] The signature can be stripped

When one IdP serves many APIs, they share signing keys — so a token minted for `reporting-api` verifies perfectly at `payments-api` unless `aud` is checked. Skipping `iss` likewise lets tokens from another issuer you happen to trust slip through. Validate `exp`, `aud`, and `iss` on every request; most libraries only do so when you configure them to.
</quiz>

<quiz>
An attacker stole a user's JWT access token (1-hour `exp`). The user resets their password. Which measures actually cut off the stolen token before it expires? Select ALL that apply:

- [x] Check a per-user `min_iat` (token-version floor) that's bumped on password change
- [x] Check the token's `jti` against a denylist for revoked tokens
- [ ] Ask the client app to delete the token from its storage
- [x] Rotate the signing key — accepting that it invalidates every user's tokens

A JWT is valid until `exp` unless the verifier consults some state. A per-user `iat` floor and a `jti` denylist both add a small, targeted lookup; rotating the signing key works but logs out everyone. Deleting the token on the client does nothing about the attacker's copy. The everyday mitigation is keeping `exp` short (5–15 minutes) with refresh tokens, so the window is small to begin with.
</quiz>
