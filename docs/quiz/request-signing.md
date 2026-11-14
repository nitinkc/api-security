# Quiz: Request Signing & Webhook Verification

Review first: [Request Signing](../authentication/request-signing.md)

<quiz>
Your HMAC verifier canonicalizes `method + path + body_hash` and accepts a `t` timestamp outside the signature. An attacker captures one valid request. What can they do?

- [ ] Nothing — the signature still verifies
- [x] Replay it with a fresh `t` — the timestamp isn't signed, so they can update it indefinitely
- [ ] Only replay it for 5 minutes
- [ ] Forge new request bodies

Every security-relevant field must be *inside* the signature. An unsigned timestamp can be rewritten freely, so the freshness check is theater — the capture stays valid forever. Put `t` in the canonical string, and then a modified timestamp invalidates the signature.
</quiz>

<quiz>
Your HMAC check uses `if computed == provided: allow()`. What's the vulnerability class?

- [ ] None — the comparison is correct
- [ ] Hash collision — an attacker can find a matching signature
- [x] Timing oracle — early-exit string comparison leaks how many leading bytes are correct, letting an attacker rebuild the signature byte-by-byte
- [ ] The signature is exposed in memory during comparison

`==` returns false at the first mismatched byte — the response time encodes *where* you diverged. Repeat enough requests and the attacker recovers a valid signature for a chosen message. `hmac.compare_digest` / `crypto.timingSafeEqual` run in constant time, eliminating the oracle.
</quiz>

<quiz>
Stripe sends a webhook; your handler parses `payload.event.type` and processes the payment *before* verifying the signature. What's wrong?

- [ ] Nothing — parsing first is required to check the timestamp
- [x] An attacker can POST arbitrary event bodies and have them processed — signature verification must gate *any* business logic, using the raw bytes
- [ ] You should verify twice — once before, once after parsing
- [ ] It's fine — the signature only guards against replay, not forgery

The signature's job is to decide whether the bytes are trustworthy at all. Acting on unverified content means a forged `payment.succeeded` event credits a real order. Verify `t` and `v1` against the *raw* body first, reject mismatches, only then parse and dispatch. (And re-serialized JSON breaks the signature — hash the bytes as received.)
</quiz>

<quiz>
You have timestamp verification (±5 min) but no nonce store. What attack remains possible, and what's the fix?

- [ ] None — the timestamp window closes replay
- [x] Replay inside the window — a sniffed request can be resent for 5 minutes; add nonce/event-ID dedup so each signature is consumed once
- [ ] Signature stripping — remove the header entirely
- [ ] Algorithm confusion — switch to a weaker HMAC

The timestamp bounds *how long* a capture is replayable, not *whether* it can be replayed. Inside the window the request is still perfectly valid. A nonce or event-ID seen-set (Redis `SET NX EX`, keyed to the signature or `jti`) makes each signed request single-use.
</quiz>

<quiz>
When is HMAC request signing the *wrong* choice? Select ALL that apply:

- [x] A public mobile app — the shared secret would ship in the binary and be extractable
- [x] When a third party must independently verify who signed — HMAC proves "a key holder," and both sides hold the key; use asymmetric signatures for non-repudiation
- [ ] A Stripe-style webhook receiver
- [x] A simple internal call where a bearer token over TLS already suffices — canonicalization is added complexity for no gain

HMAC signing earns its complexity where bearer tokens are too weak (webhooks, integrity-critical server-to-server) — but it's a poor fit for distributed client binaries (secret extractability), for third-party-verifiable signatures (shared keys can't repudiate), and for calls that don't need per-request integrity. Adding it where TLS+bearer is enough just creates a canonicalization bug farm.
</quiz>
