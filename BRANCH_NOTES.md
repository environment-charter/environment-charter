# Branch prep: feat/chain-state-v0.1-draft

This document is the human-facing scaffolding for the `feat/chain-state-v0.1-draft` branch in the environment-charter repository.

## Repo context

environment-charter is a fresh repository (initial commit only at 2026-05-01) standing up as the neutral home for the environment.* primitive family. Family-wide sections established between PRs #9 and #22 of `agent-intent/verifiable-intent` (Mastercard's repo) live upstream in those PR diffs and will be lifted into environment-charter alongside chain_state, market_state, and wallet_state in a coordinated v0.2 round.

This branch is the first primitive committed to environment-charter. No prior shared commits, no prior schemas, no prior branch-notes pattern.

## Branch scope

Single primitive added: **environment.chain_state**, drafted at v0.1.

Two files added:

- `schema/environment.chain-state.v0.1.json` — JSON Schema (Draft 2020-12) for the constraint envelope
- `spec/environment.chain-state.md` — Specification prose, type-local content only

No reference implementation. Schema and type-local spec only.

## Family integration posture

This draft does not duplicate family-wide prose. Sections §4.6, §4.7, §4.8, §5.5, §6.8, and Appendix C are referenced by section number rather than carrying byte-identical text from PRs #9 and #22.

The reasoning: family-wide convergence on those sections was achieved through coordinated commits where prose ports verbatim across both upstream PRs in lockstep — see for example PR #22 commit 7848bc2 (Gap 1 verbatim from PR #9 commit 7a8987c), PR #22 commit feb3292's §4.8 preamble byte-identical to PR #9's, Appendix C C.2-C.6 byte-identical between PR #9 commit 01490df and PR #22 commit 4f87906. Independently re-drafting that prose in chain_state v0.1 would create three-way divergence (chain_state v0.1 vs upstream market_state vs upstream wallet_state) that the next coordination round would have to reconcile.

The v0.1 posture: chain_state declares its type-local content (status enum, three freshness axes' issuer-side semantics, §4.8 table for chain_state's own fields, §5.5 distinguishing-field row for `chain_id`, §6.9 cross-chain temporal mechanics), references family-wide sections by number, and waits for a coordinated v0.2 round in this repository where family-wide prose is lifted in verbatim alongside market_state and wallet_state.

## Why now

Visa's Apr 29, 2026 nine-chain settlement expansion (Arc, Base, Canton, Polygon, Tempo added; nine chains total; Visa now validator on Tempo and Canton, design partner on Arc) makes a primitive that attests to chain-level liveness operationally necessary for any agent acting across this surface. The gap analysis (Apr 30) confirmed that no published spec in the agentic-payment standards space — Mastercard VI, Google AP2, TAP, x402, ERC-8004, Acharya, TessPay, Chainlink CCIP — claims this primitive. Moving the schema into the charter repository in draft form establishes timestamp leverage without committing to production receipts before counterparty interest materializes.

This branch is a schema flag-plant. No publication, no announcement, no production endpoint.

## Design choices for Douglas's review

Five choices made in v0.1 worth a peer-author look:

1. **Status enum: LIVE / DEGRADED / STALLED / UNKNOWN.** Parallel to environment.market_state's OPEN/CLOSED/HALTED/UNKNOWN in cardinality and fail-closed posture.

2. **`max_attestation_age` at constraint level (family-wide), per-axis `max_age_seconds` inside the attestation.** Two layers, two principals: constraint-level `max_attestation_age` is the consumer-policy TOCTOU bound (REQUIRED, no default, malformed if missing — strict pattern from PR #22 v0.4); per-axis `max_age_seconds` inside `freshness.{chain_liveness,finality_lag,rpc_consensus}` is the issuer's internal telemetry obligation. Same structural separation §4.6 establishes upstream.

3. **`chain_id` REQUIRED, no fallback, as the §5.5 distinguishing field.** Mirrors environment.wallet_state's `subject_wallet` (REQUIRED, no fallback) rather than environment.market_state's MIC-with-attestation-URL-fallback. CAIP-2 is the established cross-chain identifier; no fallback needed.

4. **`block_timestamp` OPTIONAL, finality measurement on verifier side.** Listed in §8 as open question. OPTIONAL preserves the §3 abstract-interface contract for non-block-time chains; verifier-side finality measurement prevents the issuer from becoming an attacker target for finality manipulation. v0.1 leaves OPTIONAL.

5. **§6.9 Cross-Chain Temporal Consistency.** Adopted from PR #22 v0.6 (commit feb3292) — orthogonal-axes framing, reorg-remediation as advisory SHOULD scoped to VI verifier's application-layer logic, measurement basis for `max_cross_chain_skew_seconds` via `block_timestamp` falling back to `iat`. Localized for multi-chain-state-attestations case.

## What is NOT in v0.1, deliberately

- Verbatim family-wide prose. Deferred to v0.2.
- Reference implementation. No `/v5/chain` endpoint, no canonical payload spec, no live test vectors.
- Mastercard VI registration. Not announced, not pursued.
- Production receipts. No HO commitment to ship until counterparty interest materializes.
- Public announcement. No essay, no Discord, no X. Branch flag-plant only.

## Commit plan

1. `chore(chain-state): add schema/environment.chain-state.v0.1.json` — schema only
2. `chore(chain-state): add spec/environment.chain-state.md` — spec prose
3. `docs(chain-state): add branch notes and design rationale` — this file

Each commit SSH-signed with `C:\Users\User\.ssh\id_ed25519_signing` (configured globally).

Co-Authored-By trailer for Douglas on every commit:

```
Co-Authored-By: douglasborthwick-crypto <256362537+douglasborthwick-crypto@users.noreply.github.com>
```

Push branch to origin. Do NOT open a PR. Draft flag-plant only.

## Verification scope (runnable in this repo)

The PR references in the spec (`PR #9 v0.5.6-draft at 68f4db4`, `PR #22 v0.6.3-draft at 3c92a54`) point into `agent-intent/verifiable-intent`, NOT this repo. They are not verifiable from inside environment-charter's git log; treat them as accurate cross-repo references sourced from conversation history dated Apr 27, 2026.

What CC should verify:

1. Schema validates as well-formed JSON Schema Draft 2020-12. (Already validated upstream: 11 test cases pass — 2 positive, 9 negative — including additionalProperties:false, CAIP-2 pattern, HTTPS pattern, status enum, max_attestation_age REQUIRED-with-no-default. Quick re-validation welcome.)

2. Spec markdown renders cleanly (table syntax intact, code fences closed, no broken links).

3. BRANCH_NOTES.md placement at repo root is reasonable (no prior precedent in this fresh repo).

4. Co-Authored-By email format is valid GitHub noreply form.

What CC should NOT propose changes to:

- Status enum (LIVE / DEGRADED / STALLED / UNKNOWN)
- Constraint-level vs per-axis freshness layering
- §8 Open Questions in the spec
- The five design choices listed above

If validation passes, commit per the commit plan and push.
