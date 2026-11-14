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

## Federation & directory

| Term | Meaning |
| --- | --- |
| SAML | XML-based SSO federation — assertions relayed through the browser to an ACS |
| ACS | Assertion Consumer Service — the SP endpoint that receives SAML assertions; must be exact-matched |
| XSW | XML Signature Wrapping — relocating the signed node so the forged assertion is consumed |
| JIT provisioning | Auto-creating accounts on first SSO login — default them to least privilege |
| LDAP / LDAPS | Directory protocol for user/group lookup and bind auth — only over TLS |
| AD / Kerberos | Microsoft's directory and its ticket-based internal SSO (SPNEGO/`Negotiate`) |
| IdP-initiated vs SP-initiated | Who starts the SSO flow — IdP-initiated skips the request and is easier to abuse |

## OAuth

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
| MFA / TOTP | Multi-factor auth; the time-based one-time-password flavor |
| Passkey / WebAuthn | Origin-bound public-key login — private key never leaves the device |
| FIDO2 / CTAP | The standard family / the device↔authenticator protocol behind passkeys |
| Relying party (RP) | The application a passkey or SAML assertion authenticates to |
| `acr` / `amr` | Token claims carrying authentication assurance level / methods used |
| signCount | Authenticator's clone-detection counter — regression means a copied key |
| Step-up authentication | Re-proving a factor inside a live session for high-risk actions |
| mTLS / SVID | Cert-on-both-sides TLS / the short-lived workload cert SPIRE issues |
| SPIFFE / SPIRE | Workload-identity URI standard / the runtime that issues SVIDs |
| SAN | Subject Alternative Name — cert field that carries SPIFFE IDs |
| Request signing | HMAC over a canonical request — proves origin + integrity without sending the secret |
| Nonce / event-ID dedup | Seen-once store that closes the replay window inside a timestamp |
| IAM | The umbrella discipline for identities, credentials, and permissions |
| SSO | Single Sign-On — one authentication event, many applications |

## Authorization

| Term | Meaning |
| --- | --- |
| Authorization | Deciding what an authenticated caller may do — distinct from proving who they are |
| 401 vs 403 | 401 = not authenticated; 403 = authenticated but not allowed |
| RBAC / ABAC / ACL | Role-, attribute-, or list-based authorization models |
| ReBAC | Relationship-based access control — permissions derived from a graph (Zanzibar-style) |
| Least privilege | Grant only the permissions, scope, and lifetime required for the task |
| Deny by default | Access is refused unless a rule explicitly allows it |
| BOLA / IDOR | Accessing another user's object by changing its ID — OWASP API #1 |
| BFLA | Calling an operation (often admin) the caller's role shouldn't allow — OWASP API #5 |
| BOPLA | Reading or writing object fields the caller shouldn't — OWASP API #3 |
| Mass assignment | Binding a request body onto a model so the caller sets privileged fields |
| PEP / PDP | Enforcement point that intercepts requests / decision point that evaluates policy |
| PIP / PAP | Attribute source for decisions / where policies are authored and managed |
| OPA / Rego | Open Policy Agent and its policy language |
| Cedar | Analyzable authorization policy language (Amazon Verified Permissions) |
| Audience (`aud`) | Token claim naming the API a token is valid for |
| Token exchange | RFC 8693 — swap a token for a narrower one to call a downstream service |
| `act` claim | Records the actor in a delegated token (who is acting for `sub`) |
| RLS | Row-level security — database-enforced row filtering, e.g. by tenant |
| Impersonation | An operator acting as a user; must be scoped, time-boxed, and audited |

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
