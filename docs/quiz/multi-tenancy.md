# Quiz: Multi-Tenancy & Delegation

Review first: [Multi-Tenancy & Delegation](../authorization/multi-tenancy.md)

<quiz>
A multi-tenant API reads the tenant from an `X-Tenant-ID` header set by the frontend and filters queries by it. What's the right fix? Select ALL that apply:

- [x] Derive the tenant from the verified access token, and reject requests where a supplied tenant disagrees
- [x] Add database row-level security keyed on a transaction-local tenant setting as a backstop
- [ ] Sign the `X-Tenant-ID` header value with a shared secret embedded in the frontend
- [x] Include the tenant in cache keys so cached objects can't cross tenants

Any client-supplied tenant is attacker-controlled. The tenant must come from the verified credential; row-level security catches the query that forgets the filter; and cache keys without the tenant are a classic cross-tenant leak. A secret embedded in a frontend is public, so signing with it proves nothing.
</quiz>

<quiz>
Service A receives a user's access token (audience `api-a`, scopes `orders:read orders:write payments:refund`) and needs to call service B to read the user's orders. What's the recommended approach?

- [ ] Forward the user's token unchanged so B sees the original identity
- [ ] Call B with A's own service-account token and pass the user ID in a header
- [x] Use OAuth token exchange to get a token for audience `api-b` with only `orders:read`, carrying an `act` claim for A
- [ ] Mint a new JWT inside A with the same claims and A's signing key

Forwarding the token sends an over-scoped credential to a service that should reject it (wrong audience). A service token plus a user-ID header lets A claim to be any user. Token exchange (RFC 8693) issues a narrower, correctly-audienced token that records both the user and the acting service. Services minting their own user tokens bypass the authorization server entirely.
</quiz>

<quiz>
With PostgreSQL row-level security, each request runs `SET app.tenant_id = '<tenant>'` on a pooled connection, outside any transaction. Occasionally a request sees another tenant's data. Why?

- [ ] RLS policies are cached per table, not per connection
- [x] A session-level `SET` persists on the pooled connection, so a later request that fails to set it inherits the previous tenant
- [ ] RLS only applies to `UPDATE` and `DELETE`, not `SELECT`
- [ ] The tenant ID must be an integer, not a UUID

Session-level settings live as long as the connection, and pools hand connections from one request to the next. Use transaction-local settings — `SET LOCAL` or `set_config('app.tenant_id', ..., true)` — inside the request's transaction so the value disappears at commit or rollback.
</quiz>

<quiz>
You enabled row-level security on `invoices`, but tests show the application still reads every tenant's rows. Which could explain it? Select ALL that apply:

- [x] The app connects as the table owner and `FORCE ROW LEVEL SECURITY` isn't set
- [x] The app's database role is a superuser or has `BYPASSRLS`
- [ ] The RLS policy uses `current_setting('app.tenant_id')`
- [x] No policy was created, or it uses an always-true expression

Table owners skip RLS unless it's forced, and superusers or `BYPASSRLS` roles skip it entirely — application roles should be neither. A missing or always-true policy also leaves the table open. Reading the tenant from `current_setting` is the intended pattern.
</quiz>

<quiz>
A nightly worker generates invoice exports from a queue. Since you enabled row-level security, the exports are either empty or contain every tenant's data. What's the root cause?

- [ ] Workers can't use row-level security
- [x] The job runs with no tenant context — the tenant must travel in the job payload and be set before any query
- [ ] Exports should run as a superuser to avoid RLS
- [ ] The queue is reordering jobs between tenants

Request-scoped tenant context disappears once work moves to a background job. Depending on how the worker connects, missing context either matches nothing or — with a privileged role — bypasses isolation. Carry the tenant in the job payload, set it at the start of the job's transaction, and apply the same rule to search indexes and other derived stores.
</quiz>

<quiz>
Support wants a "log in as user" feature to debug customer issues. Which controls should it have? Select ALL that apply:

- [x] A dedicated impersonation permission, separate from normal admin rights
- [x] A required reason, a short expiry, and visibility to the impersonated user
- [x] A token recording both identities (`sub` = user, `act` = agent) and a full audit trail
- [ ] Issue the agent the user's normal session so the behavior is identical

Impersonation is delegation and should look like it: a scoped, time-boxed token whose `act` claim names the agent, so every action is attributable. Handing over an ordinary user session erases the agent from the logs and grants full power with no trace.
</quiz>
