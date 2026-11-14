# Quiz: Basic Auth

Review first: [Basic Auth](../authentication/basic-auth.md)

<quiz>
You're building a webhook endpoint that receives callbacks from a payment provider. The provider only supports Basic Auth over HTTPS. Which additional control matters MOST?

- [ ] Base64-encode the expected credentials before comparing them
- [x] Enforce an allowlist of the provider's source IPs and rate-limit the endpoint
- [ ] Rotate the password every 24 hours via an automated job
- [ ] Switch the endpoint to digest authentication

Basic Auth is acceptable for server-to-server webhooks because TLS hides the credentials and the caller set is tiny. The real risk is an attacker *finding* the endpoint and brute-forcing it — IP allowlisting plus rate limiting shrinks that surface. Base64 adds no protection (it's encoding, not secrecy), aggressive rotation breaks the provider, and digest auth isn't supported.
</quiz>

<quiz>
During a review you find this comparison in an auth check: `if (request.password == stored)`. What's the concrete vulnerability?

- [x] Timing attack — the comparison exits early on the first mismatching byte, leaking password content through latency
- [ ] The password is stored in plaintext
- [ ] The comparison is vulnerable to SQL injection
- [ ] Nothing — string equality is safe for credentials

Language-level string equality short-circuits on the first differing character, so response time reveals how much of the guess was right. Use a constant-time compare (Python `secrets.compare_digest`, Java `MessageDigest.isEqual`). The snippet doesn't reveal the storage format, so "plaintext" isn't supported by the evidence.
</quiz>

<quiz>
An internal tool serves Basic Auth on both port 80 and 443. A teammate argues port 80 is fine "because the credentials are Base64-encoded anyway". What should you do?

- [ ] Nothing — Base64 makes the credentials unreadable to casual sniffing
- [ ] Double-encode the credentials with Base64 on port 80
- [x] Reject plain HTTP entirely, serve only over TLS, and enable HSTS
- [ ] Keep port 80 but require a stronger password policy

Base64 is a reversible encoding — anyone who captures the `Authorization` header decodes it instantly. Because Basic Auth sends the password on **every** request, plaintext transport is a guaranteed credential leak. TLS is mandatory; refuse port 80 rather than redirecting after the credentials have already been sent, and use HSTS so browsers never try HTTP. Password strength doesn't help once the password itself is on the wire.
</quiz>

<quiz>
Users of a Basic-Auth-protected admin page report that clicking "Log out" doesn't actually log them out. What's the underlying cause?

- [ ] The server forgot to delete the session record
- [x] Browsers cache Basic credentials and resend them automatically — there's no server-side session to end
- [ ] The `Authorization` header is stored in an HttpOnly cookie
- [ ] Basic Auth tokens have a fixed 24-hour expiry

Basic Auth is stateless: the browser remembers the username and password after the 401 challenge and attaches them to every request until the browser is closed or its cache is cleared. The server has nothing to revoke. That makes reliable logout one of the classic weaknesses of Basic Auth for human-facing UIs — use a session or token scheme when logout matters.
</quiz>

<quiz>
A Basic-Auth API slows down sharply under load. Profiling shows most CPU time goes to Argon2 password verification on every request. What's the right fix?

- [ ] Lower the Argon2 cost parameters to the minimum
- [ ] Cache the plaintext password in memory after the first successful check
- [ ] Switch password storage to fast SHA-256
- [x] Verify the password once, then issue a short-lived token or session for subsequent requests

Password hashes are *meant* to be slow so offline cracking is expensive — weakening them or caching plaintext trades a performance problem for a breach. The structural issue is that Basic Auth re-verifies the password on every call. Exchanging it once for a short-lived token or session keeps the expensive check to the login step.
</quiz>
