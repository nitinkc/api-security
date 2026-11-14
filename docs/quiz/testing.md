# Quiz: Testing & Pitfalls

Review first: [Testing & Pitfalls](../authorization/testing.md)

<quiz>
You have one afternoon to look for BOLA in an API you're authorized to test. Which technique gives the most coverage?

- [ ] Run a generic vulnerability scanner without credentials
- [x] Create data as User A, then replay every captured request with User B's token and flag any 2xx
- [ ] Fuzz every parameter with SQL injection payloads
- [ ] Test every endpoint with an admin token

The two-user test targets BOLA directly: the only difference between the requests is *who* sends them, so any success for User B is a finding. Repeat with a user from another tenant, a lower role, and no token at all. Unauthenticated scanners can't see authorization flaws, and admin tokens are allowed almost everything, so nothing fails.
</quiz>

<quiz>
A developer adds `GET /reports/revenue` and forgets the authorization decorator. Which automated control catches this before it ships?

- [ ] A linter that enforces consistent route naming
- [ ] Code coverage thresholds
- [x] A CI check that diffs the router's (or OpenAPI's) routes against routes with a declared authorization rule and fails on any gap
- [ ] A load test of all endpoints

A route inventory diff turns "every endpoint has a rule" into a test that fails the build. Pair it with a deny-by-default router so a missed rule fails closed even in production. Coverage and load tests exercise code; they don't check whether it's protected.
</quiz>

<quiz>
`POST /documents/{id}/transfer` checks that the caller owns the document, then — in a separate query a few lines later — updates its owner. Under concurrent requests, two different users each end up "owning" the same document. What's the bug class and fix?

- [ ] BFLA — add a role check
- [x] Check-then-act race (TOCTOU) — perform the check and write in one transaction or as a conditional update (`UPDATE ... WHERE id = :id AND owner_id = :me`)
- [ ] Mass assignment — use an input DTO
- [ ] CSRF — add a token

Between the check and the write, the state the check relied on can change. Making the ownership condition part of the write itself — or holding a row lock inside one transaction — closes the window. Check how many rows the conditional update affected; zero means deny.
</quiz>

<quiz>
REST endpoints enforce ownership correctly, but a pentester reads other users' orders through the GraphQL API, and also through the CSV export. What's the systemic fix?

- [ ] Disable GraphQL
- [ ] Add the same `if` statement to each GraphQL resolver and export job
- [x] Move authorization into a shared layer (repository, loader, or PDP) that every entry point — REST, GraphQL, gRPC, websockets, exports — must go through
- [ ] Rate-limit GraphQL more aggressively

Inconsistent checks across channels happen when authorization lives in controllers. Copying `if` statements just recreates the drift. A shared authorization layer where data is loaded makes every channel inherit the same rules, and the test matrix should cover every channel too.
</quiz>

<quiz>
`GET /invoices/{id}` returns 403 when the invoice exists but belongs to someone else, and 404 when it doesn't exist. Why is this a problem for sensitive resources?

- [ ] 403 is the wrong status code for authorization failures
- [x] The difference lets an attacker enumerate which IDs exist — return 404 for both
- [ ] 404 responses are cached by browsers
- [ ] It isn't a problem; this is the recommended pattern

Different responses for "exists but not yours" and "doesn't exist" leak existence — useful for enumeration and sometimes sensitive in itself (does this patient, account, or case exist?). Return 404 in both cases when existence is sensitive; a 403 is fine once the caller can already see the object.
</quiz>

<quiz>
All authorization tests pass, yet a pentest finds BFLA on a dozen endpoints. The fixtures show every test authenticates as `admin@example.com`. Which changes address this? Select ALL that apply:

- [x] Make the least-privileged role the default test fixture
- [x] Generate tests from an actor × action × object matrix, including expected 401/403/404 cells
- [x] Fuzz write payloads with privileged fields such as `role`, `is_admin`, and `owner_id`
- [ ] Add more admin-level happy-path tests

Admins are allowed almost everything, so admin-only tests can't fail on authorization. Testing deliberately as the wrong user, tenant, and role — driven by a matrix that *is* the specification — is what exposes missing checks. Property fuzzing catches the mass-assignment variants.
</quiz>
