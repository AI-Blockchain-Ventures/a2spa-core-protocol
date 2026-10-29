# A2SPA Core Protocol

**Status:** Independent Technical Working Draft

**Specification Version:** 0.9.3

**Implementation Snapshot:** A2SPA hosted service and helper packages `2.4.2` (protocol `v1.0`, HTTP API `/v1`)

**Review Status:** External technical review requested

Agent-to-Secure Payload Authorization (A2SPA) defines an execution-time authorization model for deciding whether a specific structured agent action should be accepted immediately before execution-time consequence.

## Current Specification

The current reviewable text is:

- [`spec/A2SPA_Core_Protocol_Specification_v0.9.3.md`](spec/A2SPA_Core_Protocol_Specification_v0.9.3.md)

The machine-readable API contract is published at <https://aimodularity.com/A2SPA/openapi.json>.

## Core Invariant

No signature, no execution.

An execution request MUST NOT be accepted as authorized unless the verifier has validated the exact signed payload, sender identity, target binding, timestamp, nonce, signature profile, policy constraints, and any required continuity evidence.

## Product Boundary

Agent Verify and A2SPA are separate but complementary controls.

- Agent Verify is pre-deployment analysis for finding agent and workflow execution vulnerabilities.
- A2SPA is execution-time authorization for stopping unauthorized runtime execution.
- This repository specifies the A2SPA protocol boundary. It does not specify Agent Verify scanning logic.

## Scope

A2SPA addresses deterministic, cryptographic validation of structured execution requests at the execution boundary.

It does not validate natural-language prompts, model alignment, semantic correctness, business correctness, legality, or the desirability of an upstream decision.

This repository describes the externally observable wire contract needed for interoperability review. It does not publish verifier source code, internal architecture, deployment design, extended-profile implementations, or internal validation evidence.

## Public Implementation Alignment

The public A2SPA helper packages are available as:

- Python: `pip install a2spa==2.4.2`
- Node.js: `npm install a2spa@2.4.2`

The public API is available at:

- Base: `https://api.aimodularity.com/A2SPA`
- Canonical routes: `/v1/*`
- Compatibility aliases: `/api/*` (same decisions and responses; new integrations should use `/v1`)

Releases `2.4.1` and `2.4.2` made no protocol, API, or signed-byte changes.

## Technical Review Requested

Feedback is welcome on:

- Security assumptions and threat-model gaps
- Signed-byte compatibility and canonicalization clarity
- Replay, nonce, operation-identity, timestamp, and failure-state handling
- Deployment and bypass risks
- Conformance requirements
- Implementation concerns and edge cases
- Architectural flaws or falsification conditions

Please open a GitHub Issue with the relevant section number and severity: `blocker`, `high`, `medium`, or `low`. Report suspected vulnerabilities privately as described in [`SECURITY.md`](SECURITY.md), not in public Issues.

## Status Clarification

This repository is an independent technical working draft. It is not an IETF working-group document, adopted standard, certification, or security audit. A2SPA `2.4.2` has completed internal engineering validation only; it has not been independently audited or certified.

## Rights

Copyright © 2026 AI Blockchain Ventures LLC. All rights reserved. See [`NOTICE.md`](NOTICE.md).
