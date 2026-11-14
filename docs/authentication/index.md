# Authentication Schemes

Each page in this section follows the same structure so you can compare
schemes directly:

1. **What it is** — definition and the governing specification
2. **How it works** — the request/response flow on the wire
3. **Reference implementation** — working code in Python and Java
4. **Strengths & weaknesses** — honest trade-offs
5. **Common attacks & mitigations** — what goes wrong in production
6. **When to use / avoid** — decision guidance

| Page | One-line summary |
| --- | --- |
| [Basic Auth](basic-auth.md) | `username:password`, Base64-encoded, sent on every request |
| [Token Auth](token-auth.md) | Opaque key or bearer token validated against a server-side store |
| [JWT](jwt.md) | Self-contained signed claims; verified without a server lookup |
| [OAuth 2.0](oauth.md) | Delegated authorization framework issuing access tokens |
| [Session Auth](session-auth.md) | Stateful server-side session referenced by a cookie |
| [MFA](mfa.md) | Two+ factor categories at login; `acr`/`amr` claims carry the result to APIs |
| [Enterprise SSO](enterprise-sso.md) | SAML/OIDC federation plus LDAP/AD/Kerberos directory backends |
| [Passkeys & WebAuthn](passkeys.md) | Origin-bound public-key credentials — phishing-resistant by construction |
| [Mutual TLS](mtls.md) | Both parties present certificates; workload identity via SPIFFE/SPIRE |
| [Request Signing](request-signing.md) | HMAC over a canonical request — webhook verification and replay defense |
