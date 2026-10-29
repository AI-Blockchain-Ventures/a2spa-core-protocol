# Changelog

## v0.9.3 - 2.4.2-Aligned Review Draft - October 29, 2026

Aligns the protocol repository with the A2SPA hosted service and helper packages `2.4.2` (protocol `v1.0`, HTTP API `/v1`).

- Moves the implementation snapshot from helper packages `0.1.3` to `2.4.2`, and links the published OpenAPI 3.1 contract.
- Documents `/v1` as the canonical API routes, with `/api/*` retained as compatibility aliases.
- Documents `alert_threshold` as optional with a default of `10`, adds the optional `operation_id` and `delegation` payload fields, and states nonce and field constraints.
- Adds `operation_id` and `delegation` to the Signed Bytes V1 signable field set when present.
- Documents exact-retry results and fail-closed reuse of a nonce or operation identity with changed content.
- Adds `429` and `503` handling, error `reason_code`, and the top-level `decision` field.
- Documents the signed receipt envelope, the `operation_id` receipt field, and rejection of revoked or disabled receipt keys.
- States that `2.4.2` has completed internal engineering validation only, not independent audit or certification.
- Adds a scope-of-disclosure appendix and a rights notice. The specification covers the external wire contract only.

## v0.9.2 - Implementation-Aligned Review Draft - September 3, 2026

Aligns the protocol repository with the public A2SPA app/API and helper packages `0.1.3`.

- Adds a Markdown specification as the current reviewable text.
- Clarifies that this repository is independent and is not an IETF working-group document.
- Clarifies the product boundary between Agent Verify pre-deployment analysis and A2SPA execution-time authorization.
- Replaces unverified draft claims about Go/Rust reference implementations, Ed25519 baseline requirements, RFC 8785 baseline canonicalization, EEP-issued challenge nonces, and execution tokens with wording that matches the current public implementation.
- Documents the current public payload envelope, signed-byte compatibility profile, RSA SHA-256 signature profile, optional ML-DSA profile, nonce/timestamp replay controls, state-continuity checks, and signed execution receipts.
- Removes the `v0.9.1-clean` Word artifact to avoid publishing stale protocol text as current guidance.

## v0.9.1 — External Technical Review Draft — June 28, 2026

Initial public external technical-review publication of the A2SPA Core Protocol Specification.

This version is frozen for external review. Editorial corrections may be made before publication; substantive protocol changes will be evaluated for a subsequent draft.
