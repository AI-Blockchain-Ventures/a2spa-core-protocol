# A2SPA Core Protocol

**Status:** Independent Technical Working Draft  
**Version:** 0.9.1  
**Review Status:** External technical review requested

Agent-to-Secure Payload Authorization (A2SPA) defines an execution-time authorization model for determining whether a specific structured execution request may be admitted immediately before execution.

## Core Invariant

An Execution Request SHALL NOT be admitted to the Execution Environment unless the Execution Enforcement Point has received a successful Authorization Decision from the Authorization Verifier for the corresponding Signed Canonical Execution Request.

## Scope

A2SPA addresses deterministic, cryptographic validation of structured execution requests at the execution boundary.

It does not validate natural-language prompts, model alignment, semantic correctness, business correctness, legality, or the desirability of an upstream decision.

## Technical Review Requested

Feedback is welcome on:

- Security assumptions and threat-model gaps
- Canonicalization and signature interoperability
- Replay, nonce, TTL, and failure-state handling
- Deployment and bypass risks
- Conformance requirements
- Implementation concerns and edge cases
- Architectural flaws or falsification conditions

Please open a GitHub Issue with the relevant section number and severity: `blocker`, `high`, `medium`, or `low`.

## Status Clarification

This repository is an independent technical working draft. It is not an IETF working-group document, adopted standard, certification, or security audit.