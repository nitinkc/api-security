# Glossary

Every term here also renders as a **hover tooltip** throughout the site —
hover any dotted-underlined abbreviation (e.g. TLS, JWT, PKCE) to see its
definition inline.

## Transport security

| Term | Meaning |
| --- | --- |
| TLS | Transport Layer Security — encrypts and authenticates connections; successor to SSL |
| mTLS | Mutual TLS — both parties present certificates; used for service-to-service auth |
| HSTS | HTTP Strict Transport Security — tells browsers to refuse plaintext HTTP |
| CA | Certificate Authority — trusted issuer of TLS certificates |
| OCSP / CRL | Certificate revocation mechanisms — real-time check vs. published list |
| SNI | Server Name Indication — lets one IP serve certs for many hostnames |

## Tokens & signatures

| Term | Meaning |
| --- | --- |
| JWT | JSON Web Token — base64url header.payload.signature; verified, not decrypted |
| JWS / JWE | The signature and encryption containers defined alongside JWT |
| JWK / JWKS | JSON keys; a JWKS endpoint publishes rotating verification keys |
| kid / alg | JWT header fields — which key, which algorithm |
| HS256 vs RS256/ES256 | Shared-secret HMAC vs. asymmetric signatures — asymmetric allows public verifiers |
| Access token | Credential presented to the API; usually short-lived |
| Refresh token | Longer-lived credential exchanged for new access tokens |
| ID token | OIDC token carrying identity claims to the *client*, never to APIs |

## OAuth & federation

| Term | Meaning |
| --- | --- |
| OAuth 2.0 | Delegated-authorization framework — grants scoped access without sharing passwords |
| OIDC | Identity layer on OAuth 2.0 — adds login + ID token |
| PKCE | Code-verifier challenge making the auth-code flow safe for public clients |
| Grant type | The flow used to obtain a token — auth code, client credentials, etc. |
| Scope | Permission claim on a token — least-privilege unit of authorization |
| IdP / SP | Identity Provider authenticates; Service Provider consumes |
| SAML | XML-based enterprise SSO federation — older alternative to OIDC |

## Attacks & defenses

| Term | Meaning |
| --- | --- |
| MITM | Interception between client and server — the attack TLS exists to stop |
| CSRF | Forged cross-site requests riding ambient cookies — blocked by same-site cookies + tokens |
| XSS | Injected attacker script — can steal tokens held in web storage |
| Credential stuffing | Replay of breached username/password pairs — countered by rate limits + MFA |
| Timing attack | Inferring secrets from comparison latency — fixed by constant-time compare |
| Alg confusion | Accepting `alg=none` or HS256-vs-RS256 swap in JWT verification |
| Session fixation | Forcing a known session ID on a victim — fixed by rotating ID at login |
| DDoS / rate limiting | Availability attack / per-client throttling defense |

## Identity & access

| Term | Meaning |
| --- | --- |
| RBAC / ABAC / ACL | Role-, attribute-, or list-based authorization models |
| MFA / TOTP | Multi-factor auth; the time-based one-time-password flavor |
| IAM | The umbrella discipline for identities, credentials, and permissions |
| SSO | Single Sign-On — one authentication event, many applications |

## Crypto hygiene

| Term | Meaning |
| --- | --- |
| HMAC | Keyed-hash message authentication — how HS256 JWTs are signed |
| bcrypt / argon2 / PBKDF2 | Slow, salted password-hashing functions — required for stored passwords |
| CSPRNG | Crypto-grade randomness — required for tokens and session IDs |
| KMS / HSM | Managed or hardware key storage — where signing keys should live |
| Salt / pepper | Per-user public randomness vs. app-wide secret added before hashing |

## Compliance & standards

| Term | Meaning |
| --- | --- |
| OWASP | The project behind the Top 10 and ASVS standards |
| CVE / CVSS | Cataloged vulnerability IDs and their severity scores |
| NIST | US standards body — source of password & crypto guidance (SP 800-63B) |
| PII / PHI | Personal data; health-regulated data — both demand encryption & audit |
| RFC / IETF | The documents and the body that standardize protocols like OAuth |
