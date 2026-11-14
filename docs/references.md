# Standards & References

## Core specifications

| Standard | Title | Covers |
| --- | --- | --- |
| [RFC 7617](https://www.rfc-editor.org/rfc/rfc7617) | The 'Basic' HTTP Authentication Scheme | Basic Auth |
| [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750) | OAuth 2.0 Bearer Token Usage | `Authorization: Bearer` |
| [RFC 7519](https://www.rfc-editor.org/rfc/rfc7519) | JSON Web Token (JWT) | Token format & claims |
| [RFC 7515](https://www.rfc-editor.org/rfc/rfc7515) | JSON Web Signature (JWS) | JWT signing |
| [RFC 6749](https://www.rfc-editor.org/rfc/rfc6749) | OAuth 2.0 Authorization Framework | Grant types |
| [RFC 7636](https://www.rfc-editor.org/rfc/rfc7636) | PKCE | Public-client code flow |
| [RFC 8628](https://www.rfc-editor.org/rfc/rfc8628) | Device Authorization Grant | TVs/CLIs |
| [RFC 9700](https://www.rfc-editor.org/rfc/rfc9700) | OAuth 2.0 Security Best Current Practice | Modern OAuth hardening — read this |
| [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html) | Identity layer on OAuth 2.0 | `id_token`, userinfo |

## Authorization specifications

| Standard | Title | Covers |
| --- | --- | --- |
| [RFC 8693](https://www.rfc-editor.org/rfc/rfc8693) | OAuth 2.0 Token Exchange | Delegation, `act` claim, downscoping |
| [RFC 8707](https://www.rfc-editor.org/rfc/rfc8707) | Resource Indicators for OAuth 2.0 | Audience-restricted tokens |
| [RFC 9068](https://www.rfc-editor.org/rfc/rfc9068) | JWT Profile for OAuth 2.0 Access Tokens | `scope`, `roles`, `groups` claims |
| [RFC 9396](https://www.rfc-editor.org/rfc/rfc9396) | OAuth 2.0 Rich Authorization Requests | Fine-grained `authorization_details` |
| [NIST SP 800-162](https://csrc.nist.gov/pubs/sp/800/162/upd2/final) | Guide to Attribute Based Access Control | ABAC concepts |
| [XACML 3.0](https://docs.oasis-open.org/xacml/3.0/xacml-3.0-core-spec-os-en.html) | eXtensible Access Control Markup Language | PEP/PDP/PIP/PAP architecture |
| [Zanzibar (USENIX ATC '19)](https://www.usenix.org/conference/atc19/presentation/pang) | Google's Consistent, Global Authorization System | ReBAC |

## Security guidance

- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/) —
  the canonical API threat list (BOLA/IDOR is #1)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [OWASP Mass Assignment Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)
- [OWASP Multi-Tenant Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multi_Tenant_Security_Cheat_Sheet.html)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP JSON Web Token Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
- [NIST SP 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html) —
  Digital identity / authentication guidelines
- [jwt.io](https://jwt.io) — decode/inspect JWTs; library directory

## Useful tooling

| Tool | Purpose |
| --- | --- |
| `hashcat` / `jwt_tool` | Test HMAC secret strength & JWT attacks (authorized testing only) |
| `gitleaks`, `trufflehog` | Scan repos for committed credentials |
| `cURL`, `httpie` | Wire-level auth testing |
| Keycloak | Self-hosted OAuth/OIDC provider for development |
| `oauth2-proxy` | Put OAuth in front of apps that lack auth |
| Open Policy Agent, Cedar | Policy engines (ABAC/RBAC) |
| OpenFGA, SpiceDB | Zanzibar-style relationship-based authorization |
| Burp Suite Autorize, OWASP ZAP | Replay traffic with low-privilege tokens to find BOLA/BFLA (authorized testing only) |

## Library cheat sheet

| Language | JWT | OAuth/OIDC client | Sessions |
| --- | --- | --- | --- |
| Python | PyJWT, `python-jose` (unmaintained — prefer PyJWT) | Authlib, `requests-oauthlib` | Flask-Session, Django sessions |
| Java | `java-jwt`, Nimbus JOSE+JWT | Spring Security OAuth2 | Spring Session, `HttpSession` |
| Node | `jsonwebtoken`, `jose` (preferred) | `openid-client`, Passport | `express-session` |
| Go | `golang-jwt`, `lestrrat-go/jwx` | `golang.org/x/oauth2` | `gorilla/sessions` |
