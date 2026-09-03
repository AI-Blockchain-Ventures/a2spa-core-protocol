# A2SPA Core Protocol

**Status:** Independent Technical Working Draft

**Specification Version:** 0.9.2

**Implementation Snapshot:** public A2SPA app/API and helper packages `0.1.3`

**Review Status:** External technical review requested

Agent-to-Secure Payload Authorization (A2SPA) defines an execution-time authorization model for deciding whether a specific structured agent action should be accepted immediately before execution-time consequence.

## Current Specification

The current reviewable text is:

- [`spec/A2SPA_Core_Protocol_Specification_v0.9.2.md`](spec/A2SPA_Core_Protocol_Specification_v0.9.2.md)

The older `v0.9.1-clean` Word/PDF draft has been removed because it contained stale claims that do not match the current public implementation.

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

## Public Implementation Alignment

The public A2SPA helper packages are available as:

- Python: `pip install a2spa==0.1.3`
- Node.js: `npm install a2spa@0.1.3`

The public API is available at:

- Canonical base: `https://api.aimodularity.com/A2SPA`
- Bare API alias: `https://api.aimodularity.com/api/*`

## Technical Review Requested

Feedback is welcome on:

- Security assumptions and threat-model gaps
- Signed-byte compatibility and canonicalization clarity
- Replay, nonce, timestamp, and failure-state handling
- Deployment and bypass risks
- Conformance requirements
- Implementation concerns and edge cases
- Architectural flaws or falsification conditions

Please open a GitHub Issue with the relevant section number and severity: `blocker`, `high`, `medium`, or `low`.

## Status Clarification

This repository is an independent technical working draft. It is not an IETF working-group document, adopted standard, certification, or security audit.
