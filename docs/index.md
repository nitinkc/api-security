# API Security: Authentication

A language-agnostic field guide to the five dominant API authentication schemes.
Every scheme is explained systematically — how it works on the wire, reference
implementations (Python / Java), known weaknesses, common attacks, and when to
use or avoid it.

!!! info "Authentication vs. Authorization"
    **Authentication** answers *"who are you?"* — the subject of this guide.
    **Authorization** answers *"what are you allowed to do?"* — scopes,
    roles, and permissions layered on top. A correctly implemented
    authentication scheme is a prerequisite for authorization, not a
    substitute for it.

## The five schemes at a glance

| Scheme | Credential sent | Stateful? | Revocation | Typical use case |
| --- | --- | --- | --- | --- |
| [Basic Auth](authentication/basic-auth.md) | `Base64(username:password)` every request | No | Change the password | Internal tools, webhooks, dev/test |
| [Token Auth](authentication/token-auth.md) | API key / bearer token | Usually (lookup per request) | Delete/rotate token | Server-to-server, partner APIs |
| [JWT](authentication/jwt.md) | Signed self-contained token | No | TTL expiry or blacklist | Microservices, SPAs, mobile |
| [OAuth 2.0](authentication/oauth.md) | Access token issued via grant flows | Depends on token type | Revocation endpoint / TTL | Delegated access, 3rd-party integration |
| [Session Auth](authentication/session-auth.md) | Session ID cookie | Yes (server store) | Destroy session | Server-rendered web apps |

## How the guide is organized

<div class="grid cards" markdown>

-   :material-lock: **Basic Auth**

    ---

    The simplest scheme — credentials on every request. Learn why it's fine
    for a cron job and dangerous for the public internet.

    [:octicons-arrow-right-24: Read](authentication/basic-auth.md)

-   :material-key: **Token Auth**

    ---

    API keys and bearer tokens — opaque strings checked against a store.
    The workhorse of server-to-server authentication.

    [:octicons-arrow-right-24: Read](authentication/token-auth.md)

-   :material-certificate: **JWT**

    ---

    Cryptographically signed, self-contained tokens. Powerful — and the
    most commonly misconfigured scheme in this guide.

    [:octicons-arrow-right-24: Read](authentication/jwt.md)

-   :material-account-key: **OAuth 2.0**

    ---

    A delegation framework, not a single mechanism. Grant types, PKCE,
    access vs. refresh vs. ID tokens.

    [:octicons-arrow-right-24: Read](authentication/oauth.md)

-   :material-cookie: **Session Auth**

    ---

    The original web scheme: stateful sessions behind a cookie. Still the
    right answer for server-rendered applications.

    [:octicons-arrow-right-24: Read](authentication/session-auth.md)

-   :material-scale-balance: **Comparison**

    ---

    Side-by-side decision matrix and a selection flowchart for picking the
    right scheme for your API.

    [:octicons-arrow-right-24: Read](comparison.md)

</div>

## The golden rules

Regardless of which scheme you choose, these are non-negotiable:

1. **TLS everywhere.** Every scheme in this guide is trivially broken on
   plaintext HTTP. Terminate TLS correctly and enable HSTS.
2. **Never log credentials or tokens.** Scrub `Authorization` headers and
   cookies from access logs, error reports, and APM traces.
3. **Authenticate, then authorize.** Verifying identity says nothing about
   permissions — enforce scope/role checks on every endpoint.
4. **Rate-limit authentication endpoints.** Brute force and credential
   stuffing are the most common attacks against every scheme here.
5. **Prefer short-lived credentials.** The longer a stolen credential
   works, the worse the incident.

See [Security Best Practices](best-practices.md) for the full checklist.
