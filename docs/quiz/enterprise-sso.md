# Quiz: Enterprise SSO

Review first: [Enterprise SSO: SAML, OIDC & Directory](../authentication/enterprise-sso.md)

<quiz>
A customer asks whether to integrate your new SaaS app via SAML or OIDC. What's the right default recommendation?

- [ ] SAML — it's the enterprise standard and more secure
- [ ] Whichever their IdP supports; the protocols are equivalent
- [x] OIDC — it issues both ID tokens for login and access tokens for APIs, and fits SPAs/mobile; use SAML only if their IdP or a legacy app requires it
- [ ] Neither — connect directly to their LDAP directory

OIDC is OAuth 2.0-based: one integration gives you login, API access tokens, refresh flows, and first-class mobile support. SAML is solid but browser-only and can't issue API access tokens. Direct LDAP to a customer's directory is a non-starter for a SaaS product.
</quiz>

<quiz>
Your SAML service provider validates assertions with a generic XML parser you wrote — you check the signature is present and read the `NameID`. What's the exposure?

- [ ] None — you verified the signature exists
- [ ] Minor — attackers could only change cosmetic attributes
- [x] XML signature wrapping — an attacker inserts a forged assertion so the signature validates the original while your code consumes the forged one
- [ ] Only performance issues from large XML documents

Signature presence ≠ signature coverage. XSW rearranges the document so a verifier signs off on one node while the application reads another. Defeating it requires strict schema validation and comparing the signed reference to the consumed node — exactly what maintained SAML libraries do and hand-rolled parsers get wrong.
</quiz>

<quiz>
After login via SAML, your app auto-provisions the user and maps the assertion's `groups` attribute directly onto your application's role list — `groups: ["Domain Admins"]` becomes admin in your app. What's wrong? Select ALL that apply:

- [x] IdP groups don't necessarily map 1:1 to your app's roles — you need an explicit allowlist mapping
- [x] Assertion attributes could carry attacker-influenced values if group membership is self-service in their IdP
- [ ] Nothing — the assertion is signed by a trusted IdP
- [x] JIT-provisioned users should land in a least-privilege default role until mapping is reviewed

A signed assertion proves the IdP *sent* the claims, not that the claims mean what your app assumes. IdP group names are coarse, namespaced differently, and sometimes user-joinable. Map explicitly, default low, and keep authorization decisions in your own policy layer.
</quiz>

<quiz>
An internal legacy app authenticates users by attempting an LDAP bind with the submitted credentials over `ldap://ad.internal:389`. What's the primary problem?

- [ ] LDAP bind is deprecated — it never works correctly
- [ ] Port 389 is the wrong port — binds require port 636 specifically
- [x] Credentials cross the network in plaintext — `ldap://` is unencrypted; use `ldaps://` or StartTLS, and note the app still *sees* the plaintext password
- [ ] LDAP bind gives attackers read access to the entire directory

Plain LDAP sends the bind DN and password unencrypted — anyone on the segment harvests AD credentials, the crown jewels of the enterprise. LDAPS (or StartTLS) fixes transit, but the deeper issue remains: the app handles raw passwords, so prefer federating (Kerberos/SPNEGO internally, OIDC/SAML elsewhere) over bind-auth wherever possible.
</quiz>

<quiz>
A user is disabled in Active Directory at 10:00, but keeps calling your API successfully until 11:00. Tokens are JWTs with 60-minute expiry, verified only by signature. How do you close the gap? Select ALL that apply:

- [x] Shorten access-token TTL (e.g. 5–15 minutes) so staleness is bounded
- [x] Have the authorization server check account status when refreshing tokens
- [ ] Switch from JWT to Basic Auth so the password is checked each time
- [x] Add a revocation/introspection check on high-risk endpoints

Signature verification proves the token was issued, not that the account still exists. The standard fix is short TTLs plus refresh-time account checks, so offboarding takes effect within minutes. For sensitive endpoints, a per-request status check (introspection or a `min_iat` floor per user) closes it entirely — and Basic Auth is a step backwards, not a fix.
</quiz>
