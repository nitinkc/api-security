# Knowledge Check

Production-grade scenarios, not trivia. Each topic has its own short quiz
so you can test yourself right after reading the matching page. Every
question presents a situation you'd actually hit building or reviewing an
API — pick the best answer(s), and read the explanation that appears after
you respond. Your progress is saved in the browser, per quiz.

## Authentication

| Quiz | Questions | Covers |
| --- | :-: | --- |
| [Basic Auth](basic-auth.md) | 5 | Transport, timing-safe comparison, logout, hashing cost |
| [Token Auth & API Keys](token-auth.md) | 5 | Key generation, storage, transport, leakage |
| [JWT](jwt.md) | 6 | Algorithm pinning, storage, signing keys, claim validation, revocation |
| [OAuth 2.0](oauth.md) | 6 | PKCE, `state`, redirect URIs, grant selection, token types, refresh rotation |
| [Session Auth](session-auth.md) | 5 | CSRF, fixation, cookie attributes, XSS, scaling |
| [MFA](mfa.md) | 5 | Factor categories, fatigue, step-up claims, phishing resistance, seed storage |
| [Enterprise SSO](enterprise-sso.md) | 5 | SAML vs OIDC, XSW, group mapping, LDAPS, offboarding |
| [Passkeys & WebAuthn](passkeys.md) | 5 | Origin binding, challenges, signCount, weakest-path fallbacks |
| [Mutual TLS](mtls.md) | 5 | Identity verification, cert lifetimes, gateway bypass, use cases |
| [Request Signing](request-signing.md) | 5 | Canonicalization, timing oracles, webhook verification, replay |

## Authorization

| Quiz | Questions | Covers |
| --- | :-: | --- |
| [Access-Control Models](models.md) | 5 | ACL, RBAC, ABAC, ReBAC, role explosion |
| [Scopes & Claims](scopes-claims.md) | 5 | Scope vs. user permission, token contents, `insufficient_scope` |
| [Object & Function Level](object-level.md) | 6 | BOLA, BFLA, BOPLA, nested and batch resources |
| [Policy Engines & Enforcement](policy-engines.md) | 5 | PEP/PDP/PIP, fail closed, gateway limits, list filtering |
| [Multi-Tenancy & Delegation](multi-tenancy.md) | 6 | Tenant derivation, row-level security, token exchange, impersonation |
| [Testing & Pitfalls](testing.md) | 6 | Two-user test, route inventory, TOCTOU, channel consistency |
| [Worked Example: RBAC API](rbac-design.md) | 5 | Permission matrix, visibility filters, SoD, 404 vs. 403 |

## Cross-cutting

| Quiz | Questions | Covers |
| --- | :-: | --- |
| [Cross-cutting Practices](cross-cutting.md) | 6 | Logging, rate limiting, credential lifetime, TLS, status codes, scheme selection |
