# Quiz: Object & Function Level

Review first: [Object & Function Level](../authorization/object-level.md)

<quiz>
`GET /api/invoices/{id}` verifies the JWT signature, expiry, and that the token has the `invoices:read` scope, then returns the invoice by ID. A pentester reports a critical finding. What's most likely?

- [ ] The scope name should be `read:invoices` to follow convention
- [x] BOLA — any user with `invoices:read` can read any other user's invoice by changing the ID
- [ ] The token should be validated against the IdP on every request
- [ ] Invoice IDs should be UUIDs instead of integers

The scope proves the *client* may read invoices in general; it says nothing about *which* invoices. Without scoping the lookup by the caller's identity or tenant, every authenticated user can enumerate everyone's data — OWASP API #1. UUIDs make guessing harder but IDs leak through URLs, logs, and other responses, so they aren't an authorization control.
</quiz>

<quiz>
`PATCH /users/me` binds the JSON body directly onto the user entity and saves it. Which request demonstrates the vulnerability?

- [ ] `{"display_name": "<script>alert(1)</script>"}`
- [x] `{"display_name": "Alice", "role": "admin"}`
- [ ] `PATCH /users/42` with Alice's token
- [ ] `{"display_name": "Alice"}` sent without a Content-Type header

This is mass assignment — Broken Object Property Level Authorization (OWASP API #3). Because every matching field is bound, the caller can set `role`, `tenant_id`, or balances. The fix is an explicit input DTO that contains only user-editable fields and rejects unknown ones. The XSS payload is an output-encoding problem; `/users/42` would be BOLA, a different endpoint.
</quiz>

<quiz>
The admin UI only shows the "Delete user" button to admins. A regular user reads the JavaScript bundle, finds `DELETE /api/users/{id}`, and calls it successfully. Which vulnerability is this?

- [ ] BOLA
- [x] BFLA — Broken Function Level Authorization
- [ ] BOPLA
- [ ] CSRF

The user is attacking an *operation* they should never invoke, not a specific record or field — that's BFLA (OWASP API #5). Hiding UI elements isn't authorization; JavaScript bundles, OpenAPI specs, and REST conventions reveal the endpoints. Every route needs a server-side function-level check, with a deny-by-default router for anything unmapped.
</quiz>

<quiz>
`GET /projects/{pid}/tasks/{tid}` checks that the caller is a member of project `pid`, then loads task `tid` by its ID. What's the gap?

- [ ] None — project membership implies access to its tasks
- [x] Task `tid` may belong to a different project; the attacker passes a project they're in plus a task from one they're not
- [ ] The route should use query parameters instead of path segments
- [ ] Task IDs should be scoped per project so they can't collide

Checking the parent doesn't prove the child belongs to it. The lookup must be `WHERE id = :tid AND project_id = :pid` (with `pid` already verified against the caller) — verify the child belongs to the parent *and* the parent to the caller.
</quiz>

<quiz>
A `GET /users/{id}` handler returns the ORM entity directly. The frontend only displays name and avatar, but the JSON also contains `password_hash`, `email`, and `internal_risk_score`. How should this be fixed?

- [ ] Have the frontend delete the extra fields before rendering
- [ ] Encrypt the sensitive columns in the database
- [x] Return an explicit output DTO with only the fields this caller may see
- [ ] Minify the JSON so the fields are harder to spot

Every field in a response is visible to the caller, whatever the UI shows — this is excessive data exposure, the read side of BOPLA. An explicit output schema (per role where needed) means columns added later can't leak by default. Encryption at rest doesn't help once the API decrypts and serializes the value.
</quiz>

<quiz>
Your REST `GET /orders/{id}` correctly checks ownership. Which other paths commonly skip the same check? Select ALL that apply:

- [x] A batch endpoint like `GET /orders?ids=1,2,3`
- [x] A GraphQL `node(id:)` query resolving orders
- [x] `PUT` and `DELETE /orders/{id}` handlers written later
- [ ] The `OPTIONS` CORS preflight for `/orders/{id}`

Object checks often protect only the endpoint that was reviewed. Batch endpoints and GraphQL resolvers fetch many objects through separate code paths, and write verbs are frequently added without the same guard. Authorize every object returned or modified, in a shared loader or repository layer. CORS preflights return no resource data.
</quiz>
