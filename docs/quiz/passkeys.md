# Quiz: Passkeys & WebAuthn

Review first: [Passkeys & WebAuthn](../authentication/passkeys.md)

<quiz>
An attacker runs a perfect lookalike of your login page on `evi1-bank.com` and relays everything live. Why does a passkey defeat this while TOTP does not?

- [ ] Passkeys use longer secrets than TOTP codes
- [ ] The authenticator checks the site's TLS certificate itself
- [x] The signed challenge includes the relying-party ID/origin — a signature produced on the wrong origin fails verification on yours
- [ ] Passkeys require a biometric, and biometrics can't be relayed

WebAuthn binds the signature to the origin the browser actually visited. On the phishing domain the signature is produced for `evi1-bank.com`; your server expects `your-bank.com` and rejects it — the relay gains nothing. TOTP codes have no origin context, so a live relay just forwards the digits. (And biometrics never leave the device — they only unlock the local key.)
</quiz>

<quiz>
Your WebAuthn verification code accepts any `challenge` field the client sends in its assertion payload. What's the consequence?

- [ ] None — the signature still has to verify
- [x] Assertions become replayable/forgeable — the server must issue a random, one-time challenge and verify the client echoed *that* one
- [ ] Only performance degradation
- [ ] Users could pick weak challenges, weakening their own accounts only

The challenge is the freshness guarantee. If the client chooses it, an attacker captures one valid assertion and replays it forever (or pre-signs a chosen challenge). The server must generate ≥128-bit random challenges, bind them to the session, expire them fast, and verify the assertion echoes the issued value.
</quiz>

<quiz>
During verification, an assertion arrives with a `signCount` *lower* than the value stored for this credential. What does this suggest, and what should you do?

- [ ] Clock skew — retry the request
- [ ] Normal behavior — authenticators reset counters
- [x] Possible cloned authenticator — the private key may exist in two places; reject and investigate/revoke the credential
- [ ] Nothing — `signCount` is advisory only

`signCount` is the authenticator's monotonic clone-detection counter. A regression means two devices believe they're the same credential — i.e., someone copied the private key (possible with some software authenticators and stolen devices). Treat it as a compromise signal: reject, alert, and re-register.
</quiz>

<quiz>
Your app supports passkeys but keeps "password + SMS code" as a fallback for all users. A pentest shows attackers still phish accounts successfully. Why?

- [ ] The passkey implementation must be buggy
- [ ] SMS codes are too short
- [x] Attackers choose the weakest accepted path — the fallback is the effective security level, not the passkey
- [ ] Passkeys only work on managed devices

Authentication security is set by the weakest accepted path, not the strongest offered. As long as password+SMS works, that's what gets phished. The fix is a policy decision: passwordless-only mode (or hardware-only) for high-value accounts, with a recovery flow that isn't weaker than what it replaces.
</quiz>

<quiz>
Which claims about passkeys vs. other schemes are true? Select ALL that apply:

- [x] The server stores only public keys — a server database breach leaks nothing phishable
- [x] Passkeys authenticate *users*; APIs downstream still consume session cookies or OAuth tokens
- [ ] Passkeys are suitable for service-to-service authentication
- [x] `uv` (user verification) vs `up` (user presence) is the difference between a biometric/PIN check and a mere touch

There's no shared secret to steal — the DB holds public keys, so a breach doesn't yield usable credentials. Passkeys replace the human login step, not the token layer, and there's no human to "verify presence" in a service call — that's mTLS/workload-identity territory. `up` just means someone touched the authenticator; `uv` means the device actually verified *who* touched it.
</quiz>
