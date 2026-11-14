# Quiz: Policy Engines & Enforcement

Review first: [Policy Engines & Enforcement](../authorization/policy-engines.md)

<quiz>
Your service calls an OPA sidecar for every authorization decision. During an incident the sidecar is unreachable. What should the service do?

- [ ] Allow the request so customers aren't affected by an internal outage
- [ ] Fall back to the decision cached for any user who made the same request recently
- [x] Deny the request (fail closed) and alert
- [ ] Skip authorization only for read-only (`GET`) requests

Authorization must fail closed: an availability incident is recoverable, a data breach is not. Caching decisions across users is itself a cross-user leak, and "reads are harmless" is exactly how BOLA data exposure happens. Keep the PDP close (sidecar or embedded) and highly available so failing closed is rare.
</quiz>

<quiz>
The API gateway validates tokens and enforces `scope=orders:read` on `GET /orders/*`. A team proposes removing the authorization code inside the orders service to reduce duplication. What's the problem?

- [ ] None — the gateway is the policy enforcement point
- [x] The gateway can't know who owns order 42, so object-level checks must still run in the service
- [ ] Gateways can't validate JWTs
- [ ] Scope checks must be done twice for redundancy

Gateways are good at authentication, audience and scope checks, and route-level rules — function-level access. They don't have the domain data to answer "is this *Alice's* order?", so removing the in-service check reopens BOLA. Object checks run where the object is loaded, even if the decision is delegated to a shared PDP.
</quiz>

<quiz>
For `DELETE /projects/7`, the policy engine needs to know who owns project 7. In the PEP/PDP architecture, which component supplies that?

- [ ] PEP — Policy Enforcement Point
- [ ] PAP — Policy Administration Point
- [x] PIP — Policy Information Point
- [ ] The client, in a request header

The PEP intercepts and enforces, the PDP evaluates the policy, the PAP is where policies are written and versioned — and the PIP supplies extra attributes such as resource ownership, roles, or relationships. Trusting the client to report ownership would let it claim anything.
</quiz>

<quiz>
`GET /projects` must return only the projects the caller may see, from a table of 2 million rows. What's the right approach with a policy engine?

- [ ] Load every row and call the PDP once per row
- [ ] Skip authorization for list endpoints, since each item is checked when opened
- [x] Use partial evaluation or a lookup API (e.g. OPA partial eval, OpenFGA `ListObjects`) to produce a query filter
- [ ] Cache the full project list per tenant and return it to everyone in that tenant

Per-row PDP calls don't scale, and unfiltered lists are a data leak in their own right. Partial evaluation turns the policy into a `WHERE` clause; ReBAC engines expose `ListObjects`/`LookupResources` for the same purpose. A per-tenant cache ignores per-user permissions inside the tenant.
</quiz>

<quiz>
Which practices belong in a production policy-engine rollout? Select ALL that apply:

- [x] Policies stored in git, reviewed, and covered by allow/deny unit tests in CI
- [x] Each decision logged with subject, action, resource, result, and policy version
- [x] Decision caches with short TTLs, never shared across subjects
- [ ] Detailed deny reasons returned to the client to ease debugging

Policy as code, decision logging, and careful caching are the operational backbone of a PDP. Detailed deny reasons belong in logs, not responses — they reveal how your policy works and what attributes to tamper with. Return a generic 403 or 404 to the client.
</quiz>
