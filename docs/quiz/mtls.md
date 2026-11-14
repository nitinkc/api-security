# Quiz: Mutual TLS (mTLS)

Review first: [Mutual TLS](../authentication/mtls.md)

<quiz>
Your service mesh terminates mTLS at an ingress gateway, which forwards the caller's identity to backends via an `X-Client-DN` header. A pentester bypasses the gateway and hits a backend directly with a self-signed request plus a forged header — and gets in. What's the flaw?

- [ ] The backend should have checked the header format more carefully
- [x] The header is just data — the backend trusted identity that wasn't cryptographically established; mTLS must extend to the backend or the backend must be unreachable without it
- [ ] The pentester must have stolen a real certificate
- [ ] Headers shouldn't be used for identity at all

A verified cert produces the identity; a header merely *claims* it. If any path reaches the backend without the TLS-verified handshake, the trust boundary is imaginary. Either terminate mTLS at the backend (end-to-end) or guarantee — by network policy — that nothing reaches it except through the verified gateway.
</quiz>

<quiz>
A client certificate is stolen (key file + cert copied off a host). What's the effective containment strategy in a well-run mTLS system?

- [ ] Publish the serial to a CRL and wait for all servers to download it
- [x] Certs are short-lived (hours) with automated rotation, so the stolen cert dies of old age almost immediately
- [ ] Change the CA key so all certs are invalidated
- [ ] Nothing can be done — certificate theft is permanent until expiry

Short-lived workload certs are the modern answer — revocation infrastructure (CRL/OCSP) has always been laggy and unreliable, so SPIRE/mesh systems prefer lifetimes measured in minutes-to-hours where "rotation" is the revocation. Rotating the CA nukes every workload, not just the compromised one.
</quiz>

<quiz>
Your mTLS verifier checks that the client cert chains to the internal CA — and nothing else. A cert issued for the `dev` environment's monitoring agent is used to call the `payments` API and succeeds. What check was missing?

- [ ] The cert should have been pinned by fingerprint
- [x] Identity verification — chain validation proves a cert is *real*, not that *this* identity may call *this* service; check the SAN/SPIFFE ID against an allowlist
- [ ] The CA should use a longer key
- [ ] TLS 1.3 would have prevented this

Chaining to the CA answers "was this issued by us?" — every workload in the org passes it. The authorization-relevant question is "is this the caller we expected?" — which lives in the SAN/SPIFFE ID. It's the same class of bug as skipping `aud` validation on a JWT: authentication without identity scoping.
</quiz>

<quiz>
Which scenarios are genuinely good fits for mTLS? Select ALL that apply:

- [x] Service-to-service calls inside a Kubernetes cluster with a service mesh
- [x] A small set of partner systems calling your B2B API
- [ ] Consumer mobile app users logging in
- [x] Workload identity where certs are attested, short-lived, and auto-rotated (SPIFFE/SPIRE)

mTLS shines where callers are machines you can provision certs for — mesh-managed east-west traffic, managed partner integrations, attested workload identity. It's hostile to human callers (enrolling a personal cert into a phone browser is miserable UX) and doesn't scale to open third-party ecosystems, where OAuth client credentials is the practical answer.
</quiz>

<quiz>
"Internal traffic doesn't need TLS — it's inside our VPC." Which arguments actually rebut this? Select ALL that apply:

- [x] The network perimeter is not a trust boundary — compromised sidecars, flat networks, and lateral movement all intercept plaintext
- [x] Compliance regimes (PCI, HIPAA) increasingly mandate encryption in transit everywhere
- [x] Service identity becomes unforgeable with mTLS — a header-based identity scheme is spoofable
- [ ] TLS is computationally free and adds no operational cost

"Inside" is only trustworthy if you control everything in it — you don't. Defense-in-depth assumes breach; mutual TLS gives every hop authenticated encryption and workload identity. But it's not free: TLS does add latency, and cert lifecycle machinery is real operational work — which is exactly what service meshes exist to automate.
</quiz>
