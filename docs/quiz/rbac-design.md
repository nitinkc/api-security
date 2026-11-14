# Quiz: Worked Example — RBAC API

Review first: [Worked Example: RBAC API](../authorization/rbac-design.md)

<quiz>
An expense API has three roles. `POST /expenses/{id}/approve` is guarded by `hasAuthority('expense:approve')`, granted to managers and admins. Which bugs remain? Select ALL that apply:

- [x] A manager can approve expenses from teams they don't manage
- [x] A manager or admin can approve their own expense
- [x] An already-approved or draft expense can be approved again
- [ ] A regular user can approve expenses

The function-level check correctly blocks users, but RBAC alone says nothing about *which* expense. The fix is to load the expense through the role's visibility filter (team or all), then enforce separation of duties (not your own) and the state rule (only `submitted`) as explicit checks.
</quiz>

<quiz>
The team is debating `GET /expenses` for everyone plus a separate `GET /admin/expenses` for admins. Why does the worked example prefer one endpoint?

- [ ] Admin routes can't be protected by function-level checks
- [x] One code path whose data scope and view depend on the caller is easier to secure and test; parallel admin routes duplicate logic and drift
- [ ] REST forbids role names in URLs
- [ ] Separate routes break browser caching

Separate admin routes double the attack surface and tend to diverge — a fix applied to one path gets missed on the other. A single endpoint that loads through `visible_to(user)` and renders a role-specific view keeps one place to get right.
</quiz>

<quiz>
Expense 123 belongs to Bob. Alice (another employee) calls `POST /expenses/123/approve` and gets **404**. Bob calls the same endpoint on his own expense and gets **403**. Why the difference?

- [ ] It's inconsistent — both should get 403
- [ ] Bob gets 403 because users can never call approve endpoints
- [x] Alice can't see the expense, so it's hidden (404); Bob can see it but the action is forbidden by separation of duties (403)
- [ ] Alice's token is invalid

Objects outside the caller's visibility return 404 so probing reveals nothing. When the caller can already see the object, hiding it gains nothing, and a clear 403 helps legitimate clients understand the action isn't allowed — here because nobody may approve their own expense.
</quiz>

<quiz>
`GET /expenses` and `GET /expenses/{id}` both go through `visible_to(user)`. The new `GET /expenses/export` endpoint checks `expense:export:team` and then runs `SELECT * FROM expenses`. What happens when a manager calls it?

- [ ] They receive only their team's expenses, because the permission says `team`
- [x] They receive every expense in the company — the permission check passed but no data scope was applied
- [ ] The request fails with 403
- [ ] They receive only their own expenses

A permission check answers "may you call this?", not "which rows?". Non-CRUD endpoints like exports, searches, and reports are where the visibility filter is most often forgotten. Every query — exports included — must start from the same `visible_to(user)` filter.
</quiz>

<quiz>
`PUT /users/{id}/roles` is admin-only. Which additional hardening does the worked example recommend? Select ALL that apply:

- [x] Step-up MFA and an audit log entry for each role assignment
- [x] Re-check the caller's admin role against the database rather than trusting a token snapshot
- [x] Block removing the last admin, and require two-person approval for admin grants
- [ ] Let users set their initial role in the signup request body

Role assignment is the most privileged action in the system: a mistake or a stolen admin session escalates everything. Harden it with step-up MFA, a live role check, an audit trail, and guards against lock-out and single-person admin grants. Roles must never come from the request body — that's mass assignment.
</quiz>
