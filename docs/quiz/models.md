# Quiz: Access-Control Models

Review first: [Access-Control Models](../authorization/models.md)

<quiz>
Your RBAC system now has roles like `support-agent-emea-readonly-tier2` and `support-agent-apac-readwrite-tier1` — over 300 of them. What's the right move?

- [ ] Keep adding roles; RBAC scales linearly
- [ ] Merge them all into one `support-agent` role with every permission
- [x] Keep a few job-function roles and move region, tier, and access mode into attribute conditions or relationships
- [ ] Switch to per-user ACLs on every resource

Encoding attributes into role names is *role explosion* — the signal you've outgrown pure RBAC. Keep roles for coarse job functions and express region, tier, and similar dimensions as ABAC conditions (or relationships). Merging everything into one role breaks least privilege, and per-resource ACLs make bulk changes even harder to administer.
</quiz>

<quiz>
You're building document sharing: users share documents and nested folders with people or groups, and the UI needs "who has access to this?" Which model fits best?

- [ ] ACL on each document
- [ ] RBAC with a role per folder
- [ ] ABAC with a policy per document
- [x] ReBAC (Zanzibar-style) with relationships between users, groups, folders, and documents

Sharing, inheritance through nested folders, and reverse "who can see this?" queries are exactly what relationship graphs model well — use a ReBAC engine like OpenFGA or SpiceDB rather than building your own. Per-document ACLs don't express inheritance, per-folder roles explode, and ABAC policies make "who can see X?" expensive to answer.
</quiz>

<quiz>
Handlers across your codebase contain checks like `if user.role == "admin" or user.role == "support-lead"`. Product wants a new `auditor` role. What's the better design?

- [x] Map roles to permissions in one place, and have handlers check a permission such as `tickets:refund`
- [ ] Add `or user.role == "auditor"` to each relevant handler
- [ ] Give auditors the `admin` role to avoid code changes
- [ ] Store the allowed role names in each endpoint's URL

Checking role names spreads policy across every handler, so each new role means editing — and missing — dozens of `if` statements. Checking *permissions* lets you define what `auditor` grants in a single mapping. Granting `admin` violates least privilege.
</quiz>

<quiz>
A health-data API must allow a read only if the caller's tenant matches the record's, and records classified `phi` additionally require the caller to be HIPAA-trained. Which model expresses this most naturally?

- [ ] ACL
- [ ] RBAC
- [x] ABAC
- [ ] ReBAC

The decision depends on attributes of the subject (tenant, training), resource (tenant, classification), and action — that's attribute-based access control. Keep the policy declarative and testable (e.g. in a policy engine) rather than hand-coded `if` statements, so it stays auditable.
</quiz>

<quiz>
An employee was removed from the `finance-admin` role in the IdP at 9:00. At 15:00 they still approve payments. Access tokens carry roles and live for 24 hours. Which changes fix this? Select ALL that apply:

- [x] Shorten access-token lifetimes so role changes take effect within a bounded time
- [x] Look up volatile or high-risk permissions server-side instead of trusting the token snapshot
- [ ] Put every fine-grained permission in the token so it's more precise
- [x] Revoke the user's sessions and refresh tokens when their roles change

A token is a frozen copy of permissions at issue time — a 24-hour lifetime means a 24-hour lag on demotions. Short TTLs, live checks for high-risk actions, and revoking refresh tokens on role change all bound that lag. Stuffing more permissions into the token makes it bigger, not fresher.
</quiz>
