# Quiz: Cross-cutting Practices

Review first: [Security Best Practices](../best-practices.md) and
[Comparison & Selection](../comparison.md)

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

<quiz>
A request arrives with an expired access token. Another arrives with a valid token whose user lacks permission for the action. Which status codes are correct?

- [ ] 403 for both — both are access failures
- [ ] 401 for both — both involve the token
- [x] 401 for the expired token (authentication failed); 403 for the valid token without permission (authorization failed)
- [ ] 400 for the expired token; 401 for the missing permission

401 means "I don't know who you are" — missing, malformed, or expired credentials — and should prompt the client to re-authenticate. 403 means "I know who you are, and the answer is no." Returning 404 instead of 403 is a legitimate choice when you don't want to confirm an object exists.
</quiz>

<quiz>
Which scenario-to-scheme pairings match current recommendations? Select ALL that apply:

- [x] Server-rendered monolith → session auth with CSRF tokens
- [x] Microservice → microservice → JWT (RS256 from your IdP) or mTLS
- [ ] SPA + your own backend → OAuth Implicit grant
- [x] Mobile app + your backend → OAuth/OIDC issuing short-lived access tokens plus refresh tokens

Sessions fit browsers talking to a single server-rendered app; asymmetric JWTs or mTLS fit service-to-service trust; mobile apps use OIDC with short-lived tokens. The Implicit grant is deprecated — SPAs should use Authorization Code + PKCE or a backend-for-frontend with cookies.
</quiz>
