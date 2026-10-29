# A2SPA Core Protocol Specification

**Document Version:** 0.9.3

**Status:** Independent Technical Working Draft

**Implementation Snapshot:** A2SPA hosted service and helper packages `2.4.2` (protocol `v1.0`, HTTP API `/v1`)

**Date:** October 29, 2026

**Review Status:** External technical review requested

## Abstract

Agent-to-Secure Payload Authorization (A2SPA) is an execution-time authorization protocol for structured autonomous-agent actions. A2SPA is designed to determine whether a specific signed payload should be accepted immediately before execution-time consequence.

A2SPA complements identity systems, policy systems, transport security, monitoring, and pre-deployment agent analysis. It does not attempt to validate natural-language prompts, model alignment, semantic correctness, ethics, legality, or business desirability. Its narrow purpose is deterministic validation of structured execution intent at the runtime boundary.

This draft is aligned with the externally observable behavior of the A2SPA hosted service and public helper packages `2.4.2`. It is not an IETF document, not an adopted standard, not a certification program, and not a security audit.

## 1. Status and Scope

This repository is an independent technical working draft for external review.

This draft specifies the externally observable wire contract only: the request envelope, signed bytes, signature profiles, freshness and replay semantics, decisions, and receipts that an integrator needs to interoperate. It does not describe verifier internals, storage design, deployment architecture, or the implementation of extended profiles. Those are proprietary and are outside the scope of this document (see Appendix C).

Earlier draft material described an IETF working-group status, a Go/Rust reference implementation, an Ed25519 mandatory baseline, an RFC 8785 baseline signed-byte profile, an EEP-generated challenge nonce handshake, and an execution-token state model. None of these describe current A2SPA behavior.

A2SPA addresses:

- Signed structured payloads
- Deterministic signed-byte construction
- Sender and target binding
- Timestamp, nonce, and operation-identity replay controls
- Policy evaluation
- Optional continuity evidence
- Signed execution receipts
- Fail-closed caller and enforcement behavior

A2SPA does not address:

- Prompt quality
- Model alignment
- Legal or business correctness
- Natural-language intent interpretation
- Full target-side exactly-once commit semantics
- Pre-deployment vulnerability discovery logic

## 2. Product Boundary

Agent Verify and A2SPA are complementary controls in the agent-security lifecycle.

- Agent Verify is a pre-deployment analysis product that finds execution vulnerabilities in agents and workflows before launch.
- A2SPA is an execution-time authorization protocol that blocks or records unauthorized runtime execution attempts.
- This specification covers A2SPA only.

The intended lifecycle is simple: find execution vulnerabilities before deployment, then stop unauthorized execution at runtime.

## 3. Terminology

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL are to be interpreted as described in BCP 14, RFC 2119, and RFC 8174 when they appear in all capitals.

- Subject Agent: The agent, workflow, service, or automation process that produces a structured execution payload.
- Authorization Verifier: The verifier that checks the signed payload, identity binding, timestamp, nonce, policy, and continuity evidence.
- Execution Enforcement Point: The caller-side, gateway, sidecar, runtime, or target-side component that refuses to continue unless A2SPA authorization succeeds.
- Target System: The downstream system that would receive the consequence of the action.
- Signed Payload: The structured payload plus signature submitted to the verifier.
- Operation Identity: The `operation_id` if supplied, otherwise the `nonce`, that names one authorization attempt.
- Execution Receipt: A signed verifier artifact that records the authorization decision and key payload bindings.

## 4. Core Invariant

No signature, no execution.

An execution request MUST NOT be accepted as authorized unless the verifier validates the exact signed payload, sender identity, target binding, timestamp, nonce, signature profile, policy constraints, and any required continuity evidence.

If an integration uses A2SPA as a caller-side verification gate, the caller MUST fail closed unless the returned decision is explicitly authorized. Helper functions such as `requireAuthorizedResult` (Node.js) and `require_authorized_result` (Python) exist to prevent integrations from treating non-authorized `200` responses as authorized execution.

## 5. Public API Snapshot

The public API base is:

```text
https://api.aimodularity.com/A2SPA
```

The machine-readable OpenAPI 3.1 contract is published at:

```text
https://aimodularity.com/A2SPA/openapi.json
```

Requests authenticate with an account API key in the `x-api-key` header.

Canonical runtime endpoints are:

- `POST /v1/verify_payload`
- `GET /v1/receipts/{receipt_id}`
- `POST /v1/continuity_state`

The earlier `/api/*` paths, including `POST /api/verify_payload`, `POST /api/continuity_state`, `POST /api/state_continuity`, and `GET /api/receipts/{receipt_id}`, remain compatibility aliases. They return the same decisions and response envelopes as their `/v1` equivalents. New integrations SHOULD use `/v1`. Alias removal requires a published migration path and advance notice.

The public helper packages are:

- Python: `a2spa==2.4.2` (Python 3.11 and 3.12)
- Node.js: `a2spa@2.4.2` (Node.js 22 or newer)

Helper packages follow semantic versioning from `2.4.0`. Patch releases do not change the protocol, the API contract, or the signed bytes.

## 6. Request Envelope

The `POST /v1/verify_payload` request body has this shape:

```json
{
  "payload": {
    "agent_id": "user_sender",
    "target_agent_id": "user_receiver",
    "timestamp": "2026-10-29T12:00:00Z",
    "nonce": "unique-nonce-0001",
    "hash": "64 lowercase hex sha256 digest",
    "input": {"action": "deliver", "workflow_scope": "messages:deliver"},
    "output": {"status": "ready"},
    "alert_threshold": 10,
    "crypto_profile": "a2spa.classical.rsa_pkcs1v15_sha256.v1"
  },
  "signature": "hex encoded signature"
}
```

Required payload fields are:

- `agent_id`
- `target_agent_id`
- `timestamp`
- `nonce`
- `hash`
- `input`
- `output`

Optional payload fields are:

- `alert_threshold` (integer, default `10`)
- `crypto_profile`
- `state_continuity`
- `operation_id`
- `delegation`

Field constraints:

- `input` and `output` MUST be JSON strings or JSON objects.
- `nonce` MUST be 8 to 512 characters, MUST start with a letter or digit, and MUST contain only letters, digits, `_`, `.`, `:`, and `-`.
- `operation_id`, if present, MUST be 1 to 160 characters with the same character rules as `nonce`.
- `crypto_profile`, if present, MUST name a profile of the form `a2spa.<name>.v<number>`.
- `hash` MUST be the lowercase SHA-256 hex digest of the Signed Bytes V1 payload (Section 7).
- `signature` MUST be a hex string.
- Unknown payload fields MUST be rejected.

The `delegation` field carries a delegated-authority claim. Its structure appears in the published OpenAPI contract; its validation profile is outside the scope of this draft.

## 7. Signed Bytes V1 Compatibility Profile

Current A2SPA public helper packages use the Signed Bytes V1 Compatibility Profile.

The signed bytes are the UTF-8 bytes of a Python-compatible JSON string produced from the normalized signable fields using sorted keys. In the Python helper this is equivalent to:

```python
json.dumps(normalize_signable_payload(payload), sort_keys=True).encode("utf-8")
```

The normalized signable field set is:

```text
agent_id
target_agent_id
timestamp
nonce
input
output
alert_threshold       (10 if not supplied)
crypto_profile        if present and not null
state_continuity      if present and not null
operation_id          if present and not null
delegation            if present and not null
```

The `hash` and `signature` fields are never part of the signed-byte input. Because `alert_threshold` is normalized to its default before signing, omitting it and sending `10` produce the same signed bytes.

Implementers SHOULD use the public helper packages, or compare their output against the helpers' `canonicalize_payload` result, before claiming compatibility. This profile is not RFC 8785 JCS. Future canonicalization profiles may be defined, but unsupported or mismatched profiles MUST fail explicitly and MUST NOT silently downgrade.

## 8. Signature Profiles

The active default signature profile is:

```text
a2spa.classical.rsa_pkcs1v15_sha256.v1
```

This profile signs the Signed Bytes V1 payload using RSA PKCS#1 v1.5 with SHA-256.

The compatibility profile for older payloads is:

```text
a2spa.legacy.rsa_pkcs1v15_sha256.v1
```

The optional post-quantum profile is:

```text
a2spa.pq.mldsa.v1
```

The ML-DSA profile is available only where the verifier deployment and key record support it. If a payload declares a profile that the verifier or sender key does not support, verification MUST fail closed.

## 9. Freshness, Replay, and Retry Controls

A2SPA uses payload-bound freshness controls:

- `timestamp` provides a bounded wall-clock validity window.
- `nonce` provides one-use replay resistance for the sender payload stream.
- `operation_id`, if supplied, names the authorization attempt for retry purposes.
- `hash` binds the received payload to the signed bytes.
- `signature` binds the sender key to the exact signable payload.

The hosted verifier rejects payload timestamps outside the configured freshness window. The current default window is 120 seconds.

Nonce consumption is verifier-side and atomic for a given `agent_id` plus `nonce` tuple. A nonce accepted once MUST NOT be accepted again within the nonce retention window.

An exact retry of a request whose operation has already reached a terminal decision returns the stored terminal result rather than a new decision. Reusing the same nonce or operation identity with different protected content MUST fail closed.

A2SPA replay protection prevents reused authorization payloads. It does not by itself guarantee target-side exactly-once business commit. Integrations that perform real-world side effects SHOULD bind a stable operation id or idempotency key to the target action and recover unknown commit status without duplicating the side effect.

## 10. Policy and Continuity Evidence

The verifier evaluates sender key status, sender enabled state, receiver availability, policy constraints, and optional state-continuity evidence. A sender that is disabled, expired, revoked, or has exhausted a usage limit is not authorized.

Common policy inputs include:

- `action`
- `workflow_scope`
- `workflow_scopes`
- `amount_usd`
- `currency`
- `on_behalf_of`

State-continuity evidence can bind a payload to an authoritative state hash and sequence. The public API supports continuity registration through `POST /v1/continuity_state` and its `/api` aliases.

Extended policy, delegation, and state-integrity profiles exist in the implementation. They are not specified in this draft.

## 11. Decisions and HTTP Handling

A successful authorization returns:

```json
{
  "success": true,
  "decision": "authorized",
  "receipt": {"receipt": {"decision": "authorized"}}
}
```

Decisions are `authorized`, `refer_hold`, and `denied`. Non-authorized receipt-bearing outcomes may return HTTP `200` with `success: false`, a decision of `refer_hold` or `denied`, and a signed receipt. Integrations MUST check the authorization decision and MUST NOT treat every HTTP `200` as authorization.

Recommended caller logic is:

- Accept only `success: true` plus decision `authorized`.
- Treat `refer_hold` as not authorized for execution.
- Treat `denied` as not authorized for execution.
- Treat malformed responses, timeouts, missing receipts, rate limiting, service unavailability, and verifier errors as fail-closed.

Common HTTP status behavior includes:

- `200` for authorized decisions and receipt-bearing non-authorized decisions.
- `400` for malformed payloads, hash mismatch, invalid timestamp, unsupported profile, or signature failure.
- `401` for missing or invalid API keys.
- `403` for permission, policy, target availability, or insufficient credits.
- `409` for replay, operation-identity, and continuity-state conflicts.
- `429` for rate limiting. The response MAY include `retry_after` in seconds.
- `500` for verifier-side processing or receipt-signing failures.
- `503` for temporary service unavailability.

Error responses carry an `error` string and MAY carry a `reason_code`.

## 12. Execution Receipts

The hosted service emits signed execution receipts for authorized and receipt-bearing non-authorized outcomes. Receipts can be retrieved by their owner through `GET /v1/receipts/{receipt_id}`.

The signed receipt envelope contains:

- `receipt`, the receipt body
- `canonical_hash`
- `signature`
- `signature_algorithm`
- `key_id`

Receipt body fields include:

- `schema_version` (`a2spa.execution_receipt.v1`)
- `receipt_id`
- `issued_at`
- `verifier_id`
- `decision`
- `agent_id`
- `target_agent_id`
- `payload_hash`
- `payload_nonce`
- `payload_timestamp`
- `operation_id`
- `metadata`

Receipt signatures use RSA-PSS-SHA256. Receipt verification is separate from payload signature verification. A receipt verifier that uses a trusted key registry MUST reject receipts signed by a key the registry marks as revoked or disabled.

A valid receipt proves the verifier's signed decision and the fields bound into it. It does not by itself prove that a caller or downstream target executed or committed the action.

## 13. Deployment Models

A2SPA defines logical roles and verifier behavior, not a mandatory topology. Valid deployment patterns may include caller-side verifier gates, API gateways, sidecars, service mesh filters, workflow-engine plugins, MCP-style tool adapters, A2A-style agent adapters, and target-native enforcement hooks.

Any deployment that permits consequential execution without checking an A2SPA authorized decision is outside the intended security boundary.

## 14. Threat Model

A2SPA is designed to reduce runtime blast-radius risk from:

- Payload tampering
- Sender spoofing
- Replay attempts
- Stale timestamp reuse
- Unauthorized action or scope expansion
- Receiver or target mismatch
- Missing continuity evidence where continuity is required
- Use of a sender key or agent after it is disabled, revoked, expired, or exhausted
- Caller integrations that confuse identity authentication with execution authorization

Important residual risks remain:

- A compromised private key can sign bad payloads until the key is revoked.
- A bad policy can authorize bad behavior.
- A caller that ignores the decision can still execute unsafely.
- A target system can duplicate side effects unless it uses its own idempotency or commit recovery.
- A semantically wrong but signed and policy-compliant request can still be authorized.
- A receipt shows a decision, not downstream execution.

## 15. Security Considerations

Private keys MUST be kept out of source control, logs, browser code, chat transcripts, and client-side bundles. Store private keys in server-side secrets, key-management infrastructure, or equivalent protected runtime storage.

Verifier availability MUST NOT weaken fail-closed behavior. If the verifier cannot answer, or answers with a rate-limit or unavailability status, the caller or enforcement point must not proceed as if the action is authorized.

Profile negotiation MUST be explicit. Unsupported signature or serialization profiles must fail rather than downgrade silently.

Receipts are audit evidence, not a substitute for checking the live authorization decision before execution.

## 16. Conformance Levels

This draft defines two practical conformance levels for review.

### 16.1 Public Compatibility Conformance

An implementation is public-compatible if it:

- Produces the Signed Bytes V1 Compatibility Profile exactly, including the `alert_threshold` default and the optional signable fields.
- Supports `a2spa.classical.rsa_pkcs1v15_sha256.v1` payload signing or verification.
- Computes payload hashes over the same normalized signable field set.
- Enforces timestamp freshness and one-use nonce semantics.
- Rejects malformed, unsigned, replayed, mismatched, or unauthorized payloads.
- Treats only `authorized` decisions as execution authorization.

### 16.2 Enforcement Conformance

An integration is enforcement-conformant if it:

- Places A2SPA before the consequential target action.
- Fails closed on missing, malformed, timeout, rate-limited, unavailable, denied, or refer-hold outcomes.
- Does not bypass the verifier path for equivalent target actions.
- Preserves enough receipt and decision evidence for audit.
- Uses target-side idempotency where side effects can be retried or duplicated.

## 17. Interoperability Guidance

Interoperability depends on matching signed bytes exactly. The safest approach is to use the official helper package for the runtime language.

Independent implementations should verify:

- The normalized signable field set, including defaults and optional fields.
- Sorted JSON object keys.
- Python-compatible JSON formatting for the Signed Bytes V1 profile.
- UTF-8 byte encoding.
- SHA-256 payload hash calculation.
- RSA PKCS#1 v1.5 SHA-256 signature verification for the classical profile.
- Exact decision handling for `authorized`, `refer_hold`, and `denied`.
- Request and response shapes against the published OpenAPI contract.

## 18. Open Review Questions

The following topics are open for external technical review:

- Whether a future RFC 8785 or deterministic CBOR profile should become a new versioned profile.
- How best to publish cross-language test vectors for the current Signed Bytes V1 profile.
- Which deployment profiles should be treated as enforcement-conformant versus verifier-only integrations.
- How to specify target-side operation ids without overclaiming exactly-once side-effect guarantees.
- How to represent human review without allowing natural-language summaries to replace exact payload binding.
- Whether operation identity and exact-retry semantics should be normative for all verifiers or only for managed deployments.

## 19. References

- RFC 2119: Key words for use in RFCs to Indicate Requirement Levels.
- RFC 8174: Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words.
- RFC 8017: PKCS #1 RSA Cryptography Specifications Version 2.2.
- RFC 6234: US Secure Hash Algorithms.
- FIPS 204: Module-Lattice-Based Digital Signature Standard.
- OpenAPI contract: `https://aimodularity.com/A2SPA/openapi.json`.
- Python helper package: `a2spa==2.4.2`.
- Node.js helper package: `a2spa@2.4.2`.

## Appendix A. Earlier Draft Disposition

The earlier `v0.9.1-clean` draft contains useful review structure, including role separation, invariants, threat modeling, conformance framing, security considerations, and falsification criteria.

It should not be copied verbatim into current public documentation because it includes claims that do not match A2SPA behavior. Specifically, it uses IETF-style status language, names a Go/Rust reference implementation, selects Ed25519 as mandatory baseline, treats RFC 8785 JCS as the baseline signed-byte profile, and describes EEP-issued challenge nonces and execution tokens. None of these are A2SPA `2.4.2` behavior.

## Appendix B. Implementation Status

- The `2.4.2` hosted service, Python package, and npm package share one version and implement protocol `v1.0`.
- Releases `2.4.1` and `2.4.2` made no protocol, API, or signed-byte changes. Integrations built against `2.4.0` remain compatible.
- `2.4.2` has completed internal engineering validation. It has not been independently audited, certified, or validated by a third party, and nothing in this repository should be read as such a claim.

## Appendix C. Scope of Disclosure

This document is published to support interoperability review of the A2SPA wire contract. It does not disclose, and reviewers should not expect it to disclose:

- Verifier source code, internal architecture, storage, or deployment design.
- How the verifier implements atomicity, retry ownership, credit accounting, or recovery.
- The implementation of policy, delegation, state-integrity, ephemeral-agent, or target-enforcement profiles.
- Internal test suites, validation evidence, or security findings.

Publication of this draft does not grant any license to A2SPA software, implementations, or any patent rights. See `NOTICE.md`.
