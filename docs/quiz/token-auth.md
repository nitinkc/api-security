# Quiz: Token Auth & API Keys

Review first: [Token Auth](../authentication/token-auth.md)

<quiz>
Your mobile app ships with an embedded API key. A security review flags it as exposed. What's the correct fix?

- [ ] Obfuscate the key by splitting it across resources and code
- [x] Issue short-lived user-scoped tokens after login; treat any app-embedded secret as public
- [ ] Move the key into the app's native keystore so it can't be extracted
- [ ] Rotate the embedded key monthly via an app update

Anything shipped inside a client binary is extractable — obfuscation and keystore storage only raise the effort slightly, and rotation via app updates can't keep pace. The real fix is architectural: the app authenticates *users* and receives revocable, short-lived tokens instead of a static shared key.
</quiz>

<quiz>
A partner API accepts keys via `GET /data?apikey=...`. The provider asks how to reduce accidental key leakage. Select ALL that apply:

- [x] Move the key to an `Authorization` or `X-API-Key` request header
- [ ] Require the key to be sent in the request body instead
- [x] Strip the apikey query param in the gateway before it reaches origin logs
- [x] Ensure URLs are never written to access logs, browser history, or analytics

Query-string credentials land in logs, browser history, proxies, and referrer headers — headers avoid all of that. Stripping the param at the gateway keeps it out of origin logs. A request body doesn't work for GET and still isn't logged safely in most pipelines.
</quiz>

<quiz>
Which of these is an acceptable way to generate a new API key?

- [ ] `sha256(user_id + current_timestamp)`
- [ ] A UUIDv1 generated at signup
- [x] `secrets.token_urlsafe(32)` (Python) or 32 bytes from `SecureRandom` (Java)
- [ ] `Math.random().toString(36)` repeated four times

A key is only as strong as its unpredictability. It must come from a CSPRNG with at least 128 bits of entropy — 32 random bytes gives 256. Hashing a timestamp and user ID is guessable, UUIDv1 embeds the MAC address and time, and `Math.random` isn't cryptographically secure.
</quiz>

<quiz>
You're designing storage for long, high-entropy API keys that are checked on every request. How should they be stored?

- [ ] In plaintext, with the database encrypted at rest
- [x] As a SHA-256 hash, looked up by an index on the hash
- [ ] As a bcrypt hash, compared against every row on each request
- [ ] Encrypted with an AES key kept in the same database

Storing only `SHA-256(key)` means a database leak exposes hashes, not usable keys, and the hash can be indexed for a fast lookup. Slow, salted password hashes like bcrypt exist to protect *low-entropy* human passwords; a 256-bit random key can't be brute-forced, so bcrypt only adds latency and prevents indexed lookup. Disk encryption and co-located keys don't help once an attacker can query the database.
</quiz>

<quiz>
Why do providers prefix keys with strings like `sk_live_` or `ghp_`?

- [ ] The prefix adds entropy that makes keys harder to brute-force
- [ ] It lets the server skip the database lookup for invalid prefixes, which is the main security benefit
- [x] Leaked keys become recognizable — secret scanners and push protection can detect and revoke them, and humans can tell live from test keys
- [ ] Prefixes are required by RFC 6750

A fixed prefix adds no entropy, but it makes a key *identifiable*. GitHub push protection and other secret scanners pattern-match on known prefixes to catch keys committed to repos, pasted in tickets, or logged — and the `live` / `test` part helps humans avoid mixing environments. RFC 6750 defines bearer-token transport, not key formats.
</quiz>
