<!-- Auto-appended to every page via pymdownx.snippets auto_append.
     Each entry produces a dotted-underline hover tooltip.
     Keep definitions to one line. -->

*[HTTPS]: HTTP carried over a TLS-encrypted connection
*[URI]: Uniform Resource Identifier
*[URL]: Uniform Resource Locator
*[TLS]: Transport Layer Security — the encryption protocol protecting data in transit
*[SSL]: Secure Sockets Layer — deprecated predecessor of TLS
*[mTLS]: Mutual TLS — both client and server present certificates
*[HSTS]: HTTP Strict Transport Security — forces browsers to use HTTPS only
*[SNI]: Server Name Indication — TLS extension naming the requested host
*[CA]: Certificate Authority — issues and signs TLS certificates
*[CRL]: Certificate Revocation List — CA-published list of revoked certificates
*[OCSP]: Online Certificate Status Protocol — real-time certificate validity check
*[JWT]: JSON Web Token — a signed, self-contained credential (RFC 7519)
*[JWS]: JSON Web Signature — the signing format inside a JWT
*[JWE]: JSON Web Encryption — the encryption format for JWTs
*[JWK]: JSON Web Key — JSON representation of a cryptographic key
*[JWKS]: JSON Web Key Set — published collection of JWKs enabling key rotation
*[kid]: Key ID — JWT header claim selecting which key to verify with
*[alg]: Algorithm — JWT header claim naming the signature algorithm
*[HS256]: HMAC-SHA256 — symmetric JWT signing (shared secret)
*[RS256]: RSA-SHA256 — asymmetric JWT signing (private signs, public verifies)
*[ES256]: ECDSA P-256/SHA-256 — asymmetric JWT signing, short signatures
*[HMAC]: Hash-based Message Authentication Code — keyed-hash integrity check
*[SHA]: Secure Hash Algorithm — family of cryptographic hash functions
*[RSA]: Rivest–Shamir–Adleman — asymmetric public-key algorithm
*[ECDSA]: Elliptic Curve Digital Signature Algorithm
*[AES]: Advanced Encryption Standard — symmetric block cipher
*[GCM]: Galois/Counter Mode — AES mode providing authenticated encryption
*[IV]: Initialization Vector — nonce that prevents repeated ciphertexts
*[KDF]: Key Derivation Function — stretches a secret into a stronger key
*[PBKDF2]: Password-Based Key Derivation Function 2 — password-stretching KDF
*[CSPRNG]: Cryptographically Secure Pseudo-Random Number Generator
*[UUID]: Universally Unique Identifier — 128-bit ID; v4 is random
*[OAuth]: Open Authorization — delegated-access framework (RFC 6749)
*[OIDC]: OpenID Connect — identity layer on top of OAuth 2.0
*[PKCE]: Proof Key for Code Exchange — protects auth-code flow for public clients
*[SAML]: Security Assertion Markup Language — XML-based SSO federation
*[SSO]: Single Sign-On — one login reused across applications
*[IdP]: Identity Provider — the party that authenticates users
*[SP]: Service Provider — the application relying on an IdP
*[MFA]: Multi-Factor Authentication — two or more distinct factors
*[OTP]: One-Time Password — a credential valid for a single use
*[TOTP]: Time-based One-Time Password — authenticator-app codes (RFC 6238)
*[LDAP]: Lightweight Directory Access Protocol — directory lookup/bind protocol
*[LDAPS]: LDAP over TLS — the only acceptable form on a network
*[AD]: Active Directory — Microsoft's directory and Kerberos KDC
*[KDC]: Key Distribution Center — Kerberos ticket issuer
*[SPNEGO]: Negotiation mechanism wrapping Kerberos/NTLM in HTTP auth
*[ACS]: Assertion Consumer Service — the SAML SP endpoint receiving assertions
*[XSW]: XML Signature Wrapping — SAML attack relocating the signed node
*[JIT]: Just-In-Time provisioning — auto-creating accounts at first SSO login
*[WebAuthn]: Web Authentication — browser API for passkey ceremonies
*[FIDO2]: The WebAuthn+CTAP standard family for passwordless auth
*[CTAP]: Client-to-Authenticator Protocol — device↔authenticator channel
*[RP]: Relying Party — the application a passkey authenticates to
*[acr]: Authentication Context Class Reference — assurance-level token claim
*[amr]: Authentication Methods Reference — "how the user authed" token claim
*[SVID]: SPIFFE Verifiable Identity Document — a short-lived workload cert
*[SPIFFE]: Secure Production Identity Framework — workload-identity URI standard
*[SPIRE]: SPIFFE Runtime Environment — issues/rotates SVIDs to workloads
*[SAN]: Subject Alternative Name — cert field carrying hostnames/SPIFFE IDs
*[SigV4]: AWS Signature Version 4 — canonical request-signing scheme
*[AiTM]: Adversary-in-the-Middle — real-time phishing relay defeating OTPs
*[CSRF]: Cross-Site Request Forgery — attack that rides a user's ambient session
*[XSS]: Cross-Site Scripting — injecting attacker script into a page
*[CORS]: Cross-Origin Resource Sharing — browser relaxation of same-origin rules
*[CSP]: Content Security Policy — browser policy restricting script/resource origins
*[SOP]: Same-Origin Policy — browser isolation between origins
*[MITM]: Man-in-the-Middle — attacker positioned between client and server
*[WAF]: Web Application Firewall — filters malicious HTTP traffic
*[DDoS]: Distributed Denial of Service — traffic flood exhausting capacity
*[RBAC]: Role-Based Access Control — permissions grouped by role
*[ABAC]: Attribute-Based Access Control — permissions from attributes
*[ACL]: Access Control List — per-resource list of allowed principals
*[ReBAC]: Relationship-Based Access Control — permissions derived from a relationship graph
*[BOLA]: Broken Object Level Authorization — accessing another user's object by ID (OWASP API #1)
*[IDOR]: Insecure Direct Object Reference — older name for BOLA
*[BFLA]: Broken Function Level Authorization — invoking operations the role shouldn't allow (OWASP API #5)
*[BOPLA]: Broken Object Property Level Authorization — reading/writing fields the caller shouldn't (OWASP API #3)
*[PEP]: Policy Enforcement Point — intercepts requests and enforces decisions
*[PDP]: Policy Decision Point — evaluates policy and returns allow/deny
*[PIP]: Policy Information Point — supplies attributes for authorization decisions
*[PAP]: Policy Administration Point — where policies are authored and managed
*[OPA]: Open Policy Agent — general-purpose policy engine (Rego language)
*[RLS]: Row-Level Security — database-enforced per-row access filtering
*[RAR]: Rich Authorization Requests — structured OAuth authorization details (RFC 9396)
*[XACML]: eXtensible Access Control Markup Language — OASIS ABAC policy standard
*[DTO]: Data Transfer Object — explicit request/response shape decoupled from the entity
*[TOCTOU]: Time-Of-Check to Time-Of-Use — race between a check and the action it guards
*[IAM]: Identity and Access Management — systems governing identities & permissions
*[KMS]: Key Management Service — managed key storage and rotation
*[HSM]: Hardware Security Module — tamper-resistant key hardware
*[PII]: Personally Identifiable Information
*[PHI]: Protected Health Information — HIPAA-regulated health data
*[CVE]: Common Vulnerabilities and Exposures — cataloged vulnerability IDs
*[CVSS]: Common Vulnerability Scoring System — standardized severity scoring
*[NIST]: National Institute of Standards and Technology — US standards body
*[IETF]: Internet Engineering Task Force — publishes RFC standards
*[TTL]: Time To Live — how long a credential or cache entry remains valid
*[SLA]: Service Level Agreement — contractual uptime/latency commitment
