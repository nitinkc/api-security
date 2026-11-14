# JWT (JSON Web Tokens)

## What it is

A JWT ([RFC 7519](https://www.rfc-editor.org/rfc/rfc7519)) is a
**self-contained, cryptographically signed token**. Unlike an opaque API
key, the token itself carries the claims — user ID, roles, expiry — so the
server can verify it **without a database lookup**. Verification is pure
cryptography: check the signature, read the claims.

```mermaid
sequenceDiagram
    participant C as Client
    participant A as Auth Server
    participant R as Resource Server

    C->>A: POST /login (credentials)
    A->>A: Sign JWT with private key/secret
    A-->>C: eyJhbGciOi... (access token)

    C->>R: GET /api/data<br>Authorization: Bearer eyJhbGciOi...
    Note over R: Verify signature locally —<br>NO lookup needed
    R-->>C: 200 OK
```

## Anatomy of a JWT

Three Base64URL-encoded segments separated by dots:

```text
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJzdWIiOiJhbGljZSIsImlzcyI6Imh0dHBzOi8vYXV0aC5leGFtcGxlLmNvbSIsImV4cCI6MTczMTk1ODQwMH0
.
dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk...
```

| Segment | Decoded | Purpose |
| --- | --- | --- |
| Header | `{"alg":"RS256","typ":"JWT"}` | Signing algorithm and token type |
| Payload | `{"sub":"alice","iss":"...","exp":...}` | The claims — who, issued by, expiry |
| Signature | raw bytes | Signature over `header.payload` |

![JWT anatomy — three base64url segments, what's inside each, the per-segment traps, and the five-step verification pipeline](../images/jwt-anatomy.svg){ loading=lazy }

*Editable source: `docs/diagrams/jwt-anatomy.drawio`*

!!! danger "The payload is readable by anyone"
    Base64URL is encoding, not encryption. **Never** put passwords, PII you
    wouldn't want disclosed, or secret data in a JWT. Signature guarantees
    *integrity* (can't be tampered), not *confidentiality*. For encrypted
    tokens look at JWE — a different spec.

## Registered claims

Standardized in RFC 7519 — validate these on every request:

| Claim | Meaning | Validation rule |
| --- | --- | --- |
| `iss` | Issuer | Must equal your expected issuer |
| `sub` | Subject (user ID) | Present and well-formed |
| `aud` | Audience | Must include **your** API's identifier |
| `exp` | Expiration time | Must be in the future |
| `iat` | Issued at | Sanity-check it's not in the future |
| `nbf` | Not before | If present, must be in the past |
| `jti` | Unique token ID | Use for revocation/dedup tracking |

## Signing algorithms

| Alg | Type | When to use |
| --- | --- | --- |
| `HS256` | Symmetric HMAC | Single service that both issues and verifies tokens |
| `RS256` | RSA asymmetric | Distributed systems — many verifiers, one issuer (public key shared via JWKS) |
| `EdDSA` | Ed25519 asymmetric | Modern choice — fast, small keys, no padding pitfalls |

!!! tip "Asymmetric beats symmetric at scale"
    With `RS256`/`EdDSA`, resource servers hold only the **public** key —
    they can verify but never mint tokens. With `HS256`, every service that
    can verify can also forge tokens.

## Reference implementation

=== "Python (PyJWT)"

    ```python
    import datetime
    import jwt
    from jwt import PyJWKClient   # fetches + caches JWKS

    # ---- Issuing (auth server) ----
    now = datetime.datetime.now(datetime.timezone.utc)
    token = jwt.encode(
        {
            "sub": "alice",
            "iss": "https://auth.example.com",
            "aud": "orders-api",
            "iat": now,
            "exp": now + datetime.timedelta(minutes=15),
        },
        private_key_pem,           # RSA private key — never on clients
        algorithm="RS256",
    )

    # ---- Verifying (resource server) ----
    jwks = PyJWKClient("https://auth.example.com/.well-known/jwks.json")
    signing_key = jwks.get_signing_key_from_jwt(token)

    try:
        claims = jwt.decode(
            token,
            signing_key.key,
            algorithms=["RS256"],          # (1) pin the algorithm!
            audience="orders-api",          # (2)
            issuer="https://auth.example.com",
        )
    except jwt.PyJWTError:
        raise HTTPException(401, "Invalid token")
    ```

    1. Allowlisting `algorithms` prevents algorithm-confusion attacks.
    2. `aud` and `iss` checks stop tokens minted for other services being
       replayed against yours.

=== "Java (Spring Resource Server)"

    ```java
    // application.yml — that's all the wiring needed
    spring:
      security:
        oauth2:
          resourceserver:
            jwt:
              issuer-uri: https://auth.example.com
              audiences: orders-api
    ```

    ```java
    @Bean
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated())
            .oauth2ResourceServer(oauth -> oauth
                .jwt(Customizer.withDefaults()));
        return http.build();
    }
    // Spring automatically: fetches JWKS from the issuer, verifies the
    // signature, validates exp/iss/aud, and exposes claims via @AuthenticationPrincipal Jwt
    ```

=== "Client"

    ```bash
    curl -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIs..." \
         https://api.example.com/api/data
    ```

## Critical vulnerabilities

JWT's failure modes are mostly **implementation bugs**, not spec flaws.
These are the ones that show up in real breaches:

### 1. Algorithm confusion (`alg=none` and RS256→HS256)

The `alg` header is attacker-controlled input. Two classic exploits:

- **`alg: none`** — early libraries accepted unsigned tokens if the header
  said `none`. Modern libraries reject it; keep them patched.
- **RS256→HS256 confusion** — if the server expects `RS256` but a library
  accepts `HS256`, the attacker signs a forged token with HMAC using the
  **public key** (which is public!) as the HMAC secret. The verifier happily
  validates it.

**Mitigation:** allowlist `algorithms=["RS256"]` server-side. Never trust
the header's `alg` value to choose the verification method.

### 2. Weak HMAC secrets

`HS256` tokens are only as strong as the secret. Secrets like `secret`,
`changeme`, or short strings are brute-forced offline in seconds with tools
like `hashcat` (the attacker has the full token — verification is offline).

**Mitigation:** ≥256-bit random secrets from a secrets manager, or better —
move to asymmetric algorithms.

### 3. Missing claim validation

Verifying the signature but ignoring `exp`, `aud`, or `iss` means:
expired tokens keep working, and tokens minted for a *different* service
(a weaker one) get replayed against yours.

**Mitigation:** validate `exp`, `aud`, `iss` every time — most libraries do
it only if you ask.

### 4. The revocation problem

Statelessness cuts both ways: a stolen JWT is valid until `exp` — the
server can't "delete" it without keeping state.

**Mitigations, in order of pragmatism:**

1. **Short `exp`** (5–15 min) + refresh tokens — limits blast radius
2. **Blacklist `jti`** in Redis for high-risk operations
3. **Token version / `iat` floor** — bump a per-user `min_iat` on
   password change or incident response

### 5. Sensitive data in the payload

Anything in the payload is world-readable. Teams accidentally ship tokens
with emails, internal IDs, or (yes) password hashes inside.

### 6. Key management failures

Hardcoded keys in source, no rotation, JWKS endpoints fetched over HTTP.

**Mitigation:** keys in a secrets manager/KMS, publish via JWKS, rotate on
a schedule, `kid` header for key selection.

## Refresh tokens

Access tokens are short-lived by design; **refresh tokens** (long-lived,
opaque, single-use) are exchanged at the auth server for new access tokens.
Store refresh tokens hashed, rotate them on every use, and treat reuse of
an old refresh token as a theft signal (invalidate the whole family).

## Strengths & weaknesses

| Strengths | Weaknesses |
| --- | --- |
| Stateless verification — scales horizontally | Can't revoke without reintroducing state |
| Claims travel with the token — no lookup | Larger than opaque tokens (~1KB+ in headers) |
| Standardized — mature libraries in every language | Easy to misconfigure (the vulns above) |
| Asymmetric signing enables clean multi-service trust | Payload leaks data if you're careless |
| Works well for SPAs, mobile, microservices | Complexity: JWKS, rotation, refresh flows |

## When to use / avoid

!!! success "Use it for"
    - Microservice meshes where per-request DB lookups are too costly
    - SPAs and mobile apps (usually issued via OAuth 2.0 / OIDC)
    - Short-lived cross-service identity propagation

!!! failure "Avoid it for"
    - Long-lived tokens (the revocation problem bites hard)
    - Carrying sensitive data
    - Simple single-service APIs where a plain bearer token is enough —
      don't adopt JWT just because it's fashionable

## Checklist

- [x] `algorithms` allowlisted server-side — `alg` header never trusted
- [x] `exp` ≤ 15 min on access tokens; `aud`, `iss`, `exp` validated
- [x] Asymmetric alg (RS256/EdDSA) when more than one service verifies
- [x] No secrets or PII in the payload
- [x] Keys in KMS/secrets manager; published via JWKS; rotated regularly
- [x] Refresh tokens: hashed storage, rotation on use, reuse detection
- [x] JWT libraries kept patched (the `alg=none` era lives in old versions)
