# Quiz: Multi-Factor Authentication

Review first: [MFA](../authentication/mfa.md)

<quiz>
A user enters their password, then answers "What was your first pet's name?" Which statement is correct?

- [ ] This is valid two-factor authentication
- [x] This is not MFA — both are knowledge factors, and one phishing page captures both
- [ ] It counts as MFA only if the answer is hashed
- [ ] It counts as MFA for low-risk applications

Factors must come from *different* categories: something you know, have, or are. Password and security question are both knowledge — the same credential-phishing page harvests them in one shot, so the second "factor" adds friction but no independent defense.
</quiz>

<quiz>
Employees are receiving dozens of push-notification approvals at 3 AM and some tap "Approve" to stop the noise. Attackers already have their passwords. Which mitigations actually help? Select ALL that apply:

- [x] Switch the push prompt to number matching — the user must type a code shown on screen
- [x] Rate-limit push attempts and alert on repeated denials
- [ ] Increase password complexity requirements
- [x] Move high-privilege users to passkeys / FIDO2 keys

This is MFA fatigue (the Uber 2022 pattern). Number matching forces the attacker to also phish the on-screen code live; rate limits stop the spam; phishing-resistant factors make the stolen password nearly useless. A stronger password doesn't help — it's already compromised.
</quiz>

<quiz>
Your API must require a *fresh* second factor before allowing `POST /admin/roles`. The request arrives with a valid access token issued 2 hours ago after password login. What should the API check?

- [ ] The `exp` claim — reject tokens older than 1 hour
- [x] The `acr`/`amr` claims and `auth_time` — require the step-up level and a recent authentication
- [ ] The user's password again on every role assignment
- [ ] Nothing — a valid token is a valid token

The token *is* valid, but it doesn't prove a recent strong authentication. `amr` tells you *how* the user authenticated, `acr` the assurance level, and `auth_time` *when*. If they don't meet the step-up bar, return `403` with `insufficient_user_authentication` and let the client re-authenticate with `acr_values=step-up`.
</quiz>

<quiz>
An attacker phishes a user on a lookalike domain that proxies to your real site — relaying username, password, and the 6-digit TOTP code live. Which factor types would have broken this attack?

- [ ] A longer TOTP code (8 digits instead of 6)
- [x] WebAuthn / passkeys — the signed challenge is bound to the legitimate origin
- [ ] A second TOTP code from a different app
- [ ] Email OTP instead of TOTP

Phishable factors (TOTP, SMS, email codes) all fail against a real-time relay — the attacker just forwards them. WebAuthn signs a challenge that includes the *origin*; a credential produced on `evi1-site.com` fails verification for `your-app.com`, so the relay dies at the TLS layer.
</quiz>

<quiz>
A database leak exposes your TOTP secrets table. Why is this catastrophic, and what was the correct storage design?

- [ ] It's fine — TOTP codes change every 30 seconds
- [x] Anyone with the seed can generate the user's second factor forever — seeds should be encrypted at rest (KMS), like password keys
- [ ] It's only a problem if the attacker also has passwords — which they never will
- [ ] TOTP seeds can't be stored encrypted; the spec forbids it

A TOTP seed *is* the second factor. Whoever holds it mints valid codes indefinitely — MFA silently collapses to single-factor for every leaked user. Encrypt seeds at rest via KMS/HSM and treat them as secrets. Meanwhile, recovery codes (unrelated) should be stored *hashed* — the opposite treatment, same reason: the DB is not a safe place for usable credentials.
</quiz>
