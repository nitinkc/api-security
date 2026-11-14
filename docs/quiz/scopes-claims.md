# Quiz: Scopes & Claims

Review first: [Scopes & Claims](../authorization/scopes-claims.md)

<quiz>
Alice is an org admin who may read and edit every calendar. She grants a third-party scheduling app `calendar:read`. What may the app do with her token?

- [ ] Read and edit every calendar — the token belongs to an admin
- [x] Read the calendars Alice is allowed to read, but not edit anything
- [ ] Read every calendar in every org that uses the API
- [ ] Read only Alice's personal calendar, whatever her permissions are

The effective permission is the **intersection** of what the user may do and what the client was granted. The scope limits the *app* to reading; Alice's permissions limit *which* calendars. Neither alone is the full answer, and the API must still apply its object-level checks.
</quiz>

<quiz>
A designer proposes scopes like `orders:42:read` and `orders:43:read` so tokens can grant access to specific orders. What's wrong with this?

- [ ] Nothing — it's the most precise least-privilege design
- [ ] Scopes can't contain numbers
- [x] Scopes are coarse, consent-level permissions; object IDs belong in a server-side check or RAR `authorization_details`
- [ ] It only works with opaque tokens, not JWTs

Scopes appear on consent screens and get baked into clients — they're meant to be few and stable (`orders:read`). Per-object scopes explode token size and can't follow ownership changes. Fine-grained grants like "this account, up to this amount" are what Rich Authorization Requests (RFC 9396) are for; ordinary ownership is a server-side check.
</quiz>

<quiz>
Which of these should be looked up server-side rather than carried in the access token? Select ALL that apply:

- [x] Ownership of individual objects
- [x] Permissions that must be revoked immediately
- [x] A user's 400 group memberships
- [ ] The caller's tenant / `org_id`

Tokens are a good home for stable, coarse claims — `sub`, tenant, audience, scope. Object ownership changes constantly and isn't the token's job, instant revocation is impossible with a snapshot, and hundreds of groups bloat the token past header size limits. Remember the payload is readable by anyone holding the token, so keep sensitive data out too.
</quiz>

<quiz>
A request arrives with a valid, unexpired token for your API, but it lacks the `orders:write` scope required by `POST /orders`. What's the correct response?

- [ ] `401 Unauthorized` with `WWW-Authenticate: Bearer error="invalid_token"`
- [x] `403 Forbidden` with `WWW-Authenticate: Bearer error="insufficient_scope"`
- [ ] `404 Not Found` to hide the endpoint
- [ ] `400 Bad Request`

The caller authenticated successfully — the token is fine — so this isn't a 401. The token simply doesn't carry enough privilege, which RFC 6750 §3.1 defines as `insufficient_scope` with status 403. The response can name the required scope so the client knows what to request.
</quiz>

<quiz>
A service reads the `roles` claim with `jwt.decode(token, options={"verify_signature": False})` "for speed, since the gateway already authenticated the request". What's the risk?

- [ ] None — decoding without verification is equivalent once a gateway is involved
- [ ] The `roles` claim will be missing
- [x] Anyone who can reach the service directly, or bypass the gateway, can forge any roles they like
- [ ] The token's `exp` will be extended

An unverified JWT is just JSON the caller wrote. Any path that skips the gateway — internal network access, a misrouted request, another compromised service — turns that into instant privilege escalation. Verify signature, `iss`, `exp`, and `aud` before reading a single claim; with a cached JWKS this costs microseconds.
</quiz>
